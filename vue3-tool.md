# **工具**

## 1. 单文件组件

### 1.1 介绍

在很多 Vue 项目中，我们使用 `app.component` 来定义全局组件，紧接着用 `app.mount('#app')` 在每个页面内指定一个容器元素。

这对于中小型项目非常有效，在这些项目里 JavaScript 只被用来增强特定的视图。但当在更复杂的项目中，或者你的前端完全由 JavaScript 驱动的时候，下面这些缺点将变得非常明显：

- **全局定义 (Global definitions)** 强制要求每个 component 中的命名不得重复；
- **字符串模板 (String templates)** 缺乏语法高亮，在 HTML 有多行的时候，需要用到丑陋的 `\`；
- **不支持 CSS (No CSS support)** 意味着当 HTML 和 JavaScript 组件化时，CSS 明显被遗漏；
- **没有构建步骤 (No build step)** 限制只能使用 HTML 和 ES5 JavaScript，而不能使用预处理器，如 Pug (曾经的 Jade) 和 Babel。

所有这些都可以通过扩展名为 `.vue` 的 **single-file components (单文件组件)** 来解决，并且还可以使用 webpack 或 Browserify 等构建工具。

这是一个文件名为 `Hello.vue` 的简单实例：

```
<template>
  <p>{{ greeting }} World!</p>
</template>

<script>
module.exports = {
  data() {
    return {
      greeting: "Hello"
    };
  }
};
</script>

<style scoped>
p {
  font-size: 2em;
  text-align: center;
}
</style>
```

现在我们获得：

- [完整语法高亮](https://github.com/vuejs/awesome-vue#source-code-editing)
- [CommonJS 模块](https://webpack.js.org/concepts/modules/#what-is-a-webpack-module)
- [组件作用域的 CSS](https://vue-loader.vuejs.org/en/features/scoped-css.html)

正如我们说过的，我们可以使用预处理器来构建简洁和功能更丰富的组件，比如 Pug，Babel (with ES2015 modules)，和 Stylus。

```
<template lang="pug">
p {{ greeting }} World!
other-component
</template>

<script>
import OtherComponent from "./OtherComponent.vue"

export default {
  components: {
    OtherComponent
  },

  data() {
    return {
      greeting: "Hello"
    }
  }
}
</script>

<style lang="stylus">
p
  font-size 2em
  text-align center
</style>
```

这些特定的语言只是例子，你可以只是简单地使用 Babel，TypeScript，SCSS，PostCSS 或者其他任何能够帮助你提高生产力的预处理器。如果搭配 `vue-loader` 使用 webpack，它也能为 CSS Modules 提供头等支持。

####  怎么看待关注点分离？

值得注意的是，**关注点分离不等于文件类型分离**。在现代 UI 开发中，我们已经发现相比于把代码库分离成三个大的层次并将其相互交织起来，把它们划分为松散耦合的组件再将其组合起来更合理一些。在一个组件里，其模板、逻辑和样式是内部耦合的，并且把他们搭配在一起实际上使得组件更加内聚且更可维护。

即便你不喜欢单文件组件，你仍然可以把 JavaScript、CSS 分离成独立的文件然后做到热重载和预编译。

```html
<!-- my-component.vue -->
<template>
  <div>This will be pre-compiled</div>
