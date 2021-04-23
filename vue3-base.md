```
访问地址
https://v3.cn.vuejs.org/guide/installation.html#%E5%8F%91%E5%B8%83%E7%89%88%E6%9C%AC%E8%AF%B4%E6%98%8E
```



# 基础   开始



## 1. 安装

###  1.1 Vue Devtools

> 目前处于测试阶段 - Vuex 和 Router 的集成仍在进行中。

### 1.2 将 Vue.js 添加到项目中有三种主要方式：

1.  CDN

   <script src="https://unpkg.com/vue@next"></script>

2.  npm

   ```bash
   $ npm install vue@next
   ```

3.  命令行工具 (CLI)

   ```bash
   对于 Vue 3，你应该使用 npm 上可用的 Vue CLI v4.5 作为 @vue/cli。
   
   要升级，你应该需要全局重新安装最新版本的 @vue/cli：
   yarn global add @vue/cli
   # OR
   npm install -g @vue/cli
   
   然后在 Vue 项目中运行：
   vue upgrade --next
   ```

4. Vite

   ```bash
   Vite 是一个 web 开发构建工具，由于其原生 ES 模块导入方式，可以实现闪电般的冷服务器启动。
   使用 npm：
   $ npm init @vitejs/app <project-name>
   $ cd <project-name>
   $ npm install
   $ npm run dev
   
   或者 yarn：
   $ yarn create @vitejs/app <project-name>
   $ cd <project-name>
   $ yarn
   $ yarn dev
   ```

### 1.3 对不同构建版本的解释

