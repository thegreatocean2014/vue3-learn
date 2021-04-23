# **高阶指南**      

## 1. **响应性**

### 1.1 深入响应性原理

现在是时候深入了！Vue 最独特的特性之一，是其非侵入性的响应性系统。

数据模型是被代理的 JavaScript 对象。而当你修改它们时，视图会进行更新。这让状态管理非常简单直观，不过理解其工作原理同样重要，这样你可以避开一些常见的问题。在这个章节，我们将研究一下 Vue 响应性系统的底层的细节。

#### 1.1.1 什么是响应性

那么我们如何用 JavaScript 实现这一点呢？

- 检测其中某一个值是否发生变化
- 用跟踪 (track) 函数修改值
- 用触发 (trigger) 函数更新为最新的值

#### 1.1.2 Vue 如何追踪变化？

当把一个普通的 JavaScript 对象作为 `data` 选项传给应用或组件实例的时候，Vue 会使用带有 getter 和 setter 的处理程序遍历其所有 property 并将其转换为 [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)。这是 ES6 仅有的特性，但是我们在 Vue 3 版本也使用了 `Object.defineProperty` 来支持 IE 浏览器。两者具有相同的 Surface API，但是 Proxy 版本更精简，同时提升了性能。

该部分需要稍微地了解下 [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) 的某些知识！所以，让我们深入了解一下。关于 Proxy 的文献很多，但是你真正需要知道的是 **Proxy 是一个包含另一个对象或函数并允许你对其进行拦截的对象。**

我们是这样使用它的：`new Proxy(target, handler)`

```js
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, prop) {
    return target[prop]
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// tacos
```

好的，到目前为止，我们只是包裹这个对象并返回它。很酷，但还不是那么有用。请注意，我们把对象包裹在 Proxy 里的同时可以对其进行拦截。这种拦截被称为陷阱。

```js
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, prop) {
    console.log('intercepted!')
    return target[prop]
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// tacos
```

除了控制台日志，我们可以在这里做任何我们想做的事情。如果我们愿意，我们甚至可以不返回实际值。这就是为什么 Proxy 对于创建 API 如此强大。

此外，Proxy 还提供了另一个特性。我们不必像这样返回值：`target[prop]`，而是可以进一步使用一个名为 `Reflect` 的方法，它允许我们正确地执行 `this` 绑定，就像这样：

```js
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, prop, receiver) {
    return Reflect.get(...arguments)
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// tacos
```

我们之前提到过，为了有一个 API 能够在某些内容发生变化时更新最终值，我们必须在内容发生变化时设置新的值。我们在处理器，一个名为 `track` 的函数中执行此操作，该函数可以传入 `target` 和 `key` 两个参数。

```js
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, prop, receiver) {
    track(target, prop)
    return Reflect.get(...arguments)
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// tacos
```

最后，当某些内容发生改变时我们会设置新的值。为此，我们将通过触发这些更改来设置新 proxy 的更改：

```js
const dinner = {
  meal: 'tacos'
}

const handler = {
  get(target, prop, receiver) {
    track(target, prop)
    return Reflect.get(...arguments)
  },
  set(target, key, value, receiver) {
    trigger(target, key)
    return Reflect.set(...arguments)
  }
}

const proxy = new Proxy(dinner, handler)
console.log(proxy.meal)

// tacos
```

还记得几段前的列表吗？现在我们有了一些关于 Vue 如何处理这些更改的答案：

- ~~当某个值发生变化时进行检测~~：我们不再需要这样做，因为 Proxy 允许我们拦截它
- **跟踪更改它的函数**：我们在 proxy 中的 getter 中执行此操作，称为 `effect`
- **触发函数以便它可以更新最终值**：我们在 proxy 中的 setter 中进行该操作，名为 `trigger`