</template>
<script src="./my-component.js"></script>
<style src="./my-component.css"></style>
```

### 1.2 起步

#### 在线演示

如果你希望深入了解并开始使用单文件组件，请来 CodeSandbox [看看这个简单的 todo 应用](https://codesandbox.io/s/vue-todo-list-app-with-single-file-component-vzkl3?file=/src/App.vue)。

#### 针对刚接触 JavaScript 模块开发系统的用户

有了 `.vue` 组件，我们就进入了高阶 JavaScript 应用领域。如果你没有准备好的话，意味着还需要学会使用一些附加的工具：

- **Node 包管理器 (npm)**： 阅读 [Getting Started guide](https://docs.npmjs.com/packages-and-modules/getting-packages-from-the-registry) 中关于如何从注册地 (registry) 获取包的章节。
- **现代 JavaScript 与 ES2015/16**：阅读 Babel 的 [Learn ES2015 guide](https://babeljs.io/docs/en/learn)。你不需要立刻记住每一个方法，但是你可以保留这个页面以便后期参考。

在你花一天时间了解这些资源之后，我们建议你参考 [Vue CLI](https://cli.vuejs.org/)。只要遵循指示，你就能很快地运行一个带有 `.vue` 组件、ES2015、webpack 和热重载 (hot-reloading) 的 Vue 项目！

#### 针对高阶用户

CLI 会为你搞定大多数工具的配置问题，同时也支持细粒度自定义[配置项](https://cli.vuejs.org/config/)。

有时你会想从零搭建你自己的构建工具，这时你需要通过 [Vue Loader](https://vue-loader.vuejs.org/) 手动配置 webpack。关于学习更多 webpack 的内容，请查阅[其官方文档](https://webpack.js.org/configuration/)和 [Webpack Academy](https://webpack.academy/p/the-core-concepts)。

#### 使用 rollup 构建

在开发第三方库的时候，大多数时候我们都希望以一种允许类库用户 [tree shake](https://webpack.js.org/guides/tree-shaking/) 的方式来构建它。为了实现 tree-shaking，我们需要构建 `esm` 模块。由于 webpack 以及 vue-cli 都不支持构建 `esm` 模块，我们需要依靠 [rollup](https://rollupjs.org/)。

- ##### **安装 Rollup**

我们需要安装 Rollup 和一些依赖：

```bash
npm install --save-dev rollup @rollup/plugin-commonjs rollup-plugin-vue
```

这些都是我们需要用来编译 `esm` 模块中的代码的最小化的 rollup 插件。我们可能还需要添加 [rollup-plugin-babel](https://github.com/rollup/plugins/tree/master/packages/babel) 来移植它们的代码，如果我们需要与库捆绑一起的依赖关系，还需要添加 [node-resolve](https://github.com/rollup/plugins/tree/master/packages/node-resolve)。

- ##### **配置 Rollup**

要配置 Rollup 进行构建，我们需要在项目的根目录创建一个 `rollup.config.js` 文件。

```bash
touch rollup.config.js
```

创建文件后，选择需要的编辑器打开并添加以下代码：

```javascript
// 导入我们的第三方插件
import commonjs from 'rollup-plugin-commonjs'
import VuePlugin from 'rollup-plugin-vue'
import pkg from './package.json' // 导入我们的 package.json 文件，重新使用命名。

export default {
  // 这是一个包含所有导出的组件/函数的文件。
  input: 'src/index.js',
  // 这是一个输出格式的数组
  output: [
    {
      file: pkg.module, // 我们的 ESM 库的名词
      format: 'esm', // 选择的格式
      sourcemap: true // 要求 rollup 包含 sourcemap
    }
  ],
  // 这是我们所包含插件的数组
  plugins: [commonjs(), VuePlugin()],
  // 要求 rollup 不要将 Vue 捆绑在库中。
  external: ['vue']
}
```

- #####  **配置 package.json**

为了利用我们新创建的 `esm` 模块，我们需要在 `package.json` 文件中添加一些字段。

```json
 "scripts": {
   ...
   "build": "rollup -c rollup.config.js",
   ...
 },
 "module": "dist/my-library-name.esm.js",
 "files": [
   "dist/",
 ],