在 [npm 包的 dist/ 目录](https://cdn.jsdelivr.net/npm/vue@3.0.2/dist/)你将会找到很多不同的 Vue.js 构建版本。下面是一个概述，根据不同的使用情况，应该使用哪个 `dist` 文件：

1. #### 使用 CDN 或没有构建工具

   ##### `vue(.runtime).global(.prod).js`：

   - 若要通过浏览器中的 `<script src="...">` 直接使用，则暴露 Vue 全局。

   - 浏览器内模板编译：

     - `vue.global.js` 是包含编译器和运行时的“完整”构建，因此它支持动态编译模板。
     - `vue.runtime.global.js` 只包含运行时，并且需要在构建步骤期间预编译模板。

   - 内联所有 Vue 核心内部包——即：它是一个单独的文件，不依赖于其他文件。这意味着你必须导入此文件和此文件中的所有内容，以确保获得相同的代码实例。

   - 包含硬编码的 prod/dev 分支，并且 prod 构建是预先压缩过的。将 `*.prod.js` 文件用于生产环境。

     ##### 提示

     *全局打包不是 [UMD](https://github.com/umdjs/umd) 构建的，它们被打包成 [IIFEs](https://developer.mozilla.org/en-US/docs/Glossary/IIFE)，并且仅用于通过 `<script src="...">` 直接使用。*

   ##### `vue(.runtime).esm-browser(.prod).js`：

   - 用于通过原生 ES 模块导入使用 (在浏览器中通过 `<script type="module">` 来使用)。
   - 与全局构建共享相同的运行时编译、依赖内联和硬编码的 prod/dev 行为。

2. #### 使用构建工具

   ##### `vue(.runtime).esm-bundler.js`：

   - 用于 `webpack`，`rollup` 和 `parcel` 等构建工具。
   - 留下 prod/dev 分支的 `process.env.NODE_ENV` 守卫语句 (必须由构建工具替换)。
   - 不提供压缩版本 (打包后与其余代码一起压缩)。
   - import 依赖 (例如：@vue/runtime-core，@vue/runtime-compiler）
     - 导入的依赖项也是 esm bundler 构建，并将依次导入其依赖项 (例如：@vue/runtime-core imports @vue/reactivity)。
     - 这意味着你**可以**单独安装/导入这些依赖，而不会导致这些依赖项的不同实例，但你必须确保它们都为同一版本。
   - 浏览器内模板编译：
     - `vue.runtime.esm-bundler.js` **(默认)** 仅运行时，并要求所有模板都要预先编译。这是构建工具的默认入口 (通过 `package.json` 中的 module 字段)，因为在使用构建工具时，模板通常是预先编译的 (例如：在 `*.vue` 文件中)。
     - `vue.esm-bundler.js` 包含运行时编译器。如果你使用了一个构建工具，但仍然想要运行时的模板编译 (例如，in-DOM 模板或通过内联 JavaScript 字符串的模板)，请使用这个文件。你需要配置你的构建工具，将 vue 设置为这个文件。

3. ####  对于服务端渲染

   ##### `vue.cjs(.prod).js`：

   - 通过 `require()` 在 Node.js 服务器端渲染使用。
   - 如果你将应用程序与带有 `target: 'node'` 的 webpack 打包在一起，并正确地将 `vue` 外部化，则将加载此文件。
   - dev/prod 文件是预构建的，但是会根据 `process.env.NODE_ENV` 自动加载相应的文件。

4. #### 运行时 + 编译器 vs. 仅运行时

   如果需要在客户端上编译模板 (即：将字符串传递给 template 选项，或者使用元素的 in-DOM HTML 作为模板挂载到元素)，你将需要编译器，因此需要完整的构建版本：

   ```js
   // 需要编译器
   Vue.createApp({
     template: '<div>{{ hi }}</div>'
   })
   
   // 不需要
   Vue.createApp({
     render() {
       return Vue.h('div', {}, this.hi)
     }
   })
   ```

   当使用 `vue-loader` 时，`*.vue` 文件中的模板会在构建时预编译为 JavaScript，在最终的捆绑包中并不需要编译器，因此可以只使用运行时构建版本。

   

## 2. 介绍

### 2.1 Vue.js 是什么

1. Vue (读音 /vjuː/，类似于 **view**) 是一套用于构建用户界面的**渐进式框架**。
2. 与其它大型框架不同的是，Vue 被设计为可以自底向上逐层应用。
3. Vue 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。
4. 另一方面，当与[现代化的工具链](https://v3.cn.vuejs.org/guide/single-file-component.html)以及各种[支持类库](https://github.com/vuejs/awesome-vue#components--libraries)结合使用时，Vue 也完全能够为复杂的单页应用提供驱动。

### 2.2 特点

1. **声明式渲染**

   (1)  Vue.js 的核心是一个允许采用简洁的模板语法来声明式地将数据渲染进 DOM 的系统， Vue 在背后做了大量工作，现在数据和 DOM 已经被建立了关联，所有东西都是**响应式的**。

   (2)  `v-bind` attribute 被称为**指令**。指令带有前缀 `v-`，以表示它们是 Vue 提供的特殊 attribute。

   (3)   在这里，该指令的意思是：“*将这个元素节点的 `title` attribute 和当前活跃实例的 `message` property 保持一致*”。

2. **处理用户输入**

   (1)   为了让用户和应用进行交互，我们可以用 `v-on` 指令添加一个事件监听器，通过它调用在实例中定义的方法

   (2)   我们更新了应用的状态，但没有触碰 DOM——所有的 DOM 操作都由 Vue 来处理，你编写的代码只需要关注逻辑层面即可

   (3)   Vue 还提供了 `v-model` 指令，它能轻松实现表单输入和应用状态之间的双向绑定。

3. **条件与循环**

   (1)  v-if 不仅可以把数据绑定到 DOM 文本或 attribute，还可以绑定到 DOM 的**结构**

   (2)  Vue 也提供一个强大的过渡效果系统，可以在 Vue 插入/更新/移除元素时自动应用[过渡效果](https://v3.cn.vuejs.org/guide/transitions-enterleave.html)

   (3)  `v-for` 指令可以绑定数组的数据来渲染一个项目列表

4. **组件化应用构建**

   (1)  组件系统是 Vue 的另一个重要概念，因为它是一种抽象，允许我们使用小型、独立和通常可复用的组件构建大型应用。

   (2)  几乎任意类型的应用界面都可以抽象为一个组件树

   (3)  在 Vue 中，组件本质上是一个具有预定义选项的实例。

   (4)  在 Vue 中注册组件很简单：如对 `App` 对象所做的那样创建一个组件对象，并将其定义在父级组件的 `components` 选项中

   (5)  **组件使用步骤：**定义组件 -> 引用组件 -> components中声明 -> 模板中使用组件 

   修改一下组件的定义，使之能够接受一个 [prop](https://v3.cn.vuejs.org/guide/component-basics.html#通过-Prop-向子组件传递数据)，可以使用 `v-bind` 指令将待办项传到循环输出的每个组件中，子单元通过 prop 接口与父单元进行了良好的解耦。

   (6)  在一个大型应用中，有必要将整个应用程序划分为多个组件，以使开发更易管理。

   <img src="https://v3.cn.vuejs.org/images/components.png" alt="Component Tree" style="zoom: 50%;" />

5. **与自定义元素的关系**

   你可能已经注意到 Vue 组件非常类似于自定义元素——它是 [Web 组件规范](https://www.w3.org/wiki/WebComponents/)的一部分，这是因为 Vue 的组件语法部分参考了该规范。例如 Vue 组件实现了 [Slot API](https://github.com/w3c/webcomponents/blob/gh-pages/proposals/Slots-Proposal.md) 与 `is` attribute。但是，还是有几个关键差别：

   1. Web Components 规范已经完成并通过，但未被所有浏览器原生实现。目前 Safari 10.1+、Chrome 54+ 和 Firefox 63+ 原生支持 Web Components。相比之下，Vue 组件不需要任何 polyfill，并且在所有支持的浏览器 (IE11 及更高版本) 之下表现一致。必要时，Vue 组件也可以包裹于原生自定义元素之内。
   2. Vue 组件提供了纯自定义元素所不具备的一些重要功能，最突出的是跨组件数据流、自定义事件通信以及构建工具集成。

   虽然 Vue 内部没有使用自定义元素，不过在应用使用自定义元素、或以自定义元素形式发布时，[依然有很好的互操作性](https://custom-elements-everywhere.com/#vue)。Vue CLI 也支持将 Vue 组件构建成为原生的自定义元素。



## 3. 应用 & 组件实例

### 3.1 创建一个应用实例

1. 每个 Vue 应用都是通过用 `createApp` 函数创建一个新的**应用实例**开始的：

2. 该应用实例是用来在应用中注册“全局”组件的。

3. 应用实例暴露的大多数方法都会返回该同一实例，允许链式

   ```js
   Vue.createApp({})
     .component('SearchInput', SearchInputComponent)
     .directive('focus', FocusDirective)
     .use(LocalePlugin)
   ```

###  3.2 根组件

1. 传递给 `createApp` 的选项用于配置**根组件**。当我们**挂载**应用时，该组件被用作渲染的起点。

2. 一个应用需要被挂载到一个 DOM 元素中。例如，如果你想把一个 Vue 应用挂载到 `<div id="app"></div>`，应该传入 `#app`：

   ```js
   const RootComponent = { 
     /* 选项 */ 
   }
   const app = Vue.createApp(RootComponent)
   const vm = app.mount('#app')
   ```

3. 与大多数应用方法不同的是，`mount` 不返回应用本身。相反，它返回的是根组件实例。

4. 虽然没有完全遵循 [MVVM 模型](https://en.wikipedia.org/wiki/Model_View_ViewModel)，但是 Vue 的设计也受到了它的启发。因此在文档中经常会使用 `vm` (ViewModel 的缩写) 这个变量名表示组件实例。

5. 尽管本页面上的所有示例都只需要一个单一的组件就可以，但是大多数的真实应用都是被组织成一个嵌套的、可重用的组件树。举个例子，一个 todo 应用组件树可能是这样的：

   ```text
   Root Component
   └─ TodoList
      ├─ TodoItem
      │  ├─ DeleteTodoButton
      │  └─ EditTodoButton
      └─ TodoListFooter
         ├─ ClearTodosButton
         └─ TodoListStatistics
   ```

6. 每个组件将有自己的组件实例 `vm`。对于一些组件，如 `TodoItem`，在任何时候都可能有多个实例渲染。这个应用中的所有组件实例将共享同一个应用实例。

7. 需要明白根组件与其他组件没什么不同，配置选项是一样的，所对应的组件实例行为也是一样的。

### 3.3 组件实例 property

1. data`，` methods`，`props`，`computed`，`inject` 和 `setup
2. 组件实例的所有 property，无论如何定义，都可以在组件的模板中访问。
3. Vue 还通过组件实例暴露了一些内置 property，如 `$attrs` 和 `$emit`。这些 property 都有一个 `$` 前缀，以避免与用户定义的 property 名冲突。

### 3.4 生命周期钩子

1. 每个组件在被创建时都要经过一系列的初始化过程——例如，需要设置数据监听、编译模板、将实例挂载到 DOM 并在数据变化时更新 DOM 等。

2. 同时在这个过程中也会运行一些叫做**生命周期钩子**的函数，这给了用户在不同阶段添加自己的代码的机会。

3. [created](https://v3.cn.vuejs.org/api/options-lifecycle-hooks.html#created) 钩子可以用来在一个实例被创建之后执行代码：

4. 也有一些其它的钩子，在实例生命周期的不同阶段被调用，如 [mounted](https://v3.cn.vuejs.org/api/options-lifecycle-hooks.html#mounted)、[updated](https://v3.cn.vuejs.org/api/options-lifecycle-hooks.html#updated) 和 [unmounted](https://v3.cn.vuejs.org/api/options-lifecycle-hooks.html#unmounted)。

5. 生命周期钩子的 `this` 上下文指向调用它的当前活动实例。

   **TIP**

   *不要在选项 property 或回调上使用[箭头函数](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions)，比如 `created: () => console.log(this.a)` 或 `vm.$watch('a', newValue => this.myMethod())`。因为箭头函数并没有 `this`，`this` 会作为变量一直向上级词法作用域查找，直至找到为止，经常导致 `Uncaught TypeError: Cannot read property of undefined` 或 `Uncaught TypeError: this.myMethod is not a function` 之类的错误。*

### 3.5 生命周期图示

<img src="https://v3.cn.vuejs.org/images/lifecycle.svg" alt="Component Tree" style="zoom: 100%;" />

## 4. 模板语法

Vue.js 使用了基于 HTML 的模板语法，允许开发者声明式地将 DOM 绑定至底层组件实例的数据。

所有 Vue.js 的模板都是合法的 HTML，所以能被遵循规范的浏览器和 HTML 解析器解析。

在底层的实现上，Vue 将模板编译成虚拟 DOM 渲染函数。结合响应性系统，Vue 能够智能地计算出最少需要重新渲染多少组件，并把 DOM 操作次数减到最少。

如果你熟悉虚拟 DOM 并且偏爱 JavaScript 的原始力量，你也可以不用模板，[直接写渲染 (render) 函数](https://v3.cn.vuejs.org/guide/render-function.html)，使用可选的 JSX 语法。

### 4.1 插值

#### 文本

数据绑定最常见的形式就是使用“Mustache”语法 (双大括号) 的文本插值：

```html
<span>Message: {{ msg }}</span>
```

Mustache 标签将会被替代为对应组件实例中 `msg` property 的值。无论何时，绑定的组件实例上 `msg` property 发生了改变，插值处的内容都会更新。

通过使用 [v-once 指令](https://v3.cn.vuejs.org/api/directives.html#v-once)，你也能执行一次性地插值，当数据改变时，插值处的内容不会更新。但请留心这会影响到该节点上的其它数据绑定：

```html
<span v-once>这个将不会改变: {{ msg }}</span>
```

####  原始 HTML

双大括号会将数据解释为普通文本，而非 HTML 代码。为了输出真正的 HTML，你需要使用[`v-html` 指令](https://v3.cn.vuejs.org/api/directives.html#v-html)：

```html
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```

这个 `span` 的内容将会被替换成为 property 值 `rawHtml`，直接作为 HTML——会忽略解析 property 值中的数据绑定。

**注意，你不能使用 `v-html` 来复合局部模板，因为 Vue 不是基于字符串的模板引擎。反之，对于用户界面 (UI)，组件更适合作为可重用和可组合的基本单位。**

####  Attribute

使用 [`v-bind` 指令](https://v3.cn.vuejs.org/api/directives.html#v-bind)：

```html
<div v-bind:id="dynamicId"></div>
```

如果绑定的值是 `null` 或 `undefined`，那么该 attribute 将不会被包含在渲染的元素上。

对于布尔 attribute (它们只要存在就意味着值为 `true`)，`v-bind` 工作起来略有不同，在这个例子中：

```html
<button v-bind:disabled="isButtonDisabled">按钮</button>
```

1. 如果 `isButtonDisabled` 的值是 truthy[[1\]](https://v3.cn.vuejs.org/guide/template-syntax.html#footnote-1)，那么 `disabled` attribute 将被包含在内。

2. 如果该值是一个空字符串，它也会被包括在内，与 `<button disabled="">` 保持一致。

3. 对于其他错误的值，该 attribute 将被省略。


#### 使用 JavaScript 表达式

对于所有的数据绑定，Vue.js 都提供了完全的 JavaScript 表达式支持。

```html
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```

这些表达式会在当前活动实例的数据作用域下作为 JavaScript 被解析。

有个限制就是，每个绑定都只能包含**单个表达式**，所以下面的例子都**不会**生效。

```html
<!--  这是语句，不是表达式：-->
{{ var a = 1 }}

<!-- 流控制也不会生效，请使用三元表达式 -->
{{ if (ok) { return message } }}
```

### 4.2 指令

指令 (Directives) 是带有 `v-` 前缀的特殊 attribute。指令 attribute 的值预期是**单个 JavaScript 表达式**    ***(`v-for` 和 `v-on` 是例外情况)***

指令的职责是，当表达式的值改变时，将其产生的连带影响，响应式地作用于 DOM。

#### 参数

一些指令能够接收一个“参数”，在指令名称之后以冒号表示。

`v-bind` 指令可以用于响应式地更新 HTML attribute：

```html
<a v-bind:href="url"> ... </a>
```

在这里 `href` 是参数，告知 `v-bind` 指令将该元素的 `href` attribute 与表达式 `url` 的值绑定。

另一个例子是 `v-on` 指令，它用于监听 DOM 事件：

```html
<a v-on:click="doSomething"> ... </a>
```

在这里参数是监听的事件名。

#### 动态参数

也可以在指令参数中使用 JavaScript 表达式，方法是用方括号括起来：

```html
<!--
注意，参数表达式的写法存在一些约束，如之后的“对动态参数表达式的约束”章节所述。
-->
<a v-bind:[attributeName]="url"> ... </a>
```

这里的 `attributeName` 会被作为一个 JavaScript 表达式进行动态求值，求得的值将会作为最终的参数来使用。例如，如果你的组件实例有一个 data property `attributeName`，其值为 `"href"`，那么这个绑定将等价于 `v-bind:href`。

同样地，你可以使用动态参数为一个动态的事件名绑定处理函数：

```html
<a v-on:[eventName]="doSomething"> ... </a>
```

在这个示例中，当 `eventName` 的值为 `"focus"` 时，`v-on:[eventName]` 将等价于 `v-on:focus`

#### 修饰符

修饰符 (modifier) 是以半角句号 `.` 指明的特殊后缀，用于指出一个指令应该以特殊方式绑定。

例如，`.prevent` 修饰符告诉 `v-on` 指令对于触发的事件调用 `event.preventDefault()`：

```html
<form v-on:submit.prevent="onSubmit">...</form>
```

### 4.3 缩写

`v-` 前缀作为一种视觉提示，用来识别模板中 Vue 特定的 attribute。

当你在使用 Vue.js 为现有标签添加动态行为 (dynamic behavior) 时，v- 前缀很有帮助，然而，对于一些频繁用到的指令来说，就会感到使用繁琐。

同时，在构建由 Vue 管理所有模板的单页面应用程序 [(SPA - single page application)](https://en.wikipedia.org/wiki/Single-page_application) 时，`v-` 前缀也变得没那么重要了。

因此，Vue 为 `v-bind` 和 `v-on` 这两个最常用的指令，提供了特定简写：

#### `v-bind` 缩写

```html
<!-- 完整语法 -->
<a v-bind:href="url"> ... </a>

<!-- 缩写 -->
<a :href="url"> ... </a>

<!-- 动态参数的缩写 -->
<a :[key]="url"> ... </a>
```

#### `v-on` 缩写

```html
<!-- 完整语法 -->
<a v-on:click="doSomething"> ... </a>

<!-- 缩写 -->
<a @click="doSomething"> ... </a>

<!-- 动态参数的缩写 (2.6.0+) -->
<a @[event]="doSomething"> ... </a>
```

它们看起来可能与普通的 HTML 略有不同，但 `:` 与 `@` 对于 attribute 名来说都是合法字符，在所有支持 Vue 的浏览器都能被正确地解析。

而且，它们不会出现在最终渲染的标记中。

#### 注意事项

1. 对动态参数值约定

   1. 动态参数预期会求出一个字符串，异常情况下值为 `null`。
   2. 这个特殊的 `null` 值可以被显性地用于移除绑定。任何其它非字符串类型的值都将会触发一个警告。

2. 对动态参数表达式约定

   1. 动态参数表达式有一些语法约束，因为某些字符，如空格和引号，放在 HTML attribute 名里是无效的。例如：

      ```html
      <!-- 这会触发一个编译警告 -->
      <a v-bind:['foo' + bar]="value"> ... </a>
      ```

      变通的办法是使用没有空格或引号的表达式，或用[计算属性](https://v3.cn.vuejs.org/guide/computed.html)替代这种复杂表达式。

   2. 在 DOM 中使用模板时 (直接在一个 HTML 文件里撰写模板)，还需要避免使用大写字符来命名键名，因为浏览器会把 attribute 名全部强制转为小写：

      ```html
      <!--
      在 DOM 中使用模板时这段代码会被转换为 `v-bind:[someattr]`。
      除非在实例中有一个名为“someattr”的 property，否则代码不会工作。
      -->
      <a v-bind:[someAttr]="value"> ... </a>
      ```

3. JavaScript 表达式

   1. 模板表达式都被放在沙盒中，只能访问[全局变量的一个白名单](https://github.com/vuejs/vue-next/blob/master/packages/shared/src/globalsWhitelist.ts#L3)，如 `Math` 和 `Date`。
   2. 你不应该在模板表达式中试图访问用户定义的全局变量。



## 5. Data Property 和方法

### 5.1 Data Property

1. 组件的 `data` 选项是一个函数。

2. Vue 在创建新组件实例的过程中调用此函数。

3. 它应该返回一个对象，然后 Vue 会通过响应性系统将其包裹起来，并以 `$data` 的形式存储在组件实例中。

4. 为方便起见，该对象的任何顶级 property 也直接通过组件实例暴露出来：

   ```js
   const app = Vue.createApp({
     data() {
       return { count: 4 }
     }
   })
   
   const vm = app.mount('#app')
   
   console.log(vm.$data.count) // => 4
   console.log(vm.count)       // => 4
   
   // 修改 vm.count 的值也会更新 $data.count
   vm.count = 5
   console.log(vm.$data.count) // => 5
   
   // 反之亦然
   vm.$data.count = 6
   console.log(vm.count) // => 6
   ```

5. 这些实例 property 仅在实例首次创建时被添加，所以你需要确保它们都在 `data` 函数返回的对象中。必要时，要对尚未提供所需值的 property 使用 `null`、`undefined` 或其他占位的值。

6. 直接将不包含在 `data` 中的新 property 添加到组件实例是可行的。但由于该 property 不在背后的响应式 `$data` 对象内，所以 [Vue 的响应性系统](https://v3.cn.vuejs.org/guide/reactivity.html)不会自动跟踪它。

7. Vue 使用 `$` 前缀通过组件实例暴露自己的内置 API。它还为内部 property 保留 `_` 前缀。你应该避免使用这两个字符开头的的顶级 `data` property 名称。

### 5.2 方法

1. 我们用 `methods` 选项向组件实例添加方法，它应该是一个包含所需方法的对象：

   ```js
   const app = Vue.createApp({
     data() {
       return { count: 4 }
     },
     methods: {
       increment() {
         // `this` 指向该组件实例
         this.count++
       }
     }
   })
   
   const vm = app.mount('#app')
   
   console.log(vm.count) // => 4
   
   vm.increment()
   
   console.log(vm.count) // => 5
   ```

2. Vue 自动为 `methods` 绑定 `this`，以便于它始终指向组件实例。这将确保方法在用作事件监听或回调时保持正确的 `this` 指向。在定义 `methods` 时应避免使用箭头函数，因为这会阻止 Vue 绑定恰当的 `this` 指向。

3. 这些 `methods` 和组件实例的其它所有 property 一样可以在组件的模板中被访问。在模板中，它们通常被当做事件监听使用：

   ```html
   <button @click="increment">Up vote</button>
   ```

   在上面的例子中，点击 `<button>` 时，会调用 `increment` 方法。

4. 也可以直接从模板中调用方法。就像下一章节即将看到的，通常换做[计算属性](https://v3.cn.vuejs.org/guide/computed.html)会更好。但是，在计算属性不可行的情况下，使用方法可能会很有用。你可以在模板支持 JavaScript 表达式的任何地方调用方法：

   ```html
   <span :title="toTitleDate(date)">
     {{ formatDate(date) }}
   </span>
   ```

   如果 `toTitleDate` 或 `formatDate` 访问任何响应式数据，则将其作为渲染依赖项进行跟踪，就像直接在模板中使用过一样。

5. 从模板调用的方法不应该有任何副作用，比如更改数据或触发异步进程。如果你想这么做，应该换做[生命周期钩子](https://v3.cn.vuejs.org/guide/instance.html#生命周期钩子)。

#### 防抖和节流

1. Vue 没有内置支持防抖和节流，但可以使用 [Lodash](https://lodash.com/) 等库来实现。

2. 如果某个组件仅使用一次，可以在 `methods` 中直接应用防抖：

   ```html
   <script src="https://unpkg.com/lodash@4.17.20/lodash.min.js"></script>
   <script>
     Vue.createApp({
       methods: {
         // 用 Lodash 的防抖函数
         click: _.debounce(function() {
           // ... 响应点击 ...
         }, 500)
       }
     }).mount('#app')
   </script>
   ```

3. 但是，这种方法对于可复用组件有潜在的问题，因为它们都共享相同的防抖函数。为了使组件实例彼此独立，可以在生命周期钩子的 `created` 里添加该防抖函数:

   ```js
   app.component('save-button', {
     created() {
       // 用 Lodash 的防抖函数
       this.debouncedClick = _.debounce(this.click, 500)
     },
     unmounted() {
       // 移除组件时，取消定时器
       this.debouncedClick.cancel()
     },
     methods: {
       click() {
         // ... 响应点击 ...
       }
     },
     template: `
       <button @click="debouncedClick">
         Save
       </button>
     `
   })
   ```



## 6. 计算属性和侦听器

### 6.1 计算属性

模板内的表达式非常便利，但是设计它们的初衷是用于简单运算的。在模板中放入太多的逻辑会让模板过重且难以维护。

对于任何包含响应式数据的复杂逻辑，你都应该使用**计算属性**。

#### 基本例子

```html
<div id="computed-basics">
  <p>Has published books:</p>
  <span>{{ publishedBooksMessage }}</span>
</div>
```

```js
Vue.createApp({
  data() {
    return {
      author: {
        name: 'John Doe',
        books: [
          'Vue 2 - Advanced Guide',
          'Vue 3 - Basic Guide',
          'Vue 4 - The Mystery'
        ]
      }
    }
  },
  computed: {
    // 计算属性的 getter
    publishedBooksMessage() {
      // `this` 指向 vm 实例
      return this.author.books.length > 0 ? 'Yes' : 'No'
    }
  }
}).mount('#computed-basics')
```

这里声明了一个计算属性 `publishedBooksMessage`。尝试更改应用程序 `data` 中 `books` 数组的值，你将看到 `publishedBooksMessage` 如何相应地更改。

你可以像普通属性一样将数据绑定到模板中的计算属性。Vue 知道 `vm.publishedBookMessage` 依赖于 `vm.author.books`，因此当 `vm.author.books` 发生改变时，所有依赖 `vm.publishedBookMessage` 的绑定也会更新。

而且最妙的是我们已经声明的方式创建了这个依赖关系：计算属性的 getter 函数没有副作用，它更易于测试和理解。

####  计算属性缓存 vs 方法

```html
<p>{{ calculateBooksMessage() }}</p>
```

```js
// 在组件中
methods: {
  calculateBooksMessage() {
    return this.author.books.length > 0 ? 'Yes' : 'No'
  }
}
```

我们可以将同一函数定义为一个方法而不是一个计算属性。两种方式的最终结果确实是完全相同的。

然而，不同的是**计算属性是基于它们的反应依赖关系缓存的**。

计算属性只在相关响应式依赖发生改变时它们才会重新求值。这就意味着只要 `author.books` 还没有发生改变，多次访问 `publishedBookMessage` 计算属性会立即返回之前的计算结果，而不必再次执行函数。

我们为什么需要缓存？假设我们有一个性能开销比较大的计算属性 `list`，它需要遍历一个巨大的数组并做大量的计算。然后我们可能有其他的计算属性依赖于 `list`。如果没有缓存，我们将不可避免的多次执行 `list` 的 getter！如果你不希望有缓存，请用 `method` 来替代。

#### 计算属性的 Setter

计算属性默认只有 getter，不过在需要时你也可以提供一个 setter：

```js
// ...
computed: {
  fullName: {
    // getter
    get() {
      return this.firstName + ' ' + this.lastName
    },
    // setter
    set(newValue) {
      const names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
// ...
```

现在再运行 `vm.fullName = 'John Doe'` 时，setter 会被调用，`vm.firstName` 和 `vm.lastName` 也会相应地被更新。

### 6.2 侦听器

1. 虽然计算属性在大多数情况下更合适，但有时也需要一个自定义的侦听器。

   这就是为什么 Vue 通过 `watch` 选项提供了一个更通用的方法，来响应数据的变化。

2. 当需要在数据变化时执行异步或开销较大的操作时，这个方式是最有用的。

   ```html
   <div id="watch-example">
     <p>
       Ask a yes/no question:
       <input v-model="question" />
     </p>
     <p>{{ answer }}</p>
   </div>
   ```

   ```html
   <!-- 因为 AJAX 库和通用工具的生态已经相当丰富，Vue 核心代码没有重复 -->
   <!-- 提供这些功能以保持精简。这也可以让你自由选择自己更熟悉的工具。 -->
   <script src="https://cdn.jsdelivr.net/npm/axios@0.12.0/dist/axios.min.js"></script>
   <script>
     const watchExampleVM = Vue.createApp({
       data() {
         return {
           question: '',
           answer: 'Questions usually contain a question mark. ;-)'
         }
       },
       watch: {
         // whenever question changes, this function will run
         question(newQuestion, oldQuestion) {
           if (newQuestion.indexOf('?') > -1) {
             this.getAnswer()
           }
         }
       },
       methods: {
         getAnswer() {
           this.answer = 'Thinking...'
           axios
             .get('https://yesno.wtf/api')
             .then(response => {
               this.answer = response.data.answer
             })
             .catch(error => {
               this.answer = 'Error! Could not reach the API. ' + error
             })
         }
       }
     }).mount('#watch-example')
   </script>
   ```

3. 在这个示例中，使用 `watch` 选项允许我们执行异步操作 (访问一个 API)，限制我们执行该操作的频率，并在我们得到最终结果前，设置中间状态。这些都是计算属性无法做到的。

4. 除了 watch 选项之外，你还可以使用命令式的 [vm.$watch API](https://v3.cn.vuejs.org/api/instance-methods.html#watch)。

#### 计算属性 vs 侦听器

1. Vue 提供了一种更通用的方式来观察和响应当前活动的实例上的数据变动：**侦听属性**。
2. 当你有一些数据需要随着其它数据变动而变动时，你很容易滥用 `watch`——特别是如果你之前使用过 AngularJS。
3. 然而，通常更好的做法是使用计算属性而不是命令式的 `watch` 回调。细想一下这个例子：

```html
<div id="demo">{{ fullName }}</div>
```

```js
const vm = Vue.createApp({
  data() {
    return {
      firstName: 'Foo',
      lastName: 'Bar',
      fullName: 'Foo Bar'
    }
  },
  watch: {
    firstName(val) {
      this.fullName = val + ' ' + this.lastName
    },
    lastName(val) {
      this.fullName = this.firstName + ' ' + val
    }
  }
}).mount('#demo')
```

上面代码是命令式且重复的。将它与计算属性的版本进行比较：

```js
const vm = Vue.createApp({
  data() {
    return {
      firstName: 'Foo',
      lastName: 'Bar'
    }
  },
  computed: {
    fullName() {
      return this.firstName + ' ' + this.lastName
    }
  }
}).mount('#demo')
```

相比较好很多，需要根据情况，灵活应用。



## **模板中属性值的几种表示方式：**

**data属性，methods方法，computed计算属性，watch侦听器， [vm.$watch API](https://v3.cn.vuejs.org/api/instance-methods.html#watch)侦听器**



## 7. Class 与 Style 绑定

1. 操作元素的 class 列表和内联样式是数据绑定的一个常见需求。因为它们都是 attribute，所以我们可以用 `v-bind` 处理它们：只需要通过表达式计算出字符串结果即可。
2. 因此，在将 `v-bind` 用于 `class` 和 `style` 时，Vue.js 做了专门的增强。
3. 表达式结果的类型除了字符串之外，还可以是对象或数组。

### 7.1 绑定 HTML Class

#### 对象语法

我们可以传给 `:class` (`v-bind:class` 的简写) 一个对象，以动态地切换 class：

```html
<div :class="{ active: isActive }"></div>
```

上面的语法表示 `active` 这个 class 存在与否将取决于数据 property `isActive` 的 [truthiness](https://developer.mozilla.org/en-US/docs/Glossary/Truthy)

你可以在对象中传入更多字段来动态切换多个 class。

此外，`:class` 指令也可以与普通的 `class` attribute 共存。

当有如下模板：

```html
<div
  class="static"
  :class="{ active: isActive, 'text-danger': hasError }"
></div>
```

和如下 data：

```js
data() {
  return {
    isActive: true,
    hasError: false
  }
}
```

渲染的结果为：

```html
<div class="static active"></div>
```

当 `isActive` 或者 `hasError` 变化时，class 列表将相应地更新。例如，如果 `hasError` 的值为 `true`，class 列表将变为 `"static active text-danger"`。

绑定的数据对象不必内联定义在模板里：

```html
<div :class="classObject"></div>
```

```js
data() {
  return {
    classObject: {
      active: true,
      'text-danger': false
    }
  }
}
```

渲染的结果和上面一样。

我们也可以在这里绑定一个返回对象的[计算属性](https://v3.cn.vuejs.org/guide/computed.html)。这是一个常用且强大的模式：

```html
<div :class="classObject"></div>
```

```js
data() {
  return {
    isActive: true,
    error: null
  }
},
computed: {
  classObject() {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal'
    }
  }
}
```

#### 数组语法

我们可以把一个数组传给 `:class`，以应用一个 class 列表：

```html
<div :class="[activeClass, errorClass]"></div>
```

```js
data() {
  return {
    activeClass: 'active',
    errorClass: 'text-danger'
  }
}
```

渲染的结果为：

```html
<div class="active text-danger"></div>
```

如果你想根据条件切换列表中的 class，可以使用三元表达式：

```html
<div :class="[isActive ? activeClass : '', errorClass]"></div>
```

这样写将始终添加 `errorClass`，但是只有在 `isActive` 为 truthy[[1\]](https://v3.cn.vuejs.org/guide/class-and-style.html#footnote-1) 时才添加 `activeClass`。

不过，当有多个条件 class 时这样写有些繁琐。所以在数组语法中也可以使用对象语法：

```html
<div :class="[{ active: isActive }, errorClass]"></div>
```

#### 在组件上使用

当你在带有单个根元素的自定义组件上使用 `class` attribute 时，这些 class 将被添加到该元素中。此元素上的现有 class 将不会被覆盖。

例如，如果你声明了这个组件：

```js
const app = Vue.createApp({})

app.component('my-component', {
  template: `<p class="foo bar">Hi!</p>`
})
```

然后在使用它的时候添加一些 class：

```html
<div id="app">
  <my-component class="baz boo"></my-component>
</div>
```

HTML 将被渲染为：

```html
<p class="foo bar baz boo">Hi</p>
```

对于带数据绑定 class 也同样适用：

```html
<my-component :class="{ active: isActive }"></my-component>
```

当 isActive 为 truthy[[1\]](https://v3.cn.vuejs.org/guide/class-and-style.html#footnote-1) 时，HTML 将被渲染成为：

```html
<p class="foo bar active">Hi</p>
```

如果你的组件有多个根元素，你需要定义哪些部分将接收这个类。可以使用 `$attrs` 组件属性执行此操作：

```html
<div id="app">
  <my-component class="baz"></my-component>
</div>
```

```js
const app = Vue.createApp({})

app.component('my-component', {
  template: `
    <p :class="$attrs.class">Hi!</p>
    <span>This is a child component</span>
  `
})
```

### 7.2 绑定内联样式

#### 对象语法

`:style` 的对象语法十分直观——看着非常像 CSS，但其实是一个 JavaScript 对象。

CSS property 名可以用驼峰式 (camelCase) 或短横线分隔 (kebab-case，记得用引号括起来) 来命名：

```html
<div :style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```

```js
data() {
  return {
    activeColor: 'red',
    fontSize: 30
  }
}
```

直接绑定到一个样式对象通常更好，这会让模板更清晰：

```html
<div :style="styleObject"></div>
```

```js
data() {
  return {
    styleObject: {
      color: 'red',
      fontSize: '13px'
    }
  }
}
```

同样的，对象语法常常结合返回对象的计算属性使用。

#### 数组语法

`:style` 的数组语法可以将多个样式对象应用到同一个元素上：

```html
<div :style="[baseStyles, overridingStyles]"></div>
```

#### 自动添加前缀

在 `:style` 中使用需要 (浏览器引擎前缀) [vendor prefixes](https://developer.mozilla.org/en-US/docs/Glossary/Vendor_Prefix) 的 CSS property 时，如 `transform`，Vue 将自动侦测并添加相应的前缀。

#### 多重值

可以为 style 绑定中的 property 提供一个包含多个值的数组，常用于提供多个带前缀的值，例如：

```html
<div :style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
```

这样写只会渲染数组中最后一个被浏览器支持的值。在本例中，如果浏览器支持不带浏览器前缀的 flexbox，那么就只会渲染 `display: flex`。



## 8. 条件渲染

### 8.1 `v-if`

`v-if` 指令用于条件性地渲染一块内容。这块内容只会在指令的表达式返回 truthy 值的时候被渲染。

```html
<h1 v-if="awesome">Vue is awesome!</h1>
```

也可以用 `v-else` 添加一个“else 块”：

```html
<h1 v-if="awesome">Vue is awesome!</h1>
<h1 v-else>Oh no 😢</h1>
```

####  在 `<template>` 元素上使用 `v-if` 条件渲染分组

但是如果想切换多个元素呢？此时可以把一个 `<template>` 元素当做不可见的包裹元素，并在上面使用 `v-if`。

最终的渲染结果将不包含 `<template>` 元素。

```html
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```

#### `v-else`

你可以使用 `v-else` 指令来表示 `v-if` 的“else 块”：

```html
<div v-if="Math.random() > 0.5">
  Now you see me
</div>
<div v-else>
  Now you don't
</div>
```

`v-else` 元素必须紧跟在带 `v-if` 或者 `v-else-if` 的元素的后面，否则它将不会被识别。

#### `v-else-if`

充当 `v-if` 的“else-if 块”，并且可以连续使用：

```html
<div v-if="type === 'A'">
  A
</div>
<div v-else-if="type === 'B'">
  B
</div>
<div v-else-if="type === 'C'">
  C
</div>
<div v-else>
  Not A/B/C
</div>
```

与 `v-else` 的用法类似，`v-else-if` 也必须紧跟在带 `v-if` 或者 `v-else-if` 的元素之后。

### 8.2 `v-show`

另一个用于条件性展示元素的选项是 `v-show` 指令。用法大致一样：

```html
<h1 v-show="ok">Hello!</h1>
```

1. 不同的是带有 `v-show` 的元素始终会被渲染并保留在 DOM 中。
2. `v-show` 只是简单地切换元素的 CSS property `display`。
3. **注意，`v-show` 不支持 `<template>` 元素，也不支持 `v-else`。**

### 8.3 `v-if` vs `v-show`

1. `v-if` 是“真正”的条件渲染，因为它会确保在切换过程中，条件块内的事件监听器和子组件适当地被销毁和重建。
2. `v-if` 也是**惰性的**：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。
3. 相比之下，`v-show` 就简单得多——不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 进行切换。
4. 一般来说，`v-if` 有更高的切换开销，而 `v-show` 有更高的初始渲染开销。因此，如果需要非常频繁地切换，则使用 `v-show` 较好；如果在运行时条件很少改变，则使用 `v-if` 较好。

### 8.4 `v-if` 与 `v-for` 一起使用

1. **不推荐**同时使用 `v-if` 和 `v-for`。请查阅[风格指南](https://v3.cn.vuejs.org/style-guide/#avoid-v-if-with-v-for-essential)以获取更多信息。
2. 当 `v-if` 与 `v-for` 一起使用时，`v-if` 具有比 `v-for` 更高的优先级。请查阅[列表渲染指南](https://v3.cn.vuejs.org/guide/list##v-for-与-v-if-一同使用)以获取详细信息。



## 9. 列表渲染

### 9.1 用 `v-for` 把一个数组对应为一组元素

我们可以用 `v-for` 指令基于一个数组来渲染一个列表。

`v-for` 指令需要使用 `item in items` 形式的特殊语法，其中 items 是源数据数组，而 `item` 则是被迭代的数组元素的**别名**。

```html
<ul id="array-rendering">
  <li v-for="item in items">
    {{ item.message }}
  </li>
</ul>
```

```js
Vue.createApp({
  data() {
    return {
      items: [{ message: 'Foo' }, { message: 'Bar' }]
    }
  }
}).mount('#array-rendering')
```

在 `v-for` 块中，我们可以访问所有父作用域的 property。

`v-for` 还支持一个可选的第二个参数，即当前项的索引。

```html
<ul id="array-with-index">
  <li v-for="(item, index) in items">
    {{ parentMessage }} - {{ index }} - {{ item.message }}
  </li>
</ul>
```

```js
Vue.createApp({
  data() {
    return {
      parentMessage: 'Parent',
      items: [{ message: 'Foo' }, { message: 'Bar' }]
    }
  }
}).mount('#array-with-index')
```

可以用 `of` 替代 `in` 作为分隔符，因为它更接近 JavaScript 迭代器的语法：

```html
<div v-for="item of items"></div>
```

### 9.2 在 `v-for` 里使用对象

你也可以用 `v-for` 来遍历一个对象的 property。

```html
<ul id="v-for-object" class="demo">
  <li v-for="value in myObject">
    {{ value }}
  </li>
</ul>
```

```js
Vue.createApp({
  data() {
    return {
      myObject: {
        title: 'How to do lists in Vue',
        author: 'Jane Doe',
        publishedAt: '2016-04-10'
      }
    }
  }
}).mount('#v-for-object')
```

你也可以提供第二个的参数为 property 名称 (也就是键名 key)：

```html
<li v-for="(value, name) in myObject">
  {{ name }}: {{ value }}
</li>
```

还可以用第三个参数作为索引：

```html
<li v-for="(value, name, index) in myObject">
  {{ index }}. {{ name }}: {{ value }}
</li>
```

**提示**

*在遍历对象时，会按 `Object.keys()` 的结果遍历，但是不能保证它在不同 JavaScript 引擎下的结果都一致。*

### 9.3 维护状态

1. 当 Vue 正在更新使用 `v-for` 渲染的元素列表时，它默认使用“就地更新”的策略。

2. 如果数据项的顺序被改变，Vue 将不会移动 DOM 元素来匹配数据项的顺序，而是就地更新每个元素，并且确保它们在每个索引位置正确渲染。

3. 这个默认的模式是高效的，但是**只适用于不依赖子组件状态或临时 DOM 状态 (例如：表单输入值) 的列表渲染输出**。

4. 为了给 Vue 一个提示，以便它能跟踪每个节点的身份，从而重用和重新排序现有元素，你需要为每项提供一个唯一 `key` attribute：

   ```html
   <div v-for="item in items" :key="item.id">
     <!-- content -->
   </div>
   ```

5. 建议尽可能在使用 `v-for` 时提供 `key` attribute，除非遍历输出的 DOM 内容非常简单，或者是刻意依赖默认行为以获取性能上的提升。

6. 因为它是 Vue 识别节点的一个通用机制，`key` 并不仅与 `v-for` 特别关联。

**提示**

*不要使用对象或数组之类的非基本类型值作为 `v-for` 的 key。请用字符串或数值类型的值。*

### 9.4 数组更新检测

#### 变更方法

Vue 将被侦听的数组的变更方法进行了包裹，所以它们也将会触发视图更新。这些被包裹过的方法包括：

1. `push()`
2. `pop()`
3. `shift()`
4. `unshift()`
5. `splice()`
6. `sort()`
7. `reverse()`

你可以打开控制台，然后对前面例子的 `items` 数组尝试调用变更方法。比如

```
example1.items.push({ message: 'Baz' })
```

#### 替换数组

变更方法，顾名思义，会变更调用了这些方法的原始数组。相比之下，也有非变更方法，例如 `filter()`、`concat()` 和 `slice()`。它们不会变更原始数组，而**总是返回一个新数组**。

当使用非变更方法时，可以用新数组替换旧数组：

```js
example1.items = example1.items.filter(item => item.message.match(/Foo/))
```

你可能认为这将导致 Vue 丢弃现有 DOM 并重新渲染整个列表。幸运的是，事实并非如此。

Vue 为了使得 DOM 元素得到最大范围的重用而实现了一些智能的启发式方法，所以用一个含有相同元素的数组去替换原来的数组是非常高效的操作。

### 9.5 显示过滤/排序后的结果

有时，我们想要显示一个数组经过过滤或排序后的版本，而不实际变更或重置原始数据。在这种情况下，可以创建一个计算属性，来返回过滤或排序后的数组。

```html
<li v-for="n in evenNumbers" :key="n">{{ n }}</li>
```

```js
data() {
  return {
    numbers: [ 1, 2, 3, 4, 5 ]
  }
},
computed: {
  evenNumbers() {
    return this.numbers.filter(number => number % 2 === 0)
  }
}
```

在计算属性不适用的情况下 (例如，在嵌套 `v-for` 循环中) 你可以使用一个方法：

```html
<ul v-for="numbers in sets">
  <li v-for="n in even(numbers)" :key="n">{{ n }}</li>
</ul>
```

```js
data() {
  return {
    sets: [[ 1, 2, 3, 4, 5 ], [6, 7, 8, 9, 10]]
  }
},
methods: {
  even(numbers) {
    return numbers.filter(number => number % 2 === 0)
  }
}
```

### 9.6 在 `v-for` 里使用值的范围

`v-for` 也可以接受整数。在这种情况下，它会把模板重复对应次数。

```html
<div id="range" class="demo">
  <span v-for="n in 10" :key="n">{{ n }} </span>
</div>
```

### 9.7 在 `<template>` 中使用 `v-for`

类似于 `v-if`，你也可以利用带有 `v-for` 的 `<template>` 来循环渲染一段包含多个元素的内容。比如：

```html
<ul>
  <template v-for="item in items" :key="item.msg">
    <li>{{ item.msg }}</li>
    <li class="divider" role="presentation"></li>
  </template>
</ul>
```

### 9.8 `v-for` 与 `v-if` 一同使用

**TIP**

*注意我们**不**推荐在同一元素上使用 `v-if` 和 `v-for`。更多细节可查阅[风格指南](https://v3.cn.vuejs.org/style-guide/#avoid-v-if-with-v-for-essential)。*

当它们处于同一节点，`v-if` 的优先级比 `v-for` 更高，这意味着 `v-if` 将没有权限访问 `v-for` 里的变量：

```html
<!-- This will throw an error because property "todo" is not defined on instance. -->

<li v-for="todo in todos" v-if="!todo.isComplete">
  {{ todo.name }}
</li>
```

可以把 `v-for` 移动到 `<template>` 标签中来修正：

```html
<template v-for="todo in todos" :key="todo.name">
  <li v-if="!todo.isComplete">
    {{ todo.name }}
  </li>
</template>
```

### 9.9 在组件上使用 `v-for`

在自定义组件上，你可以像在任何普通元素上一样使用 `v-for`：

```html
<my-component v-for="item in items" :key="item.id"></my-component>
```

然而，任何数据都不会被自动传递到组件里，因为组件有自己独立的作用域。为了把迭代数据传递到组件里，我们要使用 props：

```html
<my-component
  v-for="(item, index) in items"
  :item="item"
  :index="index"
  :key="item.id"
></my-component>
```

不自动将 `item` 注入到组件里的原因是，这会使得组件与 `v-for` 的运作紧密耦合。明确组件数据的来源能够使组件在其他场合重复使用。

下面是一个简单的 todo 列表的完整例子：

```html
<div id="todo-list-example">
  <form v-on:submit.prevent="addNewTodo">
    <label for="new-todo">Add a todo</label>
    <input
      v-model="newTodoText"
      id="new-todo"
      placeholder="E.g. Feed the cat"
    />
    <button>Add</button>
  </form>
  <ul>
    <todo-item
      v-for="(todo, index) in todos"
      :key="todo.id"
      :title="todo.title"
      @remove="todos.splice(index, 1)"
    ></todo-item>
  </ul>
</div>
```

```js
const app = Vue.createApp({
  data() {
    return {
      newTodoText: '',
      todos: [
        {
          id: 1,
          title: 'Do the dishes'
        },
        {
          id: 2,
          title: 'Take out the trash'
        },
        {
          id: 3,
          title: 'Mow the lawn'
        }
      ],
      nextTodoId: 4
    }
  },
  methods: {
    addNewTodo() {
      this.todos.push({
        id: this.nextTodoId++,
        title: this.newTodoText
      })
      this.newTodoText = ''
    }
  }
})

app.component('todo-item', {
  template: `
    <li>
      {{ title }}
      <button @click="$emit('remove')">Remove</button>
    </li>
  `,
  props: ['title'],
  emits: ['remove']
})

app.mount('#todo-list-example')
```



## 10. 事件处理

### 10.1 监听事件

我们可以使用 `v-on` 指令 (通常缩写为 `@` 符号) 来监听 DOM 事件，并在触发事件时执行一些 JavaScript。

用法为 `v-on:click="methodName"` 或使用快捷方式 `@click="methodName"`

例如：

```html
<div id="basic-event">
  <button @click="counter += 1">Add 1</button>
  <p>The button above has been clicked {{ counter }} times.</p>
</div>
```

```js
Vue.createApp({
  data() {
    return {
      counter: 0
    }
  }
}).mount('#basic-event')
```

### 10.2 事件处理方法

然而许多事件处理逻辑会更为复杂，所以直接把 JavaScript 代码写在 `v-on` 指令中是不可行的。因此 `v-on` 还可以接收一个需要调用的方法名称。

```html
<div id="event-with-method">
  <!-- `greet` 是在下面定义的方法名 -->
  <button @click="greet">Greet</button>
</div>
```

```js
Vue.createApp({
  data() {
    return {
      name: 'Vue.js'
    }
  },
  methods: {
    greet(event) {
      // `methods` 内部的 `this` 指向当前活动实例
      alert('Hello ' + this.name + '!')
      // `event` 是原生 DOM event
      if (event) {
        alert(event.target.tagName)
      }
    }
  }
}).mount('#event-with-method')
```

### 10.3 内联处理器中的方法

除了直接绑定到一个方法，也可以在内联 JavaScript 语句中调用方法：

```html
<div id="inline-handler">
  <button @click="say('hi')">Say hi</button>
  <button @click="say('what')">Say what</button>
</div>
```

```js
Vue.createApp({
  methods: {
    say(message) {
      alert(message)
    }
  }
}).mount('#inline-handler')
```

有时也需要在内联语句处理器中访问原始的 DOM 事件。可以用特殊变量 `$event` 把它传入方法：

```html
<button @click="warn('Form cannot be submitted yet.', $event)">
  Submit
</button>
```

```js
// ...
methods: {
  warn(message, event) {
    // 现在可以访问到原生事件
    if (event) {
      event.preventDefault()
    }
    alert(message)
  }
}
```

### 10.4 多事件处理器

事件处理程序中可以有多个方法，这些方法由逗号运算符分隔：

```html
<!-- 这两个 one() 和 two() 将执行按钮点击事件 -->
<button @click="one($event), two($event)">
  Submit
</button>
```

```js
// ...
methods: {
  one(event) {
    // 第一个事件处理器逻辑...
  },
  two(event) {
   // 第二个事件处理器逻辑...
  }
}
```

### 10.5 事件修饰符

在事件处理程序中调用 `event.preventDefault()` 或 `event.stopPropagation()` 是非常常见的需求。

尽管我们可以在方法中轻松实现这点，但更好的方式是：方法只有纯粹的数据逻辑，而不是去处理 DOM 事件细节。

为了解决这个问题，Vue.js 为 `v-on` 提供了**事件修饰符**。

之前提过，修饰符是由点开头的指令后缀来表示的。

1. `.stop`
2. `.prevent`
3. `.capture`
4. `.self`
5. `.once`
6. `.passive`

```html
<!-- 阻止单击事件继续传播 -->
<a @click.stop="doThis"></a>

<!-- 提交事件不再重载页面 -->
<form @submit.prevent="onSubmit"></form>

<!-- 修饰符可以串联 -->
<a @click.stop.prevent="doThat"></a>

<!-- 只有修饰符 -->
<form @submit.prevent></form>

<!-- 添加事件监听器时使用事件捕获模式 -->
<!-- 即内部元素触发的事件先在此处理，然后才交由内部元素进行处理 -->
<div @click.capture="doThis">...</div>

<!-- 只当在 event.target 是当前元素自身时触发处理函数 -->
<!-- 即事件不是从内部元素触发的 -->
<div @click.self="doThat">...</div>
```

**TIP**

*使用修饰符时，顺序很重要；相应的代码会以同样的顺序产生。因此，用 `v-on:click.prevent.self` 会阻止所有的点击，而 `v-on:click.self.prevent` 只会阻止对元素自身的点击。*

```html
<!-- 点击事件将只会触发一次 -->
<a @click.once="doThis"></a>
```

不像其它只能对原生的 DOM 事件起作用的修饰符，`.once` 修饰符还能被用到自定义的[组件事件](https://v3.cn.vuejs.org/guide/component-custom-events.html)上。

Vue 还对应 [`addEventListener` 中的 passive 选项](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#Parameters)提供了 `.passive` 修饰符。

```html
<!-- 滚动事件的默认行为 (即滚动行为) 将会立即触发   -->
<!-- 而不会等待 `onScroll` 完成                   -->
<!-- 这其中包含 `event.preventDefault()` 的情况   -->
<div @scroll.passive="onScroll">...</div>
```

这个 `.passive` 修饰符尤其能够提升移动端的性能。

**TIP**

*不要把 `.passive` 和 `.prevent` 一起使用，因为 `.prevent` 将会被忽略，同时浏览器可能会向你展示一个警告。请记住，`.passive` 会告诉浏览器你不想阻止事件的默认行为。*

### 10.6 按键修饰符

在监听键盘事件时，我们经常需要检查详细的按键。Vue 允许为 `v-on` 或者 `@` 在监听键盘事件时添加按键修饰符：

```html
<!-- 只有在 `key` 是 `Enter` 时调用 `vm.submit()` -->
<input @keyup.enter="submit" />
```

你可以直接将 [`KeyboardEvent.key`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values) 暴露的任意有效按键名转换为 kebab-case 来作为修饰符。

```html
<input @keyup.page-down="onPageDown" />
```

在上述示例中，处理函数只会在 `$event.key` 等于 `'PageDown'` 时被调用。

#### 按键别名

Vue 为最常用的键提供了别名：

1. `.enter`
2. `.tab`
3. `.delete` (捕获“删除”和“退格”键)
4. `.esc`
5. `.space`
6. `.up`
7. `.down`
8. `.left`
9. `.right`

### 10.7 系统修饰键

可以用如下修饰符来实现仅在按下相应按键时才触发鼠标或键盘事件的监听器。

1. `.ctrl`
2. `.alt`
3. `.shift`
4. `.meta`

**提示**

*注意：在 Mac 系统键盘上，meta 对应 command 键 (⌘)。在 Windows 系统键盘 meta 对应 Windows 徽标键 (⊞)。在 Sun 操作系统键盘上，meta 对应实心宝石键 (◆)。在其他特定键盘上，尤其在 MIT 和 Lisp 机器的键盘、以及其后继产品，比如 Knight 键盘、space-cadet 键盘，meta 被标记为“META”。在 Symbolics 键盘上，meta 被标记为“META”或者“Meta”。*

```html
<!-- Alt + Enter -->
<input @keyup.alt.enter="clear" />

<!-- Ctrl + Click -->
<div @click.ctrl="doSomething">Do something</div>
```

**TIP**

*请注意修饰键与常规按键不同，在和 `keyup` 事件一起用时，事件触发时修饰键必须处于按下状态。换句话说，只有在按住 `ctrl` 的情况下释放其它按键，才能触发 `keyup.ctrl`。而单单释放 `ctrl` 也不会触发事件。*

#### `.exact` 修饰符

`.exact` 修饰符允许你控制由精确的系统修饰符组合触发的事件。

```html
<!-- 即使 Alt 或 Shift 被一同按下时也会触发 -->
<button @click.ctrl="onClick">A</button>

<!-- 有且只有 Ctrl 被按下的时候才触发 -->
<button @click.ctrl.exact="onCtrlClick">A</button>

<!-- 没有任何系统修饰符被按下的时候才触发 -->
<button @click.exact="onClick">A</button>
```

####  鼠标按钮修饰符

1. `.left`
2. `.right`
3. `.middle`

这些修饰符会限制处理函数仅响应特定的鼠标按钮。

### 10.8 为什么在 HTML 中监听事件？

你可能注意到这种事件监听的方式违背了关注点分离 (separation of concern) 这个长期以来的优良传统。

但不必担心，因为所有的 Vue.js 事件处理方法和表达式都严格绑定在当前视图的 ViewModel 上，它不会导致任何维护上的困难。

实际上，使用 `v-on` 或 `@` 有几个好处：

1. 扫一眼 HTML 模板便能轻松定位在 JavaScript 代码里对应的方法。
2. 因为你无须在 JavaScript 里手动绑定事件，你的 ViewModel 代码可以是非常纯粹的逻辑，和 DOM 完全解耦，更易于测试。
3. 当一个 ViewModel 被销毁时，所有的事件处理器都会自动被删除。你无须担心如何清理它们。



## 11. 表单输入绑定

### 11.1 基础用法

你可以用 v-model 指令在表单 `<input>`、`<textarea>` 及 `<select>` 元素上创建双向数据绑定。

它会根据控件类型自动选取正确的方法来更新元素。

尽管有些神奇，但 `v-model` 本质上不过是语法糖。

它负责监听用户的输入事件来更新数据，并在某种极端场景下进行一些特殊处理。

**提示**

*`v-model` 会忽略所有表单元素的 `value`、`checked`、`selected` attribute 的初始值而总是将当前活动实例的数据作为数据来源。你应该通过 JavaScript 在组件的 `data` 选项中声明初始值。*

`v-model` 在内部为不同的输入元素使用不同的 property 并抛出不同的事件：

- text 和 textarea 元素使用 `value` property 和 `input` 事件；
- checkbox 和 radio 使用 `checked` property 和 `change` 事件；
- select 字段将 `value` 作为 prop 并将 `change` 作为事件。

**提示**

*对于需要使用[输入法](https://en.wikipedia.org/wiki/Input_method) (如中文、日文、韩文等) 的语言，你会发现 `v-model` 不会在输入法组织文字过程中得到更新。如果你也想响应这些更新，请使用 `input` 事件监听器和 `value` 绑定，而不是使用 `v-model`。*

####  文本 (Text)

```html
<input v-model="message" placeholder="edit me" />
<p>Message is: {{ message }}</p>
```

#### 多行文本 (Textarea)

```html
<span>Multiline message is:</span>
<p style="white-space: pre-line;">{{ message }}</p>
<br />
<textarea v-model="message" placeholder="add multiple lines"></textarea>
```

在文本区域插值不起作用，应该使用 `v-model` 来代替。

```html
<!-- bad -->
<textarea>{{ text }}</textarea>

<!-- good -->
<textarea v-model="text"></textarea>
```

####  复选框 (Checkbox)

单个复选框，绑定到布尔值：

```html
<input type="checkbox" id="checkbox" v-model="checked" />
<label for="checkbox">{{ checked }}</label>
```

多个复选框，绑定到同一个数组：

```html
<div id="v-model-multiple-checkboxes">
  <input type="checkbox" id="jack" value="Jack" v-model="checkedNames" />
  <label for="jack">Jack</label>
  <input type="checkbox" id="john" value="John" v-model="checkedNames" />
  <label for="john">John</label>
  <input type="checkbox" id="mike" value="Mike" v-model="checkedNames" />
  <label for="mike">Mike</label>
  <br />
  <span>Checked names: {{ checkedNames }}</span>
</div>
```

```js
Vue.createApp({
  data() {
    return {
      checkedNames: []
    }
  }
}).mount('#v-model-multiple-checkboxes')
```

#### 单选框 (Radio)

```html
<div id="v-model-radiobutton">
  <input type="radio" id="one" value="One" v-model="picked" />
  <label for="one">One</label>
  <br />
  <input type="radio" id="two" value="Two" v-model="picked" />
  <label for="two">Two</label>
  <br />
  <span>Picked: {{ picked }}</span>
</div>
```

```js
Vue.createApp({
  data() {
    return {
      picked: ''
    }
  }
}).mount('#v-model-radiobutton')
```

####  选择框 (Select)

单选时：

```html
<div id="v-model-select" class="demo">
  <select v-model="selected">
    <option disabled value="">Please select one</option>
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
  <span>Selected: {{ selected }}</span>
</div>
```

```js
Vue.createApp({
  data() {
    return {
      selected: ''
    }
  }
}).mount('#v-model-select')
```

**注意**

*如果 `v-model` 表达式的初始值未能匹配任何选项，`<select>` 元素将被渲染为“未选中”状态。在 iOS 中，这会使用户无法选择第一个选项。因为这样的情况下，iOS 不会触发 `change` 事件。因此，更推荐像上面这样提供一个值为空的禁用选项。*

多选时 (绑定到一个数组)：

```html
<select v-model="selected" multiple>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
<br />
<span>Selected: {{ selected }}</span>
```

用 `v-for` 渲染的动态选项：

```html
<div id="v-model-select-dynamic" class="demo">
  <select v-model="selected">
    <option v-for="option in options" :value="option.value">
      {{ option.text }}
    </option>
  </select>
  <span>Selected: {{ selected }}</span>
</div>
```

```js
Vue.createApp({
  data() {
    return {
      selected: 'A',
      options: [
        { text: 'One', value: 'A' },
        { text: 'Two', value: 'B' },
        { text: 'Three', value: 'C' }
      ]
    }
  }
}).mount('#v-model-select-dynamic')
```

### 11.2 值绑定

对于单选按钮，复选框及选择框的选项，`v-model` 绑定的值通常是静态字符串 (对于复选框也可以是布尔值)：

```html
<!-- 当选中时，`picked` 为字符串 "a" -->
<input type="radio" v-model="picked" value="a" />

<!-- `toggle` 为 true 或 false -->
<input type="checkbox" v-model="toggle" />

<!-- 当选中第一个选项时，`selected` 为字符串 "abc" -->
<select v-model="selected">
  <option value="abc">ABC</option>
</select>
```

但是有时我们可能想把值绑定到当前活动实例的一个动态 property 上，这时可以用 `v-bind` 实现，此外，使用 `v-bind` 可以将输入值绑定到非字符串。

#### 复选框 (Checkbox)

```html
<input type="checkbox" v-model="toggle" true-value="yes" false-value="no" />
```

```js
// when checked:
vm.toggle === 'yes'
// when unchecked:
vm.toggle === 'no'
```

**提示**

*这里的 `true-value` 和 `false-value` attribute 并不会影响输入控件的 `value` attribute，因为浏览器在提交表单时并不会包含未被选中的复选框。如果要确保表单中这两个值中的一个能够被提交，(即“yes”或“no”)，请换用单选按钮。*

####  单选框 (Radio)

```html
<input type="radio" v-model="pick" v-bind:value="a" />
```

```js
// 当选中时
vm.pick === vm.a
```

####  选择框选项 (Select Options)

```html
<select v-model="selected">
  <!-- 内联对象字面量 -->
  <option :value="{ number: 123 }">123</option>
</select>
```

```js
// 当被选中时
typeof vm.selected // => 'object'
vm.selected.number // => 123
```

### 11.3 修饰符

#### `.lazy`

在默认情况下，`v-model` 在每次 `input` 事件触发后将输入框的值与数据进行同步 (除了[上述](https://v3.cn.vuejs.org/guide/forms.html#vmodel-ime-tip)输入法组织文字时)。

你可以添加 `lazy` 修饰符，从而转为在 `change` 事件_之后_进行同步：

```html
<!-- 在“change”时而非“input”时更新 -->
<input v-model.lazy="msg" />
```

#### `.number`

如果想自动将用户的输入值转为数值类型，可以给 `v-model` 添加 `number` 修饰符：

```html
<input v-model.number="age" type="number" />
```

这通常很有用，因为即使在 `type="number"` 时，HTML 输入元素的值也总会返回字符串。

如果这个值无法被 `parseFloat()` 解析，则会返回原始的值。

#### `.trim`

如果要自动过滤用户输入的首尾空白字符，可以给 `v-model` 添加 `trim` 修饰符：

```html
<input v-model.trim="msg" />
```

### 11.4 在组件上使用 `v-model`

HTML 原生的输入元素类型并不总能满足需求。幸好，Vue 的组件系统允许你创建具有完全自定义行为且可复用的输入组件。

这些输入组件甚至可以和 `v-model` 一起使用！



## 12. 组件基础

### 12.1 基本实例

```js
// 创建一个Vue 应用
const app = Vue.createApp({})

// 定义一个名为 button-counter 的新全局组件
app.component('button-counter', {
  data() {
    return {
      count: 0
    }
  },
  template: `
    <button @click="count++">
      You clicked me {{ count }} times.
    </button>`
})
```

组件是带有名称的可复用实例，在这个例子中是 `<button-counter>`。我们可以把这个组件作为一个根实例中的自定义元素来使用：

```html
<div id="components-demo">
  <button-counter></button-counter>
</div>
```

```js
app.mount('#components-demo')
```

因为组件是可复用的组件实例，所以它们与根实例接收相同的选项，例如 `data`、`computed`、`watch`、`methods` 以及生命周期钩子等。

### 12.2 组件的复用

你可以将组件进行任意次数的复用：

```html
<div id="components-demo">
  <button-counter></button-counter>
  <button-counter></button-counter>
  <button-counter></button-counter>
</div>
```

注意当点击按钮时，每个组件都会各自独立维护它的 `count`。因为你每用一次组件，就会有一个它的新**实例**被创建。

### 12.3 组件的组织

通常一个应用会以一棵嵌套的组件树的形式来组织：

<img src="https://v3.cn.vuejs.org/images/components.png" alt="Component Tree" style="zoom: 50%;" />

例如，你可能会有页头、侧边栏、内容区等组件，每个组件又包含了其它的像导航链接、博文之类的组件。

为了能在模板中使用，这些组件必须先注册以便 Vue 能够识别。

这里有两种组件的注册类型：**全局注册**和**局部注册**。

至此，我们的组件都只是通过 `component` 方法全局注册的：

```js
const app = Vue.createApp({})

app.component('my-component-name', {
  // ... 选项 ...
})
```

全局注册的组件可以在应用中的任何组件的模板中使用。

### 12.4 通过 Prop 向子组件传递数据

Prop 是你可以在组件上注册的一些自定义 attribute。

为了给博文组件传递一个标题，我们可以用 `props` 选项将其包含在该组件可接受的 prop 列表中：

```js
const app = Vue.createApp({})

app.component('blog-post', {
  props: ['title'],
  template: `<h4>{{ title }}</h4>`
})

app.mount('#blog-post-demo')
```

当一个值被传递给一个 prop attribute 时，它就成为该组件实例中的一个 property。

该 property 的值可以在模板中访问，就像任何其他组件 property 一样。

一个组件默认可以拥有任意数量的 prop，无论任何值都可以传递给 prop。

```html
<div id="blog-post-demo" class="demo">
  <blog-post title="My journey with Vue"></blog-post>
  <blog-post title="Blogging with Vue"></blog-post>
  <blog-post title="Why Vue is so fun"></blog-post>
</div>
```

然而在一个典型的应用中，你可能在 `data` 里有一个博文的数组：

```js
const App = {
  data() {
    return {
      posts: [
        { id: 1, title: 'My journey with Vue' },
        { id: 2, title: 'Blogging with Vue' },
        { id: 3, title: 'Why Vue is so fun' }
      ]
    }
  }
}

const app = Vue.createApp(App)

app.component('blog-post', {
  props: ['title'],
  template: `<h4>{{ title }}</h4>`
})

app.mount('#blog-posts-demo')
```

并想要为每篇博文渲染一个组件：

```html
<div id="blog-posts-demo">
  <blog-post
    v-for="post in posts"
    :key="post.id"
    :title="post.title"
  ></blog-post>
</div>
```

如上所示，你会发现我们可以使用 `v-bind` 来动态传递 prop。这在你一开始不清楚要渲染的具体内容，是非常有用的。

### 12.5 监听子组件事件

我们在开发 `<blog-post>` 组件时，它的一些功能可能需要与父级组件进行沟通。

例如我们可能会引入一个辅助功能来放大博文的字号，同时让页面的其它部分保持默认的字号。

在其父组件中，我们可以通过添加一个 `postFontSize` 数据 property 来支持这个功能：

```js
const App = {
  data() {
    return {
      posts: [
        /* ... */
      ],
      postFontSize: 1
    }
  }
}
```

它可以在模板中用来控制所有博文的字号：

```html
<div id="blog-posts-events-demo">
  <div :style="{ fontSize: postFontSize + 'em' }">
    <blog-post
      v-for="post in posts"
      :key="post.id"
      :title="post.title"
    ></blog-post>
  </div>
</div>
```

现在我们在每篇博文正文之前添加一个按钮来放大字号：

```js
app.component('blog-post', {
  props: ['title'],
  template: `
    <div class="blog-post">
      <h4>{{ title }}</h4>
      <button>
        Enlarge text
      </button>
    </div>
  `
})
```

组件实例提供了一个自定义事件的系统来解决这个问题。父级组件可以像处理原生 DOM 事件一样通过 `v-on` 或 `@` 监听子组件实例的任意事件：

```html
<blog-post ... @enlarge-text="postFontSize += 0.1"></blog-post>
```

同时子组件可以通过调用内建的 [**$emit** 方法](https://v3.cn.vuejs.org/api/instance-methods.html#emit)并传入事件名称来触发一个事件：

```html
<button @click="$emit('enlargeText')">
  Enlarge text
</button>
```

我们可以在组件的 `emits` 选项中列出已抛出的事件：

```js
app.component('blog-post', {
  props: ['title'],
  emits: ['enlargeText']
})
```

这将允许我们检查组件抛出的所有事件，还可以选择 [validate them](https://v3.cn.vuejs.org/guide/component-custom-events.html#validate-emitted-events)。

#### 使用事件抛出一个值

有的时候用一个事件来抛出一个特定的值是非常有用的。

例如我们可能想让 `<blog-post>` 组件决定它的文本要放大多少。这时可以使用 `$emit` 的第二个参数来提供这个值：

```html
<button @click="$emit('enlargeText', 0.1)">
  Enlarge text
</button>
```

然后当在父级组件监听这个事件的时候，我们可以通过 `$event` 访问到被抛出的这个值：

```html
<blog-post ... @enlarge-text="postFontSize += $event"></blog-post>
```

或者，如果这个事件处理函数是一个方法：

```html
<blog-post ... @enlarge-text="onEnlargeText"></blog-post>
```

那么这个值将会作为第一个参数传入这个方法：

```js
methods: {
  onEnlargeText(enlargeAmount) {
    this.postFontSize += enlargeAmount
  }
}
```

#### 在组件上使用 v-model

自定义事件也可以用于创建支持 `v-model` 的自定义输入组件。记住：

```html
<input v-model="searchText" />
```

等价于：

```html
<input :value="searchText" @input="searchText = $event.target.value" />
```

当用在组件上时，`v-model` 则会这样：

```html
<custom-input
  :model-value="searchText"
  @update:model-value="searchText = $event"
></custom-input>
```

**WARNING**

*请注意，我们在这里使用的是 `model-value`，因为我们使用的是 DOM 模板中的 kebab-case。你可以在 [DOM Template Parsing Caveats](https://v3.cn.vuejs.org/guide/component-basics.html#dom-template-parsing-caveats) 部分找到关于 kebab cased 和 camelCased 属性的详细说明*

为了让它正常工作，这个组件内的 `<input>` 必须：

- 将其 `value` attribute 绑定到一个名叫 `modelValue` 的 prop 上
- 在其 `input` 事件被触发时，将新的值通过自定义的 `update:modelValue` 事件抛出

写成代码之后是这样的：

```js
app.component('custom-input', {
  props: ['modelValue'],
  emits: ['update:modelValue'],
  template: `
    <input
      :value="modelValue"
      @input="$emit('update:modelValue', $event.target.value)"
    >
  `
})
```

现在 `v-model` 就应该可以在这个组件上完美地工作起来了：

```html
<custom-input v-model="searchText"></custom-input>
```

在该组件中实现 `v-model` 的另一种方法是使用 `computed` property 的功能来定义 getter 和 setter。

- `get` 方法应返回 `modelValue` property，
- `set` 方法应该触发相应的事件。

```js
app.component('custom-input', {
  props: ['modelValue'],
  emits: ['update:modelValue'],
  template: `
    <input v-model="value">
  `,
  computed: {
    value: {
      get() {
        return this.modelValue
      },
      set(value) { 
        this.$emit('update:modelValue', value)
      }
    }
  }
})
```

### 12.6 通过插槽分发内容

和 HTML 元素一样，我们经常需要向一个组件传递内容，这可以通过使用 Vue 的自定义 `<slot>` 元素来实现：

```js
app.component('alert-box', {
  template: `
    <div class="demo-alert-box">
      <strong>Error!</strong>
      <slot></slot>
    </div>
  `
})
```

如你所见，我们使用 `<slot>` 作为我们想要插入内容的占位符

### 12.7 动态组件

有的时候，在不同组件之间进行动态切换是非常有用的，比如在一个多标签的界面里：

上述内容可以通过 Vue 的 `<component>` 元素加一个特殊的 `is` attribute 来实现：

```html
<!-- 组件会在 `currentTabComponent` 改变时改变 -->
<component :is="currentTabComponent"></component>
```

在上述示例中，`currentTabComponent` 可以包括

- 已注册组件的名字，或
- 一个组件的选项对象

你也可以使用 `is` attribute 来创建常规的 HTML 元素。

### 12.8 解析 DOM 模板时的注意事项

有些 HTML 元素，诸如 `<ul>`、`<ol>`、`<table>` 和 `<select>`，对于哪些元素可以出现在其内部是有严格限制的。

而有些元素，诸如 `<li>`、`<tr>` 和 `<option>`，只能出现在其它某些特定的元素内部。

这会导致我们使用这些有约束条件的元素时遇到一些问题。例如：

```html
<table>
  <blog-post-row></blog-post-row>
</table>
```

这个自定义组件 `<blog-post-row>` 会被作为无效的内容提升到外部，并导致最终渲染结果出错。

我们可以使用特殊的 `v-is` 指令作为一个变通的办法：

```html
<table>
  <tr v-is="'blog-post-row'"></tr>
</table>
```

**WARNING**

*`v-is` 值应为 JavaScript 字符串文本：*

```html
<!-- 错误的，这样不会渲染任何东西 -->
<tr v-is="blog-post-row"></tr>

<!-- 正确的 -->
<tr v-is="'blog-post-row'"></tr>
```

另外，HTML attribute 名不区分大小写，因此浏览器将所有大写字符解释为小写。

这意味着当你在 DOM 模板中使用时，驼峰 prop 名称和 event 处理器参数需要使用它们的 kebab-cased (横线字符分隔) 等效值：

```js
//  在JavaScript中的驼峰

app.component('blog-post', {
  props: ['postTitle'],
  template: `
    <h3>{{ postTitle }}</h3>
  `
})
```

```html
<!-- 在HTML则是横线字符分割 -->

<blog-post post-title="hello!"></blog-post>
```

需要注意的是**如果我们从以下来源使用模板的话，这条限制是\*不存在\*的**：

- 字符串模板 (例如：`template: '...'`)
- [单文件组件](https://v3.cn.vuejs.org/guide/single-file-component.html)
- `<script type="text/x-template">`





# 基础   结束

## 总结







# Vue3 Api

https://v3.cn.vuejs.org/api/application-config.html



- 应用配置
  - [errorHandler](https://v3.cn.vuejs.org/api/application-config.html#errorhandler)
  - [warnHandler](https://v3.cn.vuejs.org/api/application-config.html#warnhandler)
  - [globalProperties](https://v3.cn.vuejs.org/api/application-config.html#globalproperties)
  - [isCustomElement](https://v3.cn.vuejs.org/api/application-config.html#iscustomelement)
  - [optionMergeStrategies](https://v3.cn.vuejs.org/api/application-config.html#optionmergestrategies)
  - [performance](https://v3.cn.vuejs.org/api/application-config.html#performance)

- 应用 API
  - [component](https://v3.cn.vuejs.org/api/application-api.html#component)
  - [config](https://v3.cn.vuejs.org/api/application-api.html#config)
  - [directive](https://v3.cn.vuejs.org/api/application-api.html#directive)
  - [mixin](https://v3.cn.vuejs.org/api/application-api.html#mixin)
  - [mount](https://v3.cn.vuejs.org/api/application-api.html#mount)
  - [provide](https://v3.cn.vuejs.org/api/application-api.html#provide)
  - [unmount](https://v3.cn.vuejs.org/api/application-api.html#unmount)
  - [use](https://v3.cn.vuejs.org/api/application-api.html#use)

- 全局 API
  - [createApp](https://v3.cn.vuejs.org/api/global-api.html#createapp)
  - [h](https://v3.cn.vuejs.org/api/global-api.html#h)
  - [defineComponent](https://v3.cn.vuejs.org/api/global-api.html#definecomponent)
  - [defineAsyncComponent](https://v3.cn.vuejs.org/api/global-api.html#defineasynccomponent)
  - [resolveComponent](https://v3.cn.vuejs.org/api/global-api.html#resolvecomponent)
  - [resolveDynamicComponent](https://v3.cn.vuejs.org/api/global-api.html#resolvedynamiccomponent)
  - [resolveDirective](https://v3.cn.vuejs.org/api/global-api.html#resolvedirective)
  - [withDirectives](https://v3.cn.vuejs.org/api/global-api.html#withdirectives)
  - [createRenderer](https://v3.cn.vuejs.org/api/global-api.html#createrenderer)
  - [nextTick](https://v3.cn.vuejs.org/api/global-api.html#nexttick)
  - [mergeProps](https://v3.cn.vuejs.org/api/global-api.html#mergeprops)

- [选项](https://v3.cn.vuejs.org/api/options-api)
  - Data
    - [data](https://v3.cn.vuejs.org/api/options-data.html#data-2)
    - [props](https://v3.cn.vuejs.org/api/options-data.html#props)
    - [computed](https://v3.cn.vuejs.org/api/options-data.html#computed)
    - [methods](https://v3.cn.vuejs.org/api/options-data.html#methods)
    - [watch](https://v3.cn.vuejs.org/api/options-data.html#watch)
    - [emits](https://v3.cn.vuejs.org/api/options-data.html#emits)

- - DOM
    - [template](https://v3.cn.vuejs.org/api/options-dom.html#template)
    - [render](https://v3.cn.vuejs.org/api/options-dom.html#render)

- - 生命周期钩子
    - [beforeCreate](https://v3.cn.vuejs.org/api/options-lifecycle-hooks.html#beforecreate)
    - [created](https://v3.cn.vuejs.org/api/options-lifecycle-hooks.html#created)
    - [beforeMount](https://v3.cn.vuejs.org/api/options-lifecycle-hooks.html#beforemount)
    - [mounted](https://v3.cn.vuejs.org/api/options-lifecycle-hooks.html#mounted)
    - [beforeUpdate](https://v3.cn.vuejs.org/api/options-lifecycle-hooks.html#beforeupdate)
    - [updated](https://v3.cn.vuejs.org/api/options-lifecycle-hooks.html#updated)
    - [activated](https://v3.cn.vuejs.org/api/options-lifecycle-hooks.html#activated)
    - [deactivated](https://v3.cn.vuejs.org/api/options-lifecycle-hooks.html#deactivated)
    - [beforeUnmount](https://v3.cn.vuejs.org/api/options-lifecycle-hooks.html#beforeunmount)
    - [unmounted](https://v3.cn.vuejs.org/api/options-lifecycle-hooks.html#unmounted)
    - [errorCaptured](https://v3.cn.vuejs.org/api/options-lifecycle-hooks.html#errorcaptured)
    - [renderTracked](https://v3.cn.vuejs.org/api/options-lifecycle-hooks.html#rendertracked)
    - [renderTriggered](https://v3.cn.vuejs.org/api/options-lifecycle-hooks.html#rendertriggered)

- - 选项/资源
    - [directives](https://v3.cn.vuejs.org/api/options-assets.html#directives)
    - [components](https://v3.cn.vuejs.org/api/options-assets.html#components)

- - 组合
    - [mixins](https://v3.cn.vuejs.org/api/options-composition.html#mixins)
    - [extends](https://v3.cn.vuejs.org/api/options-composition.html#extends)
    - [provide / inject](https://v3.cn.vuejs.org/api/options-composition.html#provide-inject)
    - [setup](https://v3.cn.vuejs.org/api/options-composition.html#setup)

- - 杂项
    - [name](https://v3.cn.vuejs.org/api/options-misc.html#name)
    - [delimiters](https://v3.cn.vuejs.org/api/options-misc.html#delimiters)
    - [inheritAttrs](https://v3.cn.vuejs.org/api/options-misc.html#inheritattrs)

- 实例 property
  - [$data](https://v3.cn.vuejs.org/api/instance-properties.html#data)
  - [$props](https://v3.cn.vuejs.org/api/instance-properties.html#props)
  - [$el](https://v3.cn.vuejs.org/api/instance-properties.html#el)
  - [$options](https://v3.cn.vuejs.org/api/instance-properties.html#options)
  - [$parent](https://v3.cn.vuejs.org/api/instance-properties.html#parent)
  - [$root](https://v3.cn.vuejs.org/api/instance-properties.html#root)
  - [$slots](https://v3.cn.vuejs.org/api/instance-properties.html#slots)
  - [$refs](https://v3.cn.vuejs.org/api/instance-properties.html#refs)
  - [$attrs](https://v3.cn.vuejs.org/api/instance-properties.html#attrs)

- 实例方法
  - [$watch](https://v3.cn.vuejs.org/api/instance-methods.html#watch)
  - [$emit](https://v3.cn.vuejs.org/api/instance-methods.html#emit)
  - [$forceUpdate](https://v3.cn.vuejs.org/api/instance-methods.html#forceupdate)
  - [$nextTick](https://v3.cn.vuejs.org/api/instance-methods.html#nexttick)

- 指令
  - [v-text](https://v3.cn.vuejs.org/api/directives.html#v-text)
  - [v-html](https://v3.cn.vuejs.org/api/directives.html#v-html)
  - [v-show](https://v3.cn.vuejs.org/api/directives.html#v-show)
  - [v-if](https://v3.cn.vuejs.org/api/directives.html#v-if)
  - [v-else](https://v3.cn.vuejs.org/api/directives.html#v-else)
  - [v-else-if](https://v3.cn.vuejs.org/api/directives.html#v-else-if)
  - [v-for](https://v3.cn.vuejs.org/api/directives.html#v-for)
  - [v-on](https://v3.cn.vuejs.org/api/directives.html#v-on)
  - [v-bind](https://v3.cn.vuejs.org/api/directives.html#v-bind)
  - [v-model](https://v3.cn.vuejs.org/api/directives.html#v-model)
  - [v-slot](https://v3.cn.vuejs.org/api/directives.html#v-slot)
  - [v-pre](https://v3.cn.vuejs.org/api/directives.html#v-pre)
  - [v-cloak](https://v3.cn.vuejs.org/api/directives.html#v-cloak)
  - [v-once](https://v3.cn.vuejs.org/api/directives.html#v-once)
  - [v-is](https://v3.cn.vuejs.org/api/directives.html#v-is)

- 特殊指令
  - [key](https://v3.cn.vuejs.org/api/special-attributes.html#key)
  - [ref](https://v3.cn.vuejs.org/api/special-attributes.html#ref)
  - [is](https://v3.cn.vuejs.org/api/special-attributes.html#is)

- 内置组件
  - [component](https://v3.cn.vuejs.org/api/built-in-components.html#component)
  - [transition](https://v3.cn.vuejs.org/api/built-in-components.html#transition)
  - [transition-group](https://v3.cn.vuejs.org/api/built-in-components.html#transition-group)
  - [keep-alive](https://v3.cn.vuejs.org/api/built-in-components.html#keep-alive)
  - [slot](https://v3.cn.vuejs.org/api/built-in-components.html#slot)
  - [teleport](https://v3.cn.vuejs.org/api/built-in-components.html#teleport)

- [响应性 API](https://v3.cn.vuejs.org/api/reactivity-api)
  - 响应性基础 API
    - [reactive](https://v3.cn.vuejs.org/api/basic-reactivity.html#reactive)
    - [readonly](https://v3.cn.vuejs.org/api/basic-reactivity.html#readonly)
    - [isProxy](https://v3.cn.vuejs.org/api/basic-reactivity.html#isproxy)
    - [isReactive](https://v3.cn.vuejs.org/api/basic-reactivity.html#isreactive)
    - [isReadonly](https://v3.cn.vuejs.org/api/basic-reactivity.html#isreadonly)
    - [toRaw](https://v3.cn.vuejs.org/api/basic-reactivity.html#toraw)
    - [markRaw](https://v3.cn.vuejs.org/api/basic-reactivity.html#markraw)
    - [shallowReactive](https://v3.cn.vuejs.org/api/basic-reactivity.html#shallowreactive)
    - [shallowReadonly](https://v3.cn.vuejs.org/api/basic-reactivity.html#shallowreadonly)

- - Refs
    - [ref](https://v3.cn.vuejs.org/api/refs-api.html#ref)
    - [unref](https://v3.cn.vuejs.org/api/refs-api.html#unref)
    - [toRef](https://v3.cn.vuejs.org/api/refs-api.html#toref)
    - [toRefs](https://v3.cn.vuejs.org/api/refs-api.html#torefs)
    - [isRef](https://v3.cn.vuejs.org/api/refs-api.html#isref)
    - [customRef](https://v3.cn.vuejs.org/api/refs-api.html#customref)
    - [shallowRef](https://v3.cn.vuejs.org/api/refs-api.html#shallowref)
    - [triggerRef](https://v3.cn.vuejs.org/api/refs-api.html#triggerref)

- - Computed 与 watch
    - [computed](https://v3.cn.vuejs.org/api/computed-watch-api.html#computed)
    - [watchEffect](https://v3.cn.vuejs.org/api/computed-watch-api.html#watcheffect)
    - watch
      - [侦听一个单一源](https://v3.cn.vuejs.org/api/computed-watch-api.html#侦听一个单一源)
      - [侦听多个源](https://v3.cn.vuejs.org/api/computed-watch-api.html#侦听多个源)
      - [与 watchEffect 相同的行为](https://v3.cn.vuejs.org/api/computed-watch-api.html#与-watcheffect-相同的行为)

[组合式 API](https://v3.cn.vuejs.org/api/composition-api.html)

- [setup](https://v3.cn.vuejs.org/api/composition-api.html#setup)
- [生命周期钩子](https://v3.cn.vuejs.org/api/composition-api.html#生命周期钩子)
- [Provide / Inject](https://v3.cn.vuejs.org/api/composition-api.html#provide-inject)
- [getCurrentInstance](https://v3.cn.vuejs.org/api/composition-api.html#getcurrentinstance)























































