proxy 对象对于用户来说是不可见的，但是在内部，它们使 Vue 能够在 property 的值被访问或修改的情况下进行依赖跟踪和变更通知。从 Vue 3 开始，我们的响应性现在可以在[独立的包](https://github.com/vuejs/vue-next/tree/master/packages/reactivity)中使用。需要注意的是，记录转换后的数据对象时，浏览器控制台输出的格式会有所不同，因此你可能需要安装 [vue-devtools](https://github.com/vuejs/vue-devtools)，以提供一种更易于检查的界面。

##### Proxy 对象

Vue 在内部跟踪所有已被设置为响应式的对象，因此它始终会返回同一个对象的 proxy 版本。

从响应式 proxy 访问嵌套对象时，该对象在返回之前*也*被转换为 proxy：

```js
const handler = {
  get(target, prop, receiver) {
    track(target, prop)
    const value = Reflect.get(...arguments)
    if (isObject(value)) {
      return reactive(value)
    } else {
      return value
    }
  }
  // ...
}
```

##### Proxy vs 原始标识

Proxy 的使用确实引入了一个需要注意的新警告：在身份比较方面，被代理对象与原始对象不相等 (`===`)。例如：

```js
const obj = {}
const wrapped = new Proxy(obj, handlers)

console.log(obj === wrapped) // false
```

在大多数情况下，原始版本和被包裹版本的行为相同，但请注意，它们在依赖严格比对的操作下将是失败的，例如 `.filter()` 或 `.map()`。使用选项式 API 时，这种警告不太可能出现，因为所有响应式都是从 `this` 访问的，并保证已经是 proxy。

但是，当使用组合式 API 显式创建响应式对象时，最佳做法是不要保留对原始对象的引用，而只使用响应式版本：

```js
const obj = reactive({
  count: 0
}) // no reference to original
```

#### 1.1.3 侦听器

每个组件实例都有一个相应的侦听器实例，该实例将在组件渲染期间把“触碰”的所有 property 记录为依赖项。之后，当触发依赖项的 setter 时，它会通知侦听器，从而使得组件重新渲染。

将对象作为数据传递给组件实例时，Vue 会将其转换为 proxy。这个 proxy 使 Vue 能够在 property 被访问或修改时执行依赖项跟踪和更改通知。每个 property 都被视为一个依赖项。

首次渲染后，组件将跟踪一组依赖列表——即在渲染过程中被访问的 property。反过来，组件就成为了其每个 property 的订阅者。当 proxy 拦截到 set 操作时，该 property 将通知其所有订阅的组件重新渲染。

如果你使用的是 Vue2.x 及以下版本，你可能会对这些版本中存在的一些更改检测警告感兴趣，[在这里进行更详细的探讨](https://v3.cn.vuejs.org/guide/change-detection.html)。



### 1.2 响应性基础

#### 1.2.1 声明响应式状态

要为 JavaScript 对象创建响应式状态，可以使用 `reactive` 方法：

```js
import { reactive } from 'vue'

// 响应式状态
const state = reactive({
  count: 0
})
```

`reactive` 相当于 Vue 2.x 中的 `Vue.observable()` API ，为避免与 RxJS 中的 observables 混淆因此对其重命名。该 API 返回一个响应式的对象状态。该响应式转换是“深度转换”——它会影响嵌套对象传递的所有 property。

Vue 中响应式状态的基本用例是我们可以在渲染期间使用它。因为依赖跟踪的关系，当响应式状态改变时视图会自动更新。

这就是 Vue 响应性系统的本质。当从组件中的 `data()` 返回一个对象时，它在内部交由 `reactive()` 使其成为响应式对象。模板会被编译成能够使用这些响应式 property 的[渲染函数](https://v3.cn.vuejs.org/guide/render-function.html)。

在[响应性基础 API](https://v3.cn.vuejs.org/api/basic-reactivity.html) 章节你可以学习更多关于 `reactive` 的内容。

#### 1.2.2 创建独立的响应式值作为 `refs`

想象一下，我们有一个独立的原始值 (例如，一个字符串)，我们想让它变成响应式的。当然，我们可以创建一个拥有相同字符串 property 的对象，并将其传递给 `reactive`。Vue 为我们提供了一个可以做相同事情的方法 ——`ref`：

```js
import { ref } from 'vue'

const count = ref(0)
```

`ref` 会返回一个可变的响应式对象，该对象作为它的内部值——一个**响应式的引用**，这就是名称的来源。此对象只包含一个名为 `value` 的 property ：

```js
import { ref } from 'vue'

const count = ref(0)
console.log(count.value) // 0

count.value++
console.log(count.value) // 1
```

##### Ref 展开

当 ref 作为渲染上下文 (从 [setup()](https://v3.cn.vuejs.org/guide/composition-api-setup.html) 中返回的对象) 上的 property 返回并可以在模板中被访问时，它将自动展开为内部值。不需要在模板中追加 `.value`：

```vue-html
<template>
  <div>
    <span>{{ count }}</span>
    <button @click="count ++">Increment count</button>
  </div>
</template>

<script>
  import { ref } from 'vue'
  export default {
    setup() {
      const count = ref(0)
      return {
        count
      }
    }
  }
</script>
```

##### 访问响应式对象

当 `ref` 作为响应式对象的 property 被访问或更改时，为使其行为类似于普通 property，它会自动展开内部值：

```js
const count = ref(0)
const state = reactive({
  count
})

console.log(state.count) // 0

state.count = 1
console.log(count.value) // 1
```

如果将新的 ref 赋值给现有 ref 的 property，将会替换旧的 ref：

```js
const otherCount = ref(2)

state.count = otherCount
console.log(state.count) // 2
console.log(count.value) // 1
```

Ref 展开仅发生在被响应式 `Object` 嵌套的时候。当从 `Array` 或原生集合类型如 [`Map`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)访问 ref 时，不会进行展开：

```js
const books = reactive([ref('Vue 3 Guide')])
// 这里需要 .value
console.log(books[0].value)

const map = reactive(new Map([['count', ref(0)]]))
// 这里需要 .value
console.log(map.get('count').value)
```

#### 1.2.3 响应式状态解构

当我们想使用大型响应式对象的一些 property 时，可能很想使用 [ES6 解构](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)来获取我们想要的 property：

```js
import { reactive } from 'vue'

const book = reactive({
  author: 'Vue Team',
  year: '2020',
  title: 'Vue 3 Guide',
  description: 'You are reading this book right now ;)',
  price: 'free'
})

let { author, title } = book
```

遗憾的是，使用解构的两个 property 的响应性都会丢失。对于这种情况，我们需要将我们的响应式对象转换为一组 ref。这些 ref 将保留与源对象的响应式关联：

```js
import { reactive, toRefs } from 'vue'

const book = reactive({
  author: 'Vue Team',
  year: '2020',
  title: 'Vue 3 Guide',
  description: 'You are reading this book right now ;)',
  price: 'free'
})

let { author, title } = toRefs(book)

title.value = 'Vue 3 Detailed Guide' // 我们需要使用 .value 作为标题，现在是 ref
console.log(book.title) // 'Vue 3 Detailed Guide'
```

你可以在 [Refs API](https://v3.cn.vuejs.org/api/refs-api.html#ref) 部分中了解更多有关 `refs` 的信息

#### 1.2.4 使用 `readonly` 防止更改响应式对象

有时我们想跟踪响应式对象 (`ref` 或 `reactive`) 的变化，但我们也希望防止在应用程序的某个位置更改它。例如，当我们有一个被 [provide](https://v3.cn.vuejs.org/guide/component-provide-inject.html) 的响应式对象时，我们不想让它在注入的时候被改变。为此，我们可以基于原始对象创建一个只读的 proxy 对象：

```js
import { reactive, readonly } from 'vue'

const original = reactive({ count: 0 })

const copy = readonly(original)

// 通过 original 修改 count，将会触发依赖 copy 的侦听器

original.count++

// 通过 copy 修改 count，将导致失败并出现警告
copy.count++ // 警告: "Set operation on key 'count' failed: target is readonly."
```



### 1.3 响应式计算和侦听

#### 1.3.1 计算值

有时我们需要依赖于其他状态的状态——在 Vue 中，这是用组件[计算属性](https://v3.cn.vuejs.org/guide/computed.html#计算属性和侦听器)处理的，以直接创建计算值，我们可以使用 `computed` 方法：它接受 getter 函数并为 getter 返回的值返回一个不可变的响应式 [ref](https://v3.cn.vuejs.org/guide/reactivity-fundamentals.html#创建独立的响应式值作为-refs) 对象。

```js
const count = ref(1)
const plusOne = computed(() => count.value + 1)

console.log(plusOne.value) // 2

plusOne.value++ // error
```

或者，它可以使用一个带有 `get` 和 `set` 函数的对象来创建一个可写的 ref 对象。

```js
const count = ref(1)
const plusOne = computed({
  get: () => count.value + 1,
  set: val => {
    count.value = val - 1
  }
})

plusOne.value = 1
console.log(count.value) // 0
```

#### 1.3.2 `watchEffect`

为了根据响应式状态*自动应用*和*重新应用*副作用，我们可以使用 `watchEffect` 方法。它立即执行传入的一个函数，同时响应式追踪其依赖，并在其依赖变更时重新运行该函数。

```js
const count = ref(0)

watchEffect(() => console.log(count.value))
// -> logs 0

setTimeout(() => {
  count.value++
  // -> logs 1
}, 100)
```

##### 停止侦听

当 `watchEffect` 在组件的 [setup()](https://v3.cn.vuejs.org/guide/composition-api-setup.html) 函数或[生命周期钩子](https://v3.cn.vuejs.org/guide/composition-api-lifecycle-hooks.html)被调用时，侦听器会被链接到该组件的生命周期，并在组件卸载时自动停止。

在一些情况下，也可以显式调用返回值以停止侦听：

```js
const stop = watchEffect(() => {
  /* ... */
})

// later
stop()
```

#####  清除副作用

有时副作用函数会执行一些异步的副作用，这些响应需要在其失效时清除 (即完成之前状态已改变了) 。所以侦听副作用传入的函数可以接收一个 `onInvalidate` 函数作入参，用来注册清理失效时的回调。当以下情况发生时，这个失效回调会被触发：

- 副作用即将重新执行时
- 侦听器被停止 (如果在 `setup()` 或生命周期钩子函数中使用了 `watchEffect`，则在组件卸载时)

```js
watchEffect(onInvalidate => {
  const token = performAsyncOperation(id.value)
  onInvalidate(() => {
    // id has changed or watcher is stopped.
    // invalidate previously pending async operation
    token.cancel()
  })
})
```

我们之所以是通过传入一个函数去注册失效回调，而不是从回调返回它，是因为返回值对于异步错误处理很重要。

在执行数据请求时，副作用函数往往是一个异步函数：

```js
const data = ref(null)
watchEffect(async onInvalidate => {
   onInvalidate(() => { /* ... */ }) // 我们在Promise解析之前注册清除函数
  data.value = await fetchData(props.id)
})
```

我们知道异步函数都会隐式地返回一个 Promise，但是清理函数必须要在 Promise 被 resolve 之前被注册。另外，Vue 依赖这个返回的 Promise 来自动处理 Promise 链上的潜在错误。

##### 副作用刷新时机

Vue 的响应性系统会缓存副作用函数，并异步地刷新它们，这样可以避免同一个“tick” 中多个状态改变导致的不必要的重复调用。在核心的具体实现中，组件的 `update` 函数也是一个被侦听的副作用。当一个用户定义的副作用函数进入队列时，默认情况下，会在所有的组件 `update` **前**执行：

```html
<template>
  <div>{{ count }}</div>
</template>

<script>
  export default {
    setup() {
      const count = ref(0)

      watchEffect(() => {
        console.log(count.value)
      })

      return {
        count
      }
    }
  }
</script>
```

在这个例子中：

- `count` 会在初始运行时同步打印出来
- 更改 `count` 时，将在组件**更新前**执行副作用。

如果需要在组件更新 (例如：当与[模板引用](https://v3.cn.vuejs.org/guide/composition-api-template-refs.html#watching-template-refs)一起) **后**重新运行侦听器副作用，我们可以传递带有 `flush` 选项的附加 `options` 对象 (默认为 `'pre'`)：

```js
// 在组件更新后触发，这样你就可以访问更新的 DOM。
// 注意：这也将推迟副作用的初始运行，直到组件的首次渲染完成。
watchEffect(
  () => {
    /* ... */
  },
  {
    flush: 'post'
  }
)
```

`flush` 选项还接受 `sync`，这将强制效果始终同步触发。然而，这是低效的，应该很少需要。

##### 侦听器调试

`onTrack` 和 `onTrigger` 选项可用于调试侦听器的行为。

- `onTrack` 将在响应式 property 或 ref 作为依赖项被追踪时被调用。
- `onTrigger` 将在依赖项变更导致副作用被触发时被调用。

这两个回调都将接收到一个包含有关所依赖项信息的调试器事件。建议在以下回调中编写 `debugger` 语句来检查依赖关系：

```js
watchEffect(
  () => {
    /* 副作用 */
  },
  {
    onTrigger(e) {
      debugger
    }
  }
)
```

`onTrack` 和 `onTrigger` 只能在开发模式下工作。

#### 1.3.3 `watch`

`watch` API 完全等同于组件[侦听器](https://v3.cn.vuejs.org/guide/computed.html#侦听器) property。`watch` 需要侦听特定的数据源，并在回调函数中执行副作用。默认情况下，它也是惰性的，即只有当被侦听的源发生变化时才执行回调。

与 [watchEffect](https://v3.cn.vuejs.org/guide/reactivity-computed-watchers.html#watcheffect) 比较，`watch` 允许我们：

- 懒执行副作用；
- 更具体地说明什么状态应该触发侦听器重新运行；
- 访问侦听状态变化前后的值。

##### 侦听单个数据源

侦听器数据源可以是返回值的 getter 函数，也可以直接是 `ref`：

```js
// 侦听一个 getter
const state = reactive({ count: 0 })
watch(
  () => state.count,
  (count, prevCount) => {
    /* ... */
  }
)

// 直接侦听ref
const count = ref(0)
watch(count, (count, prevCount) => {
  /* ... */
})
```

##### 侦听多个数据源

侦听器还可以使用数组同时侦听多个源：

```js
const firstName = ref('');
const lastName = ref('');

watch([firstName, lastName], (newValues, prevValues) => {
  console.log(newValues, prevValues);
})

firstName.value = "John"; // logs: ["John",""] ["", ""]
lastName.value = "Smith"; // logs: ["John", "Smith"] ["John", ""]
```

##### 侦听响应式对象

使用侦听器来比较一个数组或对象的值，这些值是响应式的，要求它有一个由值构成的副本。

```js
const numbers = reactive([1, 2, 3, 4])

watch(
  () => [...numbers],
  (numbers, prevNumbers) => {
    console.log(numbers, prevNumbers);
  })

numbers.push(5) // logs: [1,2,3,4,5] [1,2,3,4]
```

尝试检查深度嵌套对象或数组中的 property 变化时，仍然需要 `deep` 选项设置为 true。

```js
const state = reactive({ 
  id: 1, 
  attributes: { 
    name: "",
  },
});

watch(
  () => state,
  (state, prevState) => {
    console.log(
      "not deep ",
      state.attributes.name,
      prevState.attributes.name
    );
  }
);

watch(
  () => state,
  (state, prevState) => {
    console.log(
      "deep ",
      state.attributes.name,
      prevState.attributes.name
    );
  },
  { deep: true }
);

state.attributes.name = "Alex"; // 日志: "deep " "Alex" "Alex"
```

然而，侦听一个响应式对象或数组将始终返回该对象的当前值和上一个状态值的引用。为了完全侦听深度嵌套的对象和数组，可能需要对值进行深拷贝。这可以通过诸如 [lodash.cloneDeep](https://lodash.com/docs/4.17.15#cloneDeep) 这样的实用工具来实现。

```js
import _ from 'lodash';

const state = reactive({
  id: 1,
  attributes: {
    name: "",
  },
});

watch(
  () => _.cloneDeep(state),
  (state, prevState) => {
    console.log(
      state.attributes.name, 
      prevState.attributes.name
    );
  }
);

state.attributes.name = "Alex"; // 日志: "Alex" ""
```

##### 与 `watchEffect` 共享的行为

`watch` 与 [`watchEffect`](https://v3.cn.vuejs.org/guide/reactivity-computed-watchers.html#watcheffect)共享[停止侦听](https://v3.cn.vuejs.org/guide/reactivity-computed-watchers.html#停止侦听)，[清除副作用](https://v3.cn.vuejs.org/guide/reactivity-computed-watchers.html#清除副作用) (相应地 `onInvalidate` 会作为回调的第三个参数传入)、[副作用刷新时机](https://v3.cn.vuejs.org/guide/reactivity-computed-watchers.html#副作用刷新时机)和[侦听器调试](https://v3.cn.vuejs.org/guide/reactivity-computed-watchers.html#侦听器调试)行为。



## 2. 渲染机制和优化

```
如果你仅仅想了解如何很好的使用 Vue，那么你不必阅读此页面。但如果你好奇框架底层的渲染工作是如何进行的，那么你可以阅读本页面从而获取更多的信息。
```

###  虚拟 DOM

现在我们知道了侦听器是如何更新组件的，你可能会问这些更改最终是如何应用到 DOM 中的！也许你以前听说过虚拟 DOM，包括 Vue 在内的许多框架都使用这种范式来确保界面能够有效地反映我们在 JavaScript 中更新的更改

我们用 JavaScript 生成名为 Virtual Dom 的 DOM 副本，这样做的原因是用 JavaScript 直接操作 DOM 的计算成本很高。虽然用 JavaScript 执行更新很快，但是找到所需的 DOM 节点并用 JavaScript 更新它们的成本却很高。所以我们批量处理调用，并一次性更改 DOM。

虚拟 DOM 是轻量级的 JavaScript 对象，由渲染函数创建。它包含三个参数：元素，具有数据、prop、attr 等的对象，以及一个数组。数组是我们传递子级的地方，子级也具有所有这些参数，然后它们也可以具有子级，依此类推，直到我们构建完整的元素树为止。

如果需要更新列表项，我们可以借助前面提到的响应性在 JavaScript 中进行。我们将更改应用至 JavaScript 副本、虚拟 DOM 中，然后在它们和实际 DOM 之间执行 diff。只有这样，我们才能对已更改的内容进行更新。虚拟 DOM 允许我们对 UI 进行高效的更新！



## 3. Vue 2 中的更改检测警告

```
该页面仅适用于 Vue 2.x 及更低版本，并假定你已经阅读了[响应性部分](https://v3.cn.vuejs.org/guide/reactivity.html)。请先阅读该部分。
```

由于 JavaScript 的限制，有些 Vue **无法检测**的更改类型。但是，有一些方法可以规避它们以维持响应性。

### 3.1 对于对象

Vue 无法检测到 property 的添加或删除。由于 Vue 在实例初始化期间执行 getter/setter 转换过程，因此必须在 `data` 对象中存在一个 property，以便 Vue 对其进行转换并使其具有响应式。例如：

```js
var vm = new Vue({
  data: {
    a: 1
  }
})
// `vm.a` 现在是响应式的

vm.b = 2
// `vm.b` 不是响应式的
```

对于已经创建的实例，Vue 不允许动态添加根级别的响应式 property。但是，可以使用 `Vue.set(object, propertyName, value)` 方法向嵌套对象添加响应式 property：

```js
Vue.set(vm.someObject, 'b', 2)
```

你还可以使用 `vm.$set` 实例方法，这也是全局 `Vue.set` 方法的别名：

```js
this.$set(this.someObject, 'b', 2)
```

有时你可能需要为已有对象赋值多个新 property，比如使用 `Object.assign()` 或 `_.extend()`。但是，这样添加到对象上的新 property 不会触发更新。在这种情况下，你应该用原对象与要混合进去的对象的 property 一起创建一个新的对象。

```js
// 而不是 `Object.assign(this.someObject, { a: 1, b: 2 })`
this.someObject = Object.assign({}, this.someObject, { a: 1, b: 2 })
```

### 3.2 对于数组

Vue 不能检测以下数组的变动：

1. 当你利用索引直接设置一个数组项时，例如：`vm.items[indexOfItem] = newValue`
2. 当你修改数组的长度时，例如：`vm.items.length = newLength`

例如：

```js
var vm = new Vue({
  data: {
    items: ['a', 'b', 'c']
  }
})
vm.items[1] = 'x' // 不是响应式的
vm.items.length = 2 // 不是响应式的
```

为了解决第一种问题，以下两种方式都可以实现和 `vm.items[indexOfItem] = newValue` 相同的效果，同时也会触发响应性系统的状态更新：

```js
// Vue.set
Vue.set(vm.items, indexOfItem, newValue)
```

```js
// Array.prototype.splice
vm.items.splice(indexOfItem, 1, newValue)
```

你也可以使用 [`vm.$set`](https://cn.vuejs.org/v2/api/#vm-set) 实例方法，该方法是全局方法 `Vue.set` 的一个别名：

```js
vm.$set(vm.items, indexOfItem, newValue)
```

为了解决第二种问题，你可以使用 `splice`：

```js
vm.items.splice(newLength)
```

### 3.3 声明响应式 property

由于 Vue 不允许动态添加根级响应式 property，所以你必须在初始化实例前声明所有根级响应式 property，哪怕只是一个空值：

```js
var vm = new Vue({
  data: {
    // 声明 message 为一个空值字符串
    message: ''
  },
  template: '<div>{{ message }}</div>'
})
// 之后设置 `message`
vm.message = 'Hello!'
```

如果你未在 data 选项中声明 `message`，Vue 将警告你渲染函数正在试图访问不存在的 property。

这样的限制在背后是有其技术原因的，它消除了在依赖追踪系统中的一类边界情况，也使组件实例能更好地配合类型检查系统工作。但与此同时在代码可维护性方面也有一点重要的考虑：`data` 对象就像组件的状态结构 (schema)。提前声明所有的响应式 property，可以让组件代码在未来修改或给其他开发人员阅读时更易于理解。

### 3.4 异步更新队列

可能你还没有注意到，Vue 在更新 DOM 时是**异步**执行的。只要侦听到数据变化，Vue 将开启一个队列，并缓冲在同一事件循环中发生的所有数据变更。如果同一个侦听器被多次触发，它只会被推入到队列中一次。这种在缓冲时去除重复数据对于避免不必要的计算和 DOM 操作是非常重要的。然后，在下一个的事件循环“tick”中，Vue 刷新队列并执行实际 (已去重的) 工作。Vue 在内部对异步队列尝试使用原生的 `Promise.then`、`MutationObserver` 和 `setImmediate`，如果执行环境不支持，则会采用 `setTimeout(fn, 0)` 代替。

例如，当你设置 `vm.someData = 'new value'`，该组件不会立即重新渲染。当刷新队列时，组件会在下一个事件循环“tick”中更新。多数情况我们不需要关心这个过程，但是如果你想基于更新后的 DOM 状态来做点什么，这就可能会有些棘手。虽然 Vue.js 通常鼓励开发人员使用“数据驱动”的方式思考，避免直接操作 DOM，但是有时我们必须要这么做。为了在数据变化之后等待 Vue 完成更新 DOM，可以在数据变化之后立即使用 `Vue.nextTick(callback)`。这样回调函数将在 DOM 更新完成后被调用。例如：

```html
<div id="example">{{ message }}</div>
```

```js
var vm = new Vue({
  el: '#example',
  data: {
    message: '123'
  }
})
vm.message = 'new message'              // 更改数据
vm.$el.textContent === 'new message'    // false
Vue.nextTick(function() {
  vm.$el.textContent === 'new message'  // true
})
```

在组件内使用 `vm.$nextTick()` 实例方法特别方便，因为它不需要全局 `Vue`，并且回调函数中的 `this` 将自动绑定到当前的组件实例上：

```js
Vue.component('example', {
  template: '<span>{{ message }}</span>',
  data: function() {
    return {
      message: 'not updated'
    }
  },
  methods: {
    updateMessage: function() {
      this.message = 'updated'
      console.log(this.$el.textContent)   // => 'not updated'
      this.$nextTick(function() {
        console.log(this.$el.textContent) // => 'updated'
      })
    }
  }
})
```

因为 `$nextTick()` 返回一个 Promise 对象，所以你可以使用新的 [ES2017 async/await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) 语法完成相同的事情：

```js
  methods: {
    updateMessage: async function () {
      this.message = 'updated'
      console.log(this.$el.textContent) // => 'not updated'
      await this.$nextTick()
      console.log(this.$el.textContent) // => 'updated'
    }
  }
```





















































































































# **高阶指南**      结束