```

在这里，我们要说明的是：

- 如何打包
- 我们要在依赖中捆绑哪些文件
- 什么文件代表我们的 `esm` 模块

- #####  **打包 `umd` 和 `cjs` 模块**

要构建 `umd` 和 `cjs` 模块，我们可以简单地在 `rollup.config.js` 和 `package.json` 中添加几行配置。

- #####  rollup.config.js

```javascript
output: [
  ...{
    file: pkg.main,
    format: 'cjs',
    sourcemap: true
  },
  {
    file: pkg.unpkg,
    format: 'umd',
    name: 'MyLibraryName',
    sourcemap: true,
    globals: {
      vue: 'Vue'
    }
  }
]
```

- #####  package.json

```json
"module": "dist/my-library-name.esm.js",
"main": "dist/my-library-name.cjs.js",
"unpkg": "dist/my-library-name.global.js",
```



## 2. 测试

### 2.1 介绍

当构建可靠的应用时，测试在个人或团队构建新特性、重构代码、修复 bug 等工作中扮演了关键的角色。尽管测试的流派有很多，它们在 web 应用这个领域里主要有三大类：

- 单元测试
- 组件测试
- 端到端 (E2E，end-to-end) 测试

本章节致力于引导大家了解测试的生态系统的并为 Vue 应用或组件库选择适合的工具。

### 2.2 单元测试

####  2.2.1 介绍

单元测试允许你将独立单元的代码进行隔离测试，其目的是为开发者提供对代码的信心。通过编写细致且有意义的测试，你能够有信心在构建新特性或重构已有代码的同时，保持应用的功能和稳定。

为一个 Vue 应用做单元测试并没有和为其它类型的应用做测试有什么明显的区别。

#### 2.2.2 选择框架

因为单元测试的建议通常是框架无关的，所以下面只是当你在评估应用的单元测试工具时需要的一些基本指引。

**一流的错误报告**

当测试失败时，提供有用的错误信息对于单元测试框架来说至关重要。这是断言库应尽的职责。一个具有高质量错误信息的断言能够最小化调试问题所需的时间。除了简单地告诉你什么测试失败了，断言库还应额外提供上下文以及测试失败的原因，例如预期结果 vs 实际得到的结果。

一些诸如 Jest 这样的单元测试框架会包含断言库。另一些诸如 Mocha 需要你单独安装断言库 (通常会用 Chai)。

 **活跃的社区和团队**

因为主流的单元测试框架都是开源的，所以对于一些旨在长期维护其测试且确保项目本身保持活跃的团队来说，拥有一个活跃的社区是至关重要的。额外的好处是，在任何时候遇到问题时，一个活跃的社区会为你提供更多的支持。

#### 2.2.3 框架

尽管生态系统里有很多工具，这里我们列出一些在 Vue 生态系统中常用的单元测试工具。

**Jest**

Jest 是一个专注于简易性的 JavaScript 测试框架。一个其独特的功能是可以为测试生成快照 (snapshot)，以提供另一种验证应用单元的方法。

**资料：**

- [Jest 官网](https://jestjs.io/zh-Hans/)
- [Vue CLI 官方插件 - Jest](https://cli.vuejs.org/core-plugins/unit-jest.html)

**Mocha**

Mocha 是一个专注于灵活性的 JavaScript 测试框架。因为其灵活性，它允许你选择不同的库来满足诸如侦听 (如 Sinon) 和断言 (如 Chai) 等其它常见的功能。另一个 Mocha 独特的功能是它不止可以在 Node.js 里运行测试，还可以在浏览器里运行测试。

**资料：**

- [Mocha 官网](https://mochajs.org/)
- [Vue CLI 官方插件 - Mocha](https://cli.vuejs.org/core-plugins/unit-mocha.html)

### 2.3 组件测试

#### 2.3.1 介绍

测试大多数 Vue 组件时都必须将它们挂载到 DOM (虚拟或真实) 上，才能完全断言它们正在工作。这是另一个与框架无关的概念。因此组件测试框架的诞生，是为了让用户能够以可靠的方式完成这项工作，同时还提供了 Vue 特有的诸如对 Vuex、Vue Router 和其他 Vue 插件的集成的便利性。

#### 2.3.2 选择框架

以下章节提供了在评估最适合你的应用的组件测试框架时需要记住的事项。

 **与 Vue 生态系统的最佳兼容性**

毋容置疑，最重要的标准之一就是组件测试库应该尽可能与 Vue 生态系统兼容。虽然这看起来很全面，但需要记住的一些关键集成领域包括单文件组件 (SFC)、Vuex、Vue Router 以及应用所依赖的任何其他特定于 Vue 的插件。

**一流的错误报告**

当测试失败时，提供有用的错误日志以最小化调试问题所需的时间对于组件测试框架来说至关重要。除了简单地告诉你什么测试失败了，他们还应额外提供上下文以及测试失败的原因，例如预期结果 vs 实际得到的结果。

#### 2.3.3 推荐

**Vue Testing Library (@testing-library/vue)**

Vue Testing Library 是一组专注于测试组件而不依赖实现细节的工具。由于在设计时就充分考虑了可访问性，它采用的方案也使重构变得轻而易举。

它的指导原则是，与软件使用方式相似的测试越多，它们提供的可信度就越高。

**资料：**

- [Vue Testing Library 官网](https://testing-library.com/docs/vue-testing-library/intro)

**Vue Test Utils**

Vue Test Utils 是官方的偏底层的组件测试库，它是为用户提供对 Vue 特定 API 的访问而编写的。如果你对测试 Vue 应用不熟悉，我们建议你使用 Vue Testing Library，它是 Vue Test Utils 的抽象。

**资源：**

- [Vue Test Utils 官方文档](https://vue-test-utils.vuejs.org/)
- [Vue 测试指南](https://lmiller1990.github.io/vue-testing-handbook/zh-CN/) by Lachlan Miller



### 2.4 端到端 (E2E) 测试

#### 2.4.1 介绍

虽然单元测试为开发者提供了一定程度的信心，但是单元测试和组件测试在部署到生产环境时提供应用整体覆盖的能力是有限的。因此，端到端测试可以说从应用最重要的方面进行测试覆盖：当用户实际使用应用时会发生什么。

换句话说，端到端测试验证应用中的所有层。这不仅包括你的前端代码，还包括所有相关的后端服务和基础设施，它们更能代表你的用户所处的环境。通过测试用户操作如何影响应用，端到端测试通常是提高应用是否正常运行的信心的关键。

#### 2.4.2 选择框架

虽然 web 上的端到端测试因不可信赖 (片面的) 测试和减慢开发过程而得到负面的声誉，但现代端到端工具在创建更可靠的、交互的和实用的测试方面取得了长足进步。在选择端到端测试框架时，以下章节在你为应用选择测试框架时提供了一些指导。

**跨浏览器测试**

端到端测试的一个主要优点是它能够跨多个浏览器测试应用。尽管 100% 的跨浏览器覆盖看上去很诱人，但需要注意的是，因为持续运行这些跨浏览器测试需要额外的时间和机器消耗，它会降低团队的资源回报。因此，在选择应用需要的跨浏览器测试数量时，必须注意这种权衡。

**TIP**

*针对浏览器特定问题的一个最新进展是，针对不常用的浏览器 (如：< IE11、旧版 Safari 等) 使用应用监视和错误报告工具 (如：Sentry、LogRocket 等)。*

**更快的反馈路径**

端到端测试和开发的主要问题之一是运行整个套件需要很长时间。通常，这只在持续集成和部署 (CI/CD) 管道中完成。现代的端到端测试框架通过添加类似并行化的特性来帮助解决这个问题，这使得 CI/CD 管道的运行速度通常比以前快。此外，在本地开发时，有选择地为正在处理的页面运行单个测试的能力，同时还提供测试的热重载，将有助于提高开发者的工作流程和工作效率。

 **一流的调试经验**

虽然开发者传统上依赖于在终端窗口中扫描日志来帮助确定测试中出了什么问题，但现代端到端测试框架允许开发者利用他们已经熟悉的工具，例如浏览器开发工具。

#### 2.4.3 推荐

虽然生态系统中有许多工具，但以下是一些 Vue.js 生态系统中常用的端到端测试框架。

 **Cypress.io**

Cypress.io 是一个测试框架，旨在通过使开发者能够可靠地测试他们的应用，同时提供一流的开发者体验，来提高开发者的生产率。

**资料：**

- [Cypress 官网](https://www.cypress.io/)
- [Vue CLI 官方插件 - Cypress](https://cli.vuejs.org/core-plugins/e2e-cypress.html)
- [Cypress Testing Library](https://github.com/testing-library/cypress-testing-library)

**Nightwatch.js**

Nightwatch.js 是一个端到端测试框架，可用于测试 web 应用和网站，以及 Node.js 单元测试和集成测试。

**资料：**

- [Nightwatch 官网](https://nightwatchjs.org/)
- [Vue CLI 官方插件 - Nightwatch](https://cli.vuejs.org/core-plugins/e2e-nightwatch.html)

**Puppeteer**

Puppeteer 是一个 Node.js 库，它提供高阶 API 来控制浏览器，并可以与其他测试运行程序 (例如 Jest) 配对来测试应用。

**资料：**

- [Puppeteer 官网](https://pptr.dev/)

**TestCafe**

TestCafe 是一个基于端到端的 Node.js 框架，旨在提供简单的设置，以便开发者能够专注于创建易于编写和可靠的测试。

**资料：**

- [TestCafe 官网](https://devexpress.github.io/testcafe/)



## 3. TypeScript 支持

```
[Vue CLI](https://cli.vuejs.org/) 提供内置的 TypeScript 工具支持。
```

### 3.1 NPM 包中的官方声明

随着应用的增长，静态类型系统可以帮助防止许多潜在的运行时错误，这就是为什么 Vue 3 是用 TypeScript 编写的。这意味着在 Vue 中使用 TypeScript 不需要任何其他工具——它具有一流的公民支持。

### 3.2 推荐配置

```js
// tsconfig.json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext",
    // 这样就可以对 `this` 上的数据属性进行更严格的推断`
    "strict": true,
    "jsx": "preserve",
    "moduleResolution": "node"
  }
}
```

请注意，必须包含 `strict: true` (或至少包含 `noImplicitThis: true`，它是 `strict` 标志的一部分) 才能在组件方法中利用 `this` 的类型检查，否则它总是被视为 `any` 类型。

参见 [TypeScript 编译选项文档](https://www.typescriptlang.org/docs/handbook/compiler-options.html)查看更多细节。

### 3.3 Webpack 配置

如果你使用自定义 Webpack 配置，需要配置 ' ts-loader ' 来解析 vue 文件里的 `<script lang="ts">` 代码块：

```js
// webpack.config.js
module.exports = {
  ...
  module: {
    rules: [
      {
        test: /\.tsx?$/,
        loader: 'ts-loader',
        options: {
          appendTsSuffixTo: [/\.vue$/],
        },
        exclude: /node_modules/,
      },
      {
        test: /\.vue$/,
        loader: 'vue-loader',
      }
      ...
```

### 3.4 开发工具

#### 项目创建

[Vue CLI](https://github.com/vuejs/vue-cli) 可以生成使用 TypeScript 的新项目，开始：

```bash
# 1. Install Vue CLI, 如果尚未安装
npm install --global @vue/cli@next

# 2. 创建一个新项目, 选择 "Manually select features" 选项
vue create my-project-name

# 3. 如果已经有一个不存在TypeScript的 Vue CLI项目，请添加适当的 Vue CLI插件：
vue add typescript
```

请确保组件的 `script` 部分已将语言设置为 TypeScript：

```html
<script lang="ts">
  ...
</script>
```

#### 编辑器支持

对于使用 TypeScript 开发 Vue 应用程序，我们强烈建议使用 [Visual Studio Code](https://code.visualstudio.com/)，它为 TypeScript 提供了很好的开箱即用支持。如果你使用的是[单文件组件](https://v3.cn.vuejs.org/guide/single-file-component.html) (SFCs)，那么可以使用很棒的 [Vetur extension](https://github.com/vuejs/vetur)，它在 SFCs 中提供了 TypeScript 推理和许多其他优秀的特性。

[WebStorm](https://www.jetbrains.com/webstorm/) 还为 TypeScript 和 Vue 提供现成的支持。

### 3.5 定义 Vue 组件

要让 TypeScript 正确推断 Vue 组件选项中的类型，需要使用 `defineComponent` 全局方法定义组件：

```ts
import { defineComponent } from 'vue'

const Component = defineComponent({
  // 已启用类型推断
})
```

### 3.6 与 Options API 一起使用

TypeScript 应该能够在不显式定义类型的情况下推断大多数类型。例如，如果有一个具有数字 `count` property 的组件，如果试图对其调用特定于字符串的方法，则会出现错误：

```ts
const Component = defineComponent({
  data() {
    return {
      count: 0
    }
  },
  mounted() {
    const result = this.count.split('') // => Property 'split' does not exist on type 'number'
  }
})
```

如果你有一个复杂的类型或接口，你可以使用 [type assertion](https://www.typescriptlang.org/docs/handbook/basic-types.html#type-assertions) 对其进行强制转换：

```ts
interface Book {
  title: string
  author: string
  year: number
}

const Component = defineComponent({
  data() {
    return {
      book: {
        title: 'Vue 3 Guide',
        author: 'Vue Team',
        year: 2020
      } as Book
    }
  }
})
```

#### 注解返回类型

由于 Vue 声明文件的循环特性，TypeScript 可能难以推断 computed 的类型。因此，你可能需要注解计算属性的返回类型。

```ts
import { defineComponent } from 'vue'

const Component = defineComponent({
  data() {
    return {
      message: 'Hello!'
    }
  },
  computed: {
    // 需要注解
    greeting(): string {
      return this.message + '!'
    }

    // 在使用 setter 进行计算时，需要对 getter 进行注解
    greetingUppercased: {
      get(): string {
        return this.greeting.toUpperCase();
      },
      set(newValue: string) {
        this.message = newValue.toUpperCase();
      },
    },
  }
})
```

####  注解 Props

Vue 对定义了 `type` 的 prop 执行运行时验证。要将这些类型提供给 TypeScript，我们需要使用 `PropType` 强制转换构造函数：

```ts
import { defineComponent, PropType } from 'vue'

interface Book {
  title: string
  author: string
  year: number
}

const Component = defineComponent({
  props: {
    name: String,
    success: { type: String },
    callback: {
      type: Function as PropType<() => void>
    },
    book: {
      type: Object as PropType<Book>,
      required: true
    }
  }
})
```

**WARNING**

*由于 TypeScript 中的[设计限制](https://github.com/microsoft/TypeScript/issues/38845)，当它涉及到为了对函数表达式进行类型推理，你必须注意对象和数组的 `validators` 和 `default` 值：*

```ts
import { defineComponent, PropType } from 'vue'

interface Book {
  title: string
  year?: number
}

const Component = defineComponent({
  props: {
    bookA: {
      type: Object as PropType<Book>,
      // 请务必使用箭头函数
      default: () => ({
        title: 'Arrow Function Expression'
      }),
      validator: (book: Book) => !!book.title
    },
    bookB: {
      type: Object as PropType<Book>,
      // 或者提供一个明确的 this 参数
      default(this: void) {
        return {
          title: 'Function Expression'
        }
      },
      validator(this: void, book: Book) {
        return !!book.title
      }
    }
  }
})
```

#### 注解 emit

我们可以为触发的事件注解一个有效载荷。另外，所有未声明的触发事件在调用时都会抛出一个类型错误。

```ts
const Component = defineComponent({
  emits: {
    addBook(payload: { bookName: string }) {
      // perform runtime 验证
      return payload.bookName.length > 0
    }
  },
  methods: {
    onSubmit() {
      this.$emit('addBook', {
        bookName: 123 // 类型错误！
      })
      this.$emit('non-declared-event') // 类型错误！
    }
  }
})
```

### 3.7 与组合式 API 一起使用

在 `setup()` 函数中，不需要将类型传递给 `props` 参数，因为它将从 `props` 组件选项推断类型。

```ts
import { defineComponent } from 'vue'

const Component = defineComponent({
  props: {
    message: {
      type: String,
      required: true
    }
  },

  setup(props) {
    const result = props.message.split('') // 正确, 'message' 被声明为字符串
    const filtered = props.message.filter(p => p.value) // 将引发错误: Property 'filter' does not exist on type 'string'
  }
})
```

#### 类型声明 `refs`

Refs 根据初始值推断类型：

```ts
import { defineComponent, ref } from 'vue'

const Component = defineComponent({
  setup() {
    const year = ref(2020)

    const result = year.value.split('') // => Property 'split' does not exist on type 'number'
  }
})
```

有时我们可能需要为 ref 的内部值指定复杂类型。我们可以在调用 ref 重写默认推理时简单地传递一个泛型参数：

```ts
const year = ref<string | number>('2020') // year's type: Ref<string | number>

year.value = 2020 // ok!
```

**TIP**

*如果泛型的类型未知，建议将 `ref` 转换为 `Ref<T>`。*

#### 类型声明 `reactive`

当声明类型 `reactive` property，我们可以使用接口：

```ts
import { defineComponent, reactive } from 'vue'

interface Book {
  title: string
  year?: number
}

export default defineComponent({
  name: 'HelloWorld',
  setup() {
    const book = reactive<Book>({ title: 'Vue 3 Guide' })
    // or
    const book: Book = reactive({ title: 'Vue 3 Guide' })
    // or
    const book = reactive({ title: 'Vue 3 Guide' }) as Book
  }
})
```

#### 类型声明 `computed`

计算值将根据返回值自动推断类型

```ts
import { defineComponent, ref, computed } from 'vue'

export default defineComponent({
  name: 'CounterButton',
  setup() {
    let count = ref(0)

    // 只读
    const doubleCount = computed(() => count.value * 2)

    const result = doubleCount.value.split('') // => Property 'split' does not exist on type 'number'
  }
})
```



## 4. 移动端

### 4.1 介绍

虽然 Vue.js 本身并不支持移动应用开发，但是有很多解决方案可以用 Vue.js 创建原生 iOS 和 Android 应用。

### 4.2 混合应用开发

#### Capacitor

[Capacitor](https://capacitorjs.com/) 是一个来自 [Ionic Team](https://ionic.io/) 的项目，通过提供跨多个平台运行的 API，开发者可以使用单个代码库构建原生 iOS、Android 和 PWA 应用。

**资源**

- [Capacitor + Vue.js Guide](https://capacitorjs.com/solution/vue)

#### NativeScript

[NativeScript](https://www.nativescript.org/) 使用已熟悉的 Web 技能为跨平台（真正的原生）移动应用提供支持。两者结合在一起是开发沉浸式移动体验的绝佳选择。

**资源**

- [NativeScript + Vue.js Guide](https://nativescript.org/vue/)



## 5. 生产环境部署

**INFO**

*如果你使用 [Vue CLI](https://cli.vuejs.org/)，下面的大多数提示都是默认启用的。此部分仅当你使用自定义构建设置时才相关。*

### 5.1 开启生产环境模式

在开发期间，Vue 提供了许多警告，以帮助你处理常见的错误和隐患。但是，这些警告字符串在生产环境中会变得无意义，并且会增大应用程序的负担。此外，有些警告检查的运行时成本很小，可以在[生产环境模式](https://cli.vuejs.org/guide/mode-and-env.html#modes)中避免。

#### 不使用构建工具

如果你正在使用完整的构建，即直接通过脚本标签引入 Vue，而不使用构建工具，那么请确保在生产环境中使用压缩版。这可以在[安装指南](https://v3.cn.vuejs.org/guide/installation.html#cdn)中找到。

#### 使用构建工具

当使用 Webpack 或 Browserify 这样的构建工具时，生产环境模式将由 Vue 的源代码中的 `process.env.NODE_ENV` 决定，默认为开发模式。这两种构建工具都提供了重写这个变量以启用 Vue 的生产模式的方法，并且在构建过程中警告将被压缩工具删除。Vue CLI 已经为你预先配置了这个，不过了解它的工作原理会更好：

**Webpack**

在 Webpack 4+，你可以使用 `mode` 选项：

```js
module.exports = {
  mode: 'production'
}
```

 **Browserify**

- 将当前的环境变量 `NODE_ENV` 设置为 `"production"` 作为运行的打包命令。它告诉 `vueify` 避免引入热重载和开发相关的代码。

- 将一个全局的 [envify](https://github.com/hughsk/envify) 转换应用到你的包中。这使得压缩工具可以删除包裹在环境变量条件块中的Vue源代码中的所有警告。例如:

  ```bash
  NODE_ENV=production browserify -g envify -e main.js | uglifyjs -c -m > build.js
  ```

- 或者，利用 Gulp 使用 [envify](https://github.com/hughsk/envify)：

  ```js
  // Use the envify custom module to specify environment variables
  const envify = require('envify/custom')
  
  browserify(browserifyOptions)
    .transform(vueify)
    .transform(
      // Required in order to process node_modules files
      { global: true },
      envify({ NODE_ENV: 'production' })
    )
    .bundle()
  ```

- 或者，利用 Grunt 和 [grunt-browserify](https://github.com/jmreidy/grunt-browserify) 使用 [envify](https://github.com/hughsk/envify)：

  ```js
  // Use the envify custom module to specify environment variables
  const envify = require('envify/custom')
  
  browserify: {
    dist: {
      options: {
        // Function to deviate from grunt-browserify's default order
        configure: (b) =>
          b
            .transform('vueify')
            .transform(
              // Required in order to process node_modules files
              { global: true },
              envify({ NODE_ENV: 'production' })
            )
            .bundle()
      }
    }
  }
  ```

**Rollup**

使用 [@rollup/plugin-replace](https://github.com/rollup/plugins/tree/master/packages/replace):

```js
const replace = require('@rollup/plugin-replace')

rollup({
  // ...
  plugins: [
    replace({
      'process.env.NODE_ENV': JSON.stringify( 'production' )
    })
  ]
}).then(...)
```

### 5.2 预编译模板

当使用 DOM 内模板或 JavaScript 内模板字符串时，将动态地执行从模板到渲染函数的编译。在大多数情况下，这已经足够快了，但是如果应用程序对性能敏感，最好避免这样做。

预编译模板最简单的方法是使用[单文件组件](https://v3.cn.vuejs.org/guide/single-file-component.html)——相关的构建设置自动为你执行预编译，所以构建代码包含已经编译的渲染函数，而不是原始的模板字符串。

如果你正在使用 Webpack，并且更喜欢将 JavaScript 和模板文件分开，你可以使用 [vue-template-loader](https://github.com/ktsn/vue-template-loader)，它还可以在构建步骤中将模板文件转换为 JavaScript 渲染函数。

### 5.3 提取组件CSS

当使用单文件组件时，组件内部的 CSS 会通过 JavaScript 以 `<style>` 标签的形式被动态注入。这有一个小的运行时成本，如果你使用服务器端渲染，它将导致“无样式内容的闪现” 。将所有组件的 CSS 提取到同一个文件中可以避免这些问题，还可以更好地压缩和缓存 CSS。

参考各自的构建工具文档，看看它是如何做的:

- [Webpack + vue-loader](https://vue-loader.vuejs.org/en/configurations/extract-css.html) (`vue-cli` webpack 模板已经预先配置了这个)
- [Browserify + vueify](https://github.com/vuejs/vueify#css-extraction)
- [Rollup + rollup-plugin-vue](https://rollup-plugin-vue.vuejs.org/)

### 5.4 跟踪运行时错误

如果在组件渲染期间发生运行时错误，它将被传递到全局的 `app.config.errorHandler` 配置函数，如果它已经被设置。将这个钩子与错误跟踪服务如 [Sentry](https://sentry.io/) 一起使用可能是一个好主意，它为 Vue 提供了[一个官方集成](https://sentry.io/for/vue/)。





















































































































































# **工具**    结束