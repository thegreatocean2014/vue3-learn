# **规模化**   



## 1. 路由

### 1.1 官方 Router

对于大多数单页面应用，都推荐使用官方支持的 [vue-router 库](https://github.com/vuejs/vue-router)。更多细节可以移步 [vue-router 文档](https://next.router.vuejs.org/)



### 1.2 从零开始简单的路由

如果你只需要非常简单的路由而不想引入一个功能完整的路由库，可以像这样动态渲染一个页面级的组件：

```js
const { createApp, h } = Vue

const NotFoundComponent = { template: '<p>Page not found</p>' }
const HomeComponent = { template: '<p>Home page</p>' }
const AboutComponent = { template: '<p>About page</p>' }

const routes = {
  '/': HomeComponent,
  '/about': AboutComponent
}

const SimpleRouter = {
  data: () => ({
    currentRoute: window.location.pathname
  }),

  computed: {
    CurrentComponent() {
      return routes[this.currentRoute] || NotFoundComponent
    }
  },

  render() {
    return h(this.CurrentComponent)
  }
}

createApp(SimpleRouter).mount('#app')
```

结合 [HTML5 History API](https://developer.mozilla.org/zh-CN/docs/Web/API/History_API/Working_with_the_History_API)，你可以建立一个麻雀虽小但是五脏俱全的客户端路由器。为了获得更多的实践，可以直接查看[实例应用](https://github.com/phanan/vue-3.0-simple-routing-example)。



### 1.3 整合第三方路由

如果你有更加偏爱的第三方路由，如 [Page.js](https://github.com/visionmedia/page.js) 或者 [Director](https://github.com/flatiron/director)，整合起来也[一样简单](https://github.com/phanan/vue-3.0-simple-routing-example/compare/master...pagejs)。这里有一个使用了 Page.js 的[完整示例](https://github.com/phanan/vue-3.0-simple-routing-example/tree/pagejs)。



## 2. 状态管理

### 2.1 类 Flux 状态管理的官方实现

由于状态零散地分布在许多组件和组件之间的交互中，大型应用复杂度也经常逐渐增长。为了解决这个问题，Vue 提供 [Vuex](https://next.vuex.vuejs.org/)：我们有受到 Elm 启发的状态管理库。vuex 甚至集成到 [vue-devtools](https://github.com/vuejs/vue-devtools)，无需配置即可进行[时光旅行调试 (time travel debugging)](https://raw.githubusercontent.com/vuejs/vue-devtools/master/media/demo.gif)。

#### 给 React 开发者的参考信息

如果你是来自 React 的开发者，可能会对 Vuex 和 [Redux](https://github.com/reactjs/redux) 间的差异表示关注，Redux 是 React 生态环境中最流行的 Flux 实现。Redux 事实上无法感知视图层，所以它能够轻松的通过一些[简单绑定](https://classic.yarnpkg.com/en/packages?q=redux vue&p=1)和 Vue 一起使用。Vuex 区别在于它是一个专门为 Vue 应用所设计。这使得它能够更好地和 Vue 进行整合，同时提供简洁的 API 和更好的开发体验。

### 2.2 从零打造简单状态管理

经常被忽略的是，Vue 应用中响应式 `data` 对象的实际来源——当访问数据对象时，一个组件实例只是简单的代理访问。所以，如果你有一处需要被多个实例间共享的状态，你可以使用一个 [reactive](https://v3.cn.vuejs.org/guide/reactivity-fundamentals.html#声明响应式状态) 方法让对象作为响应式对象。

```js
const { createApp, reactive } = Vue
const sourceOfTruth = reactive({
  message: 'Hello'
})

const appA = createApp({
  data() {
    return sourceOfTruth
  }
}).mount('#app-a')

const appB = createApp({
  data() {
    return sourceOfTruth
  }
}).mount('#app-b')
```

```html
<div id="app-a">App A: {{ message }}</div>

<div id="app-b">App B: {{ message }}</div>
```

现在当 `sourceOfTruth` 发生变更，`appA` 和 `appB` 都将自动地更新它们的视图。虽然现在我们有了一个真实数据来源，但调试将是一场噩梦。应用的任何部分都可以随时更改任何数据，而不会留下变更过的记录。

```js
const appB = createApp({
  data() {
    return sourceOfTruth
  },
  mounted() {
    sourceOfTruth.message = 'Goodbye' // both apps will render 'Goodbye' message now
  }
}).mount('#app-b')
```

为了解决这个问题，可以采用一个简单的 **store 模式**：

```js
const store = {
  debug: true,

  state: reactive({
    message: 'Hello!'
  }),

  setMessageAction(newValue) {
    if (this.debug) {
      console.log('setMessageAction triggered with', newValue)
    }

    this.state.message = newValue
  },

  clearMessageAction() {
    if (this.debug) {
      console.log('clearMessageAction triggered')
    }

    this.state.message = ''
  }
}
```

需要注意，所有 store 中 state 的变更，都放置在 store 自身的 action 中去管理。这种集中式状态管理能够被更容易地理解哪种类型的变更将会发生，以及它们是如何被触发。当错误出现时，现在也会有一个 log 记录 bug 之前发生了什么。

此外，每个实例/组件仍然可以拥有和管理自己的私有状态：

```html
<div id="app-a">{{sharedState.message}}</div>

<div id="app-b">{{sharedState.message}}</div>
```

```js
const appA = createApp({
  data() {
    return {
      privateState: {},
      sharedState: store.state
    }
  },
  mounted() {
    store.setMessageAction('Goodbye!')
  }
}).mount('#app-a')

const appB = createApp({
  data() {
    return {
      privateState: {},
      sharedState: store.state
    }
  }
}).mount('#app-b')
```



## 3. 服务端渲染

### 3.1 SSR 完全指南

我们创建了一份完整的构建 Vue 服务端渲染应用的指南。这份指南非常深入，适合已经熟悉 Vue、webpack 和 Node.js 开发的开发者阅读。请移步 [ssr.vuejs.org](https://ssr.vuejs.org/zh/)。

### 3.2 Nuxt.js

从头搭建一个服务端渲染的应用是相当复杂的。幸运的是，我们有一个优秀的社区项目 [Nuxt.js](https://zh.nuxtjs.org/) 让这一切变得非常简单。Nuxt.js 是一个基于 Vue 生态的更高层次的框架，为开发服务端渲染的 Vue 应用提供了极其便利的开发体验。更酷的是，你甚至可以用它来做为静态站生成器。我们强烈建议尝试一下。

### 3.3 Quasar Framework SSR + PWA

[Quasar Framework](https://quasar.dev/) 可以通过其一流的构建系统、合理的配置和开发者扩展性生成 (可选地和 PWA 互通的) SSR 应用，让你的想法的设计和构建变得轻而易举。你可以在服务端挑选执行超过上百款遵循“Material Design 2.0”的组件，并且在浏览器端可用。你甚至可以管理网站的 `<meta>` 标签。Quasar 是一个基于 Node.js 和 webpack 的开发环境，它可以通过一套代码完成 SPA、PWA、SSR、Electron、Capacitor 和 Cordova 应用的快速开发。





















































































































































































# **规模化**      结束