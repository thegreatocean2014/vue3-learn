# Vuex

## 1. **Introduction**

### 1.1 What is Vuex?

**NOTE**

*This is the docs for Vuex 4, which works with Vue 3. If you're looking for docs for Vuex 3, which works with Vue 2, [please check it out here](https://vuex.vuejs.org/).*

Vuex is a **state management pattern + library** for Vue.js applications. It serves as a centralized store for all the components in an application, with rules ensuring that the state can only be mutated in a predictable fashion.

#### What is a "State Management Pattern"?

Let's start with a simple Vue counter app:

```
const Counter = {
  // state
  data () {
    return {
      count: 0
    }
  },
  // view
  template: `
    <div>{{ count }}</div>
  `,
  // actions
  methods: {
    increment () {
      this.count++
    }
  }
}

createApp(Counter).mount('#app')
```

It is a self-contained app with the following parts:

- The **state**, the source of truth that drives our app;
- The **view**, a declarative mapping of the **state**;
- The **actions**, the possible ways the state could change in reaction to user inputs from the **view**.

This is a simple representation of the concept of "one-way data flow":

<img src="./img/flow.png" alt="./img/flow.png" style="zoom:50%;" />



<img src="https://next.vuex.vuejs.org/flow.png" alt="./img/flow.png" style="zoom:50%;" />

However, the simplicity quickly breaks down when we have **multiple components that share a common state**:

- Multiple views may depend on the same piece of state.
- Actions from different views may need to mutate the same piece of state.

For problem one, passing props can be tedious for deeply nested components, and simply doesn't work for sibling components. For problem two, we often find ourselves resorting to solutions such as reaching for direct parent/child instance references or trying to mutate and synchronize multiple copies of the state via events. Both of these patterns are brittle and quickly lead to unmaintainable code.

So why don't we extract the shared state out of the components, and manage it in a global singleton? With this, our component tree becomes a big "view", and any component can access the state or trigger actions, no matter where they are in the tree!

By defining and separating the concepts involved in state management and enforcing rules that maintain independence between views and states, we give our code more structure and maintainability.

This is the basic idea behind Vuex, inspired by [Flux](https://facebook.github.io/flux/docs/overview), [Redux](http://redux.js.org/) and [The Elm Architecture](https://guide.elm-lang.org/architecture/). Unlike the other patterns, Vuex is also a library implementation tailored specifically for Vue.js to take advantage of its granular reactivity system for efficient updates.

If you want to learn Vuex in an interactive way you can check out this [Vuex course on Scrimba](https://scrimba.com/g/gvuex), which gives you a mix of screencast and code playground that you can pause and play around with anytime.

<img src="./img/vuex.png" alt="./img/flow.png" style="zoom:100%;" />

<img src="https://next.vuex.vuejs.org/vuex.png" alt="./img/flow.png" style="zoom:50%;" />

#### When Should I Use It?

Vuex helps us deal with shared state management with the cost of more concepts and boilerplate. It's a trade-off between short term and long term productivity.

If you've never built a large-scale SPA and jump right into Vuex, it may feel verbose and daunting. That's perfectly normal - if your app is simple, you will most likely be fine without Vuex. A simple [store pattern](https://v3.vuejs.org/guide/state-management.html#simple-state-management-from-scratch) may be all you need. But if you are building a medium-to-large-scale SPA, chances are you have run into situations that make you think about how to better handle state outside of your Vue components, and Vuex will be the natural next step for you. There's a good quote from Dan Abramov, the author of Redux:

*Flux libraries are like glasses: you’ll know when you need them.*



### 1.2 Installation

#### Direct Download / CDN

https://unpkg.com/vuex@4

[Unpkg.com](https://unpkg.com/) provides NPM-based CDN links. The above link will always point to the latest release on NPM. You can also use a specific version/tag via URLs like `https://unpkg.com/vuex@4.0.0/dist/vuex.global.js`.

Include `vuex` after Vue and it will install itself automatically:

```
<script src="/path/to/vue.js"></script>
<script src="/path/to/vuex.js"></script>
```

#### NPM

```
npm install vuex@next --save
```

#### Yarn

```
yarn add vuex@next --save
```

####  Promise

Vuex requires [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises). If your supporting browsers do not implement Promise (e.g. IE), you can use a polyfill library, such as [es6-promise](https://github.com/stefanpenner/es6-promise).

You can include it via CDN:

```
<script src="https://cdn.jsdelivr.net/npm/es6-promise@4/dist/es6-promise.auto.js"></script>
```

Then `window.Promise` will be available automatically.

If you prefer using a package manager such as NPM or Yarn, install it with the following commands:

```
npm install es6-promise --save # NPM
yarn add es6-promise # Yarn
```

Furthermore, add the below line into anywhere in your code before using Vuex:

```
import 'es6-promise/auto'
```

#### Dev Build

You will have to clone directly from GitHub and build `vuex` yourself if you want to use the latest dev build.

```
git clone https://github.com/vuejs/vuex.git node_modules/vuex
cd node_modules/vuex
yarn
yarn build
```

### 1.3 Getting Started

At the center of every Vuex application is the **store**. A "store" is basically a container that holds your application **state**. There are two things that make a Vuex store different from a plain global object:

1. Vuex stores are reactive. When Vue components retrieve state from it, they will reactively and efficiently update if the store's state changes.
2. You cannot directly mutate the store's state. The only way to change a store's state is by explicitly **committing mutations**. This ensures every state change leaves a track-able record, and enables tooling that helps us better understand our applications.

#### The Simplest Store

**NOTE**

*We will be using ES2015 syntax for code examples for the rest of the docs. If you haven't picked it up, [you should](https://babeljs.io/docs/learn-es2015/)!*

After [installing](https://next.vuex.vuejs.org/installation.html) Vuex, let's create a store. It is pretty straightforward - just provide an initial state object, and some mutations:

```
import { createApp } from 'vue'
import { createStore } from 'vuex'

// Create a new store instance.
const store = createStore({
  state () {
    return {
      count: 0
    }
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})

const app = createApp({ /* your root component */ })

// Install the store instance as a plugin
app.use(store)
```

Now, you can access the state object as `store.state`, and trigger a state change with the `store.commit` method:

```
store.commit('increment')

console.log(store.state.count) // -> 1
```

In a Vue component, you can access the store as `this.$store`. Now we can commit a mutation using a component method:

```
methods: {
  increment() {
    this.$store.commit('increment')
    console.log(this.$store.state.count)
  }
}
```

Again, the reason we are committing a mutation instead of changing `store.state.count` directly, is because we want to explicitly track it. This simple convention makes your intention more explicit, so that you can reason about state changes in your app better when reading the code. In addition, this gives us the opportunity to implement tools that can log every mutation, take state snapshots, or even perform time travel debugging.

Using store state in a component simply involves returning the state within a computed property, because the store state is reactive. Triggering changes simply means committing mutations in component methods.

Next, we will discuss each core concept in much finer details, starting with [State](https://next.vuex.vuejs.org/guide/state.html).



## 2. **Core Concepts**

### 2.1 State

#### Single State Tree

Vuex uses a **single state tree** - that is, this single object contains all your application level state and serves as the "single source of truth." This also means usually you will have only one store for each application. A single state tree makes it straightforward to locate a specific piece of state, and allows us to easily take snapshots of the current app state for debugging purposes.

The single state tree does not conflict with modularity - in later chapters we will discuss how to split your state and mutations into sub modules.

The data you store in Vuex follows the same rules as the `data` in a Vue instance, ie the state object must be plain. **See also:** [Vue#data](https://v3.vuejs.org/api/options-data.html#data-2).

#### Getting Vuex State into Vue Components

So how do we display state inside the store in our Vue components? Since Vuex stores are reactive, the simplest way to "retrieve" state from it is simply returning some store state from within a [computed property](https://vuejs.org/guide/computed.html):

```
// let's create a Counter component
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return store.state.count
    }
  }
}
```

Whenever `store.state.count` changes, it will cause the computed property to re-evaluate, and trigger associated DOM updates.

However, this pattern causes the component to rely on the global store singleton. When using a module system, it requires importing the store in every component that uses store state, and also requires mocking when testing the component.

Vuex "injects" the store into all child components from the root component through Vue's plugin system, and will be available on them as `this.$store`. Let's update our `Counter` implementation:

```
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return this.$store.state.count
    }
  }
}
```

#### The `mapState` Helper

When a component needs to make use of multiple store state properties or getters, declaring all these computed properties can get repetitive and verbose. To deal with this we can make use of the `mapState` helper which generates computed getter functions for us, saving us some keystrokes:

```
// in full builds helpers are exposed as Vuex.mapState
import { mapState } from 'vuex'

export default {
  // ...
  computed: mapState({
    // arrow functions can make the code very succinct!
    count: state => state.count,

    // passing the string value 'count' is same as `state => state.count`
    countAlias: 'count',

    // to access local state with `this`, a normal function must be used
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  })
}
```

We can also pass a string array to `mapState` when the name of a mapped computed property is the same as a state sub tree name.

```
computed: mapState([
  // map this.count to store.state.count
  'count'
])
```

#### Object Spread Operator

Note that `mapState` returns an object. How do we use it in combination with other local computed properties? Normally, we'd have to use a utility to merge multiple objects into one so that we can pass the final object to `computed`. However with the [object spread operator](https://github.com/tc39/proposal-object-rest-spread), we can greatly simplify the syntax:

```
computed: {
  localComputed () { /* ... */ },
  // mix this into the outer object with the object spread operator
  ...mapState({
    // ...
  })
}
```

#### Components Can Still Have Local State

Using Vuex doesn't mean you should put **all** the state in Vuex. Although putting more state into Vuex makes your state mutations more explicit and debuggable, sometimes it could also make the code more verbose and indirect. If a piece of state strictly belongs to a single component, it could be just fine leaving it as local state. You should weigh the trade-offs and make decisions that fit the development needs of your app.

### 2.2  Getters

Sometimes we may need to compute derived state based on store state, for example filtering through a list of items and counting them:

```
computed: {
  doneTodosCount () {
    return this.$store.state.todos.filter(todo => todo.done).length
  }
}
```

If more than one component needs to make use of this, we have to either duplicate the function, or extract it into a shared helper and import it in multiple places - both are less than ideal.

Vuex allows us to define "getters" in the store. You can think of them as computed properties for stores.

**WARNING**

*As of Vue 3.0, the getter's result is **not cached** as the computed property does. This is a known issue that requires Vue 3.1 to be released. You can learn more at [PR #1878](https://github.com/vuejs/vuex/pull/1883).*

Getters will receive the state as their 1st argument:

```
const store = createStore({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    doneTodos (state) {
      return state.todos.filter(todo => todo.done)
    }
  }
})
```

#### Property-Style Access

The getters will be exposed on the `store.getters` object, and you access values as properties:

```
store.getters.doneTodos // -> [{ id: 1, text: '...', done: true }]
```

Getters will also receive other getters as the 2nd argument:

```
getters: {
  // ...
  doneTodosCount (state, getters) {
    return getters.doneTodos.length
  }
}
```

```
store.getters.doneTodosCount // -> 1
```

We can now easily make use of it inside any component:

```
computed: {
  doneTodosCount () {
    return this.$store.getters.doneTodosCount
  }
}
```

Note that getters accessed as properties are cached as part of Vue's reactivity system.

#### Method-Style Access

You can also pass arguments to getters by returning a function. This is particularly useful when you want to query an array in the store:

```
getters: {
  // ...
  getTodoById: (state) => (id) => {
    return state.todos.find(todo => todo.id === id)
  }
}
```

```
store.getters.getTodoById(2) // -> { id: 2, text: '...', done: false }
```

Note that getters accessed via methods will run each time you call them, and the result is not cached.

#### The `mapGetters` Helper

The `mapGetters` helper simply maps store getters to local computed properties:

```
import { mapGetters } from 'vuex'

export default {
  // ...
  computed: {
    // mix the getters into computed with object spread operator
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
      // ...
    ])
  }
}
```

If you want to map a getter to a different name, use an object:

```
...mapGetters({
  // map `this.doneCount` to `this.$store.getters.doneTodosCount`
  doneCount: 'doneTodosCount'
})
```



### 2.3 Mutations

The only way to actually change state in a Vuex store is by committing a mutation. Vuex mutations are very similar to events: each mutation has a string **type** and a **handler**. The handler function is where we perform actual state modifications, and it will receive the state as the first argument:

```
const store = createStore({
  state: {
    count: 1
  },
  mutations: {
    increment (state) {
      // mutate state
      state.count++
    }
  }
})
```

You cannot directly call a mutation handler. Think of it more like event registration: "When a mutation with type `increment` is triggered, call this handler." To invoke a mutation handler, you need to call `store.commit` with its type:

```
store.commit('increment')
```

#### Commit with Payload

You can pass an additional argument to `store.commit`, which is called the **payload** for the mutation:

```
// ...
mutations: {
  increment (state, n) {
    state.count += n
  }
}
```

```
store.commit('increment', 10)
```

In most cases, the payload should be an object so that it can contain multiple fields, and the recorded mutation will also be more descriptive:

```
// ...
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```

```
store.commit('increment', {
  amount: 10
})
```

#### Object-Style Commit

An alternative way to commit a mutation is by directly using an object that has a `type` property:

```
store.commit({
  type: 'increment',
  amount: 10
})
```

When using object-style commit, the entire object will be passed as the payload to mutation handlers, so the handler remains the same:

```
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
```

####  Using Constants for Mutation Types

It is a commonly seen pattern to use constants for mutation types in various Flux implementations. This allows the code to take advantage of tooling like linters, and putting all constants in a single file allows your collaborators to get an at-a-glance view of what mutations are possible in the entire application:

```
// mutation-types.js
export const SOME_MUTATION = 'SOME_MUTATION'
```

```
// store.js
import { createStore } from 'vuex'
import { SOME_MUTATION } from './mutation-types'

const store = createStore({
  state: { ... },
  mutations: {
    // we can use the ES2015 computed property name feature
    // to use a constant as the function name
    [SOME_MUTATION] (state) {
      // mutate state
    }
  }
})
```

Whether to use constants is largely a preference - it can be helpful in large projects with many developers, but it's totally optional if you don't like them.

#### Mutations Must Be Synchronous

One important rule to remember is that **mutation handler functions must be synchronous**. Why? Consider the following example:

```
mutations: {
  someMutation (state) {
    api.callAsyncMethod(() => {
      state.count++
    })
  }
}
```

Now imagine we are debugging the app and looking at the devtool's mutation logs. For every mutation logged, the devtool will need to capture a "before" and "after" snapshots of the state. However, the asynchronous callback inside the example mutation above makes that impossible: the callback is not called yet when the mutation is committed, and there's no way for the devtool to know when the callback will actually be called - any state mutation performed in the callback is essentially un-trackable!

#### Committing Mutations in Components

You can commit mutations in components with `this.$store.commit('xxx')`, or use the `mapMutations` helper which maps component methods to `store.commit` calls (requires root `store` injection):

```
import { mapMutations } from 'vuex'

export default {
  // ...
  methods: {
    ...mapMutations([
      'increment', // map `this.increment()` to `this.$store.commit('increment')`

      // `mapMutations` also supports payloads:
      'incrementBy' // map `this.incrementBy(amount)` to `this.$store.commit('incrementBy', amount)`
    ]),
    ...mapMutations({
      add: 'increment' // map `this.add()` to `this.$store.commit('increment')`
    })
  }
}
```

####  On to Actions

Asynchronicity combined with state mutation can make your program very hard to reason about. For example, when you call two methods both with async callbacks that mutate the state, how do you know when they are called and which callback was called first? This is exactly why we want to separate the two concepts. In Vuex, **mutations are synchronous transactions**:

```
store.commit('increment')
// any state change that the "increment" mutation may cause
// should be done at this moment.
```

To handle asynchronous operations, let's introduce [Actions](https://next.vuex.vuejs.org/guide/actions.html).



### 2.4 Actions

Actions are similar to mutations, the differences being that:

- Instead of mutating the state, actions commit mutations.
- Actions can contain arbitrary asynchronous operations.

Let's register a simple action:

```
const store = createStore({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    }
  }
})
```

Action handlers receive a context object which exposes the same set of methods/properties on the store instance, so you can call `context.commit` to commit a mutation, or access the state and getters via `context.state` and `context.getters`. We can even call other actions with `context.dispatch`. We will see why this context object is not the store instance itself when we introduce [Modules](https://next.vuex.vuejs.org/guide/modules.html) later.

In practice, we often use ES2015 [argument destructuring](https://github.com/lukehoban/es6features#destructuring) to simplify the code a bit (especially when we need to call `commit` multiple times):

```
actions: {
  increment ({ commit }) {
    commit('increment')
  }
}
```

#### Dispatching Actions

Actions are triggered with the `store.dispatch` method:

```
store.dispatch('increment')
```

This may look silly at first sight: if we want to increment the count, why don't we just call `store.commit('increment')` directly? Remember that **mutations have to be synchronous**. Actions don't. We can perform **asynchronous** operations inside an action:

```
actions: {
  incrementAsync ({ commit }) {
    setTimeout(() => {
      commit('increment')
    }, 1000)
  }
}
```

Actions support the same payload format and object-style dispatch:

```
// dispatch with a payload
store.dispatch('incrementAsync', {
  amount: 10
})

// dispatch with an object
store.dispatch({
  type: 'incrementAsync',
  amount: 10
})
```

A more practical example of real-world actions would be an action to checkout a shopping cart, which involves **calling an async API** and **committing multiple mutations**:

```
actions: {
  checkout ({ commit, state }, products) {
    // save the items currently in the cart
    const savedCartItems = [...state.cart.added]
    // send out checkout request, and optimistically
    // clear the cart
    commit(types.CHECKOUT_REQUEST)
    // the shop API accepts a success callback and a failure callback
    shop.buyProducts(
      products,
      // handle success
      () => commit(types.CHECKOUT_SUCCESS),
      // handle failure
      () => commit(types.CHECKOUT_FAILURE, savedCartItems)
    )
  }
}
```

Note we are performing a flow of asynchronous operations, and recording the side effects (state mutations) of the action by committing them.

#### Dispatching Actions in Components

You can dispatch actions in components with `this.$store.dispatch('xxx')`, or use the `mapActions` helper which maps component methods to `store.dispatch` calls (requires root `store` injection):

```
import { mapActions } from 'vuex'

export default {
  // ...
  methods: {
    ...mapActions([
      'increment', // map `this.increment()` to `this.$store.dispatch('increment')`

      // `mapActions` also supports payloads:
      'incrementBy' // map `this.incrementBy(amount)` to `this.$store.dispatch('incrementBy', amount)`
    ]),
    ...mapActions({
      add: 'increment' // map `this.add()` to `this.$store.dispatch('increment')`
    })
  }
}
```

#### Composing Actions

Actions are often asynchronous, so how do we know when an action is done? And more importantly, how can we compose multiple actions together to handle more complex async flows?

The first thing to know is that `store.dispatch` can handle Promise returned by the triggered action handler and it also returns Promise:

```
actions: {
  actionA ({ commit }) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        commit('someMutation')
        resolve()
      }, 1000)
    })
  }
}
```

Now you can do:

```
store.dispatch('actionA').then(() => {
  // ...
})
```

And also in another action:

```
actions: {
  // ...
  actionB ({ dispatch, commit }) {
    return dispatch('actionA').then(() => {
      commit('someOtherMutation')
    })
  }
}
```

Finally, if we make use of [async / await](https://tc39.github.io/ecmascript-asyncawait/), we can compose our actions like this:

```
// assuming `getData()` and `getOtherData()` return Promises

actions: {
  async actionA ({ commit }) {
    commit('gotData', await getData())
  },
  async actionB ({ dispatch, commit }) {
    await dispatch('actionA') // wait for `actionA` to finish
    commit('gotOtherData', await getOtherData())
  }
}
```

*It's possible for a `store.dispatch` to trigger multiple action handlers in different modules. In such a case the returned value will be a Promise that resolves when all triggered handlers have been resolved.*

### 2.5 Modules

Due to using a single state tree, all states of our application are contained inside one big object. However, as our application grows in scale, the store can get really bloated.

To help with that, Vuex allows us to divide our store into **modules**. Each module can contain its own state, mutations, actions, getters, and even nested modules - it's fractal all the way down:

```
const moduleA = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... }
}

const store = createStore({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // -> `moduleA`'s state
store.state.b // -> `moduleB`'s state
```

#### 2.5.1 Module Local State

Inside a module's mutations and getters, the first argument received will be **the module's local state**.

```
const moduleA = {
  state: () => ({
    count: 0
  }),
  mutations: {
    increment (state) {
      // `state` is the local module state
      state.count++
    }
  },
  getters: {
    doubleCount (state) {
      return state.count * 2
    }
  }
}
```

Similarly, inside module actions, `context.state` will expose the local state, and root state will be exposed as `context.rootState`:

```
const moduleA = {
  // ...
  actions: {
    incrementIfOddOnRootSum ({ state, commit, rootState }) {
      if ((state.count + rootState.count) % 2 === 1) {
        commit('increment')
      }
    }
  }
}
```

Also, inside module getters, the root state will be exposed as their 3rd argument:

```
const moduleA = {
  // ...
  getters: {
    sumWithRootCount (state, getters, rootState) {
      return state.count + rootState.count
    }
  }
}
```

#### 2.5.2 Namespacing

By default, actions and mutations are still registered under the **global namespace** - this allows multiple modules to react to the same action/mutation type. Getters are also registered in the global namespace by default. However, this currently has no functional purpose (it's as is to avoid breaking changes). You must be careful not to define two getters with the same name in different, non-namespaced modules, resulting in an error.

If you want your modules to be more self-contained or reusable, you can mark it as namespaced with `namespaced: true`. When the module is registered, all of its getters, actions and mutations will be automatically namespaced based on the path the module is registered at. For example:

```
const store = createStore({
  modules: {
    account: {
      namespaced: true,

      // module assets
      state: () => ({ ... }), // module state is already nested and not affected by namespace option
      getters: {
        isAdmin () { ... } // -> getters['account/isAdmin']
      },
      actions: {
        login () { ... } // -> dispatch('account/login')
      },
      mutations: {
        login () { ... } // -> commit('account/login')
      },

      // nested modules
      modules: {
        // inherits the namespace from parent module
        myPage: {
          state: () => ({ ... }),
          getters: {
            profile () { ... } // -> getters['account/profile']
          }
        },

        // further nest the namespace
        posts: {
          namespaced: true,

          state: () => ({ ... }),
          getters: {
            popular () { ... } // -> getters['account/posts/popular']
          }
        }
      }
    }
  }
})
```

Namespaced getters and actions will receive localized `getters`, `dispatch` and `commit`. In other words, you can use the module assets without writing prefix in the same module. Toggling between namespaced or not does not affect the code inside the module.

##### Accessing Global Assets in Namespaced Modules

If you want to use global state and getters, `rootState` and `rootGetters` are passed as the 3rd and 4th arguments to getter functions, and also exposed as properties on the `context` object passed to action functions.

To dispatch actions or commit mutations in the global namespace, pass `{ root: true }` as the 3rd argument to `dispatch` and `commit`.

```
modules: {
  foo: {
    namespaced: true,

    getters: {
      // `getters` is localized to this module's getters
      // you can use rootGetters via 4th argument of getters
      someGetter (state, getters, rootState, rootGetters) {
        getters.someOtherGetter // -> 'foo/someOtherGetter'
        rootGetters.someOtherGetter // -> 'someOtherGetter'
        rootGetters['bar/someOtherGetter'] // -> 'bar/someOtherGetter'
      },
      someOtherGetter: state => { ... }
    },

    actions: {
      // dispatch and commit are also localized for this module
      // they will accept `root` option for the root dispatch/commit
      someAction ({ dispatch, commit, getters, rootGetters }) {
        getters.someGetter // -> 'foo/someGetter'
        rootGetters.someGetter // -> 'someGetter'
        rootGetters['bar/someGetter'] // -> 'bar/someGetter'

        dispatch('someOtherAction') // -> 'foo/someOtherAction'
        dispatch('someOtherAction', null, { root: true }) // -> 'someOtherAction'

        commit('someMutation') // -> 'foo/someMutation'
        commit('someMutation', null, { root: true }) // -> 'someMutation'
      },
      someOtherAction (ctx, payload) { ... }
    }
  }
}
```

##### Register Global Action in Namespaced Modules

If you want to register global actions in namespaced modules, you can mark it with `root: true` and place the action definition to function `handler`. For example:

```
{
  actions: {
    someOtherAction ({dispatch}) {
      dispatch('someAction')
    }
  },
  modules: {
    foo: {
      namespaced: true,

      actions: {
        someAction: {
          root: true,
          handler (namespacedContext, payload) { ... } // -> 'someAction'
        }
      }
    }
  }
}
```

##### Binding Helpers with Namespace

When binding a namespaced module to components with the `mapState`, `mapGetters`, `mapActions` and `mapMutations` helpers, it can get a bit verbose:

```
computed: {
  ...mapState({
    a: state => state.some.nested.module.a,
    b: state => state.some.nested.module.b
  }),
  ...mapGetters([
    'some/nested/module/someGetter', // -> this['some/nested/module/someGetter']
    'some/nested/module/someOtherGetter', // -> this['some/nested/module/someOtherGetter']
  ])
},
methods: {
  ...mapActions([
    'some/nested/module/foo', // -> this['some/nested/module/foo']()
    'some/nested/module/bar' // -> this['some/nested/module/bar']()
  ])
}
```

In such cases, you can pass the module namespace string as the first argument to the helpers so that all bindings are done using that module as the context. The above can be simplified to:

```
computed: {
  ...mapState('some/nested/module', {
    a: state => state.a,
    b: state => state.b
  }),
  ...mapGetters('some/nested/module', [
    'someGetter', // -> this.someGetter
    'someOtherGetter', // -> this.someOtherGetter
  ])
},
methods: {
  ...mapActions('some/nested/module', [
    'foo', // -> this.foo()
    'bar' // -> this.bar()
  ])
}
```

Furthermore, you can create namespaced helpers by using `createNamespacedHelpers`. It returns an object having new component binding helpers that are bound with the given namespace value:

```
import { createNamespacedHelpers } from 'vuex'

const { mapState, mapActions } = createNamespacedHelpers('some/nested/module')

export default {
  computed: {
    // look up in `some/nested/module`
    ...mapState({
      a: state => state.a,
      b: state => state.b
    })
  },
  methods: {
    // look up in `some/nested/module`
    ...mapActions([
      'foo',
      'bar'
    ])
  }
}
```

##### Caveat for Plugin Developers

You may care about unpredictable namespacing for your modules when you create a [plugin](https://next.vuex.vuejs.org/guide/plugins.html) that provides the modules and let users add them to a Vuex store. Your modules will be also namespaced if the plugin users add your modules under a namespaced module. To adapt this situation, you may need to receive a namespace value via your plugin option:

```
// get namespace value via plugin option
// and returns Vuex plugin function
export function createPlugin (options = {}) {
  return function (store) {
    // add namespace to plugin module's types
    const namespace = options.namespace || ''
    store.dispatch(namespace + 'pluginAction')
  }
}
```

#### 2.5.3 Dynamic Module Registration

You can register a module **after** the store has been created with the `store.registerModule` method:

```
import { createStore } from 'vuex'

const store = createStore({ /* options */ })

// register a module `myModule`
store.registerModule('myModule', {
  // ...
})

// register a nested module `nested/myModule`
store.registerModule(['nested', 'myModule'], {
  // ...
})
```

The module's state will be exposed as `store.state.myModule` and `store.state.nested.myModule`.

Dynamic module registration makes it possible for other Vue plugins to also leverage Vuex for state management by attaching a module to the application's store. For example, the [`vuex-router-sync`](https://github.com/vuejs/vuex-router-sync) library integrates vue-router with vuex by managing the application's route state in a dynamically attached module.

You can also remove a dynamically registered module with `store.unregisterModule(moduleName)`. Note you cannot remove static modules (declared at store creation) with this method.

Note that you may check if the module is already registered to the store or not via `store.hasModule(moduleName)` method. One thing to keep in mind is that nested modules should be passed as arrays for both the `registerModule` and `hasModule` and not as a string with the path to the module.

##### Preserving state

It may be likely that you want to preserve the previous state when registering a new module, such as preserving state from a Server Side Rendered app. You can achieve this with `preserveState` option: `store.registerModule('a', module, { preserveState: true })`

When you set `preserveState: true`, the module is registered, actions, mutations and getters are added to the store, but the state is not. It's assumed that your store state already contains state for that module and you don't want to overwrite it.

#### 2.5.4 Module Reuse

Sometimes we may need to create multiple instances of a module, for example:

- Creating multiple stores that use the same module (e.g. To [avoid stateful singletons in the SSR](https://ssr.vuejs.org/en/structure.html#avoid-stateful-singletons) when the `runInNewContext` option is `false` or `'once'`);
- Register the same module multiple times in the same store.

If we use a plain object to declare the state of the module, then that state object will be shared by reference and cause cross store/module state pollution when it's mutated.

This is actually the exact same problem with `data` inside Vue components. So the solution is also the same - use a function for declaring module state (supported in 2.3.0+):

```
const MyReusableModule = {
  state: () => ({
    foo: 'bar'
  }),
  // mutations, actions, getters...
}
```



## 3. **Advanced**

### 3.1 Application Structure

Vuex doesn't really restrict how you structure your code. Rather, it enforces a set of high-level principles:

1. Application-level state is centralized in the store.
2. The only way to mutate the state is by committing **mutations**, which are synchronous transactions.
3. Asynchronous logic should be encapsulated in, and can be composed with **actions**.

As long as you follow these rules, it's up to you how to structure your project. If your store file gets too big, simply start splitting the actions, mutations and getters into separate files.

For any non-trivial app, we will likely need to leverage modules. Here's an example project structure:

```
├── index.html
├── main.js
├── api
│   └── ... # abstractions for making API requests
├── components
│   ├── App.vue
│   └── ...
└── store
    ├── index.js          # where we assemble modules and export the store
    ├── actions.js        # root actions
    ├── mutations.js      # root mutations
    └── modules
        ├── cart.js       # cart module
        └── products.js   # products module
```

As a reference, check out the [Shopping Cart Example](https://github.com/vuejs/vuex/tree/4.0/examples/classic/shopping-cart).

### 3.2 Composition API

To access the store within the `setup` hook, you can call the `useStore` function. This is the equivalent of retrieving `this.$store` within a component using the Option API.

```
import { useStore } from 'vuex'

export default {
  setup () {
    const store = useStore()
  }
}
```

#### Accessing State and Getters

In order to access state and getters, you will want to create `computed` references to retain reactivity. This is the equivalent of creating computed properties using the Option API.

```
import { computed } from 'vue'
import { useStore } from 'vuex'

export default {
  setup () {
    const store = useStore()

    return {
      // access a state in computed function
      count: computed(() => store.state.count),

      // access a getter in computed function
      double: computed(() => store.getters.double)
    }
  }
}
```

#### Accessing Mutations and Actions

When accessing mutations and actions, you can simply provide the `commit` and `dispatch` function inside the `setup` hook.

```
import { useStore } from 'vuex'

export default {
  setup () {
    const store = useStore()

    return {
      // access a mutation
      increment: () => store.commit('increment'),

      // access an action
      asyncIncrement: () => store.dispatch('asyncIncrement')
    }
  }
}
```

#### Examples

Check out the [Composition API example](https://github.com/vuejs/vuex/tree/4.0/examples/composition) to see example applications utilizing Vuex and Vue's Composition API.



### 3.3 Plugins

Vuex stores accept the `plugins` option that exposes hooks for each mutation. A Vuex plugin is simply a function that receives the store as the only argument:

```
const myPlugin = (store) => {
  // called when the store is initialized
  store.subscribe((mutation, state) => {
    // called after every mutation.
    // The mutation comes in the format of `{ type, payload }`.
  })
}
```

And can be used like this:

```
const store = createStore({
  // ...
  plugins: [myPlugin]
})
```

#### Committing Mutations Inside Plugins

Plugins are not allowed to directly mutate state - similar to your components, they can only trigger changes by committing mutations.

By committing mutations, a plugin can be used to sync a data source to the store. For example, to sync a websocket data source to the store (this is just a contrived example, in reality the `createWebSocketPlugin` function can take some additional options for more complex tasks):

```
export default function createWebSocketPlugin (socket) {
  return (store) => {
    socket.on('data', data => {
      store.commit('receiveData', data)
    })
    store.subscribe(mutation => {
      if (mutation.type === 'UPDATE_DATA') {
        socket.emit('update', mutation.payload)
      }
    })
  }
}
```

```
const plugin = createWebSocketPlugin(socket)

const store = createStore({
  state,
  mutations,
  plugins: [plugin]
})
```

####  Taking State Snapshots

Sometimes a plugin may want to receive "snapshots" of the state, and also compare the post-mutation state with pre-mutation state. To achieve that, you will need to perform a deep-copy on the state object:

```
const myPluginWithSnapshot = (store) => {
  let prevState = _.cloneDeep(store.state)
  store.subscribe((mutation, state) => {
    let nextState = _.cloneDeep(state)

    // compare `prevState` and `nextState`...

    // save state for next mutation
    prevState = nextState
  })
}
```

**Plugins that take state snapshots should be used only during development.** When using webpack or Browserify, we can let our build tools handle that for us:

```
const store = createStore({
  // ...
  plugins: process.env.NODE_ENV !== 'production'
    ? [myPluginWithSnapshot]
    : []
})
```

The plugin will be used by default. For production, you will need [DefinePlugin](https://webpack.js.org/plugins/define-plugin/) for webpack or [envify](https://github.com/hughsk/envify) for Browserify to convert the value of `process.env.NODE_ENV !== 'production'` to `false` for the final build.

#### Built-in Logger Plugin

Vuex comes with a logger plugin for common debugging usage:

```
import { createLogger } from 'vuex'

const store = createStore({
  plugins: [createLogger()]
})
```

The `createLogger` function takes a few options:

```
const logger = createLogger({
  collapsed: false, // auto-expand logged mutations
  filter (mutation, stateBefore, stateAfter) {
    // returns `true` if a mutation should be logged
    // `mutation` is a `{ type, payload }`
    return mutation.type !== "aBlocklistedMutation"
  },
  actionFilter (action, state) {
    // same as `filter` but for actions
    // `action` is a `{ type, payload }`
    return action.type !== "aBlocklistedAction"
  },
  transformer (state) {
    // transform the state before logging it.
    // for example return only a specific sub-tree
    return state.subTree
  },
  mutationTransformer (mutation) {
    // mutations are logged in the format of `{ type, payload }`
    // we can format it any way we want.
    return mutation.type
  },
  actionTransformer (action) {
    // Same as mutationTransformer but for actions
    return action.type
  },
  logActions: true, // Log Actions
  logMutations: true, // Log mutations
  logger: console, // implementation of the `console` API, default `console`
})
```

The logger file can also be included directly via a `<script>` tag, and will expose the `createVuexLogger` function globally.

Note the logger plugin takes state snapshots, so use it only during development.

### 3.4 Strict Mode

To enable strict mode, simply pass in `strict: true` when creating a Vuex store:

```
const store = createStore({
  // ...
  strict: true
})
```

In strict mode, whenever Vuex state is mutated outside of mutation handlers, an error will be thrown. This ensures that all state mutations can be explicitly tracked by debugging tools.

#### Development vs. Production

**Do not enable strict mode when deploying for production!** Strict mode runs a synchronous deep watcher on the state tree for detecting inappropriate mutations, and it can be quite expensive when you make large amount of mutations to the state. Make sure to turn it off in production to avoid the performance cost.

Similar to plugins, we can let the build tools handle that:

```
const store = createStore({
  // ...
  strict: process.env.NODE_ENV !== 'production'
})
```

### 3.5 Form Handling

When using Vuex in strict mode, it could be a bit tricky to use `v-model` on a piece of state that belongs to Vuex:

```
<input v-model="obj.message">
```

Assuming `obj` is a computed property that returns an Object from the store, the `v-model` here will attempt to directly mutate `obj.message` when the user types in the input. In strict mode, this will result in an error because the mutation is not performed inside an explicit Vuex mutation handler.

The "Vuex way" to deal with it is binding the `<input>`'s value and call a method on the `input` or `change` event:

```
<input :value="message" @input="updateMessage">
```

```
// ...
computed: {
  ...mapState({
    message: state => state.obj.message
  })
},
methods: {
  updateMessage (e) {
    this.$store.commit('updateMessage', e.target.value)
  }
}
```

And here's the mutation handler:

```
// ...
mutations: {
  updateMessage (state, message) {
    state.obj.message = message
  }
}
```

#### Two-way Computed Property

Admittedly, the above is quite a bit more verbose than `v-model` + local state, and we lose some of the useful features from `v-model` as well. An alternative approach is using a two-way computed property with a setter:

```
<input v-model="message">
```

```
// ...
computed: {
  message: {
    get () {
      return this.$store.state.obj.message
    },
    set (value) {
      this.$store.commit('updateMessage', value)
    }
  }
}
```

### 3.6 Testing

The main parts we want to unit test in Vuex are mutations and actions.

#### 3.6.1 Testing Mutations

Mutations are very straightforward to test, because they are just functions that completely rely on their arguments. One trick is that if you are using ES2015 modules and put your mutations inside your `store.js` file, in addition to the default export, you should also export the mutations as a named export:

```
const state = { ... }

// export `mutations` as a named export
export const mutations = { ... }

export default createStore({
  state,
  mutations
})
```

Example testing a mutation using Mocha + Chai (you can use any framework/assertion libraries you like):

```
// mutations.js
export const mutations = {
  increment: state => state.count++
}
```

```
// mutations.spec.js
import { expect } from 'chai'
import { mutations } from './store'

// destructure assign `mutations`
const { increment } = mutations

describe('mutations', () => {
  it('INCREMENT', () => {
    // mock state
    const state = { count: 0 }
    // apply mutation
    increment(state)
    // assert result
    expect(state.count).to.equal(1)
  })
})
```

#### 3.6.2 Testing Actions

Actions can be a bit more tricky because they may call out to external APIs. When testing actions, we usually need to do some level of mocking - for example, we can abstract the API calls into a service and mock that service inside our tests. In order to easily mock dependencies, we can use webpack and [inject-loader](https://github.com/plasticine/inject-loader) to bundle our test files.

Example testing an async action:

```
// actions.js
import shop from '../api/shop'

export const getAllProducts = ({ commit }) => {
  commit('REQUEST_PRODUCTS')
  shop.getProducts(products => {
    commit('RECEIVE_PRODUCTS', products)
  })
}
```

```
// actions.spec.js

// use require syntax for inline loaders.
// with inject-loader, this returns a module factory
// that allows us to inject mocked dependencies.
import { expect } from 'chai'
const actionsInjector = require('inject-loader!./actions')

// create the module with our mocks
const actions = actionsInjector({
  '../api/shop': {
    getProducts (cb) {
      setTimeout(() => {
        cb([ /* mocked response */ ])
      }, 100)
    }
  }
})

// helper for testing action with expected mutations
const testAction = (action, payload, state, expectedMutations, done) => {
  let count = 0

  // mock commit
  const commit = (type, payload) => {
    const mutation = expectedMutations[count]

    try {
      expect(type).to.equal(mutation.type)
      expect(payload).to.deep.equal(mutation.payload)
    } catch (error) {
      done(error)
    }

    count++
    if (count >= expectedMutations.length) {
      done()
    }
  }

  // call the action with mocked store and arguments
  action({ commit, state }, payload)

  // check if no mutations should have been dispatched
  if (expectedMutations.length === 0) {
    expect(count).to.equal(0)
    done()
  }
}

describe('actions', () => {
  it('getAllProducts', done => {
    testAction(actions.getAllProducts, null, {}, [
      { type: 'REQUEST_PRODUCTS' },
      { type: 'RECEIVE_PRODUCTS', payload: { /* mocked response */ } }
    ], done)
  })
})
```

If you have spies available in your testing environment (for example via [Sinon.JS](http://sinonjs.org/)), you can use them instead of the `testAction` helper:

```
describe('actions', () => {
  it('getAllProducts', () => {
    const commit = sinon.spy()
    const state = {}

    actions.getAllProducts({ commit, state })

    expect(commit.args).to.deep.equal([
      ['REQUEST_PRODUCTS'],
      ['RECEIVE_PRODUCTS', { /* mocked response */ }]
    ])
  })
})
```

#### 3.6.3  Testing Getters

If your getters have complicated computation, it is worth testing them. Getters are also very straightforward to test for the same reason as mutations.

Example testing a getter:

```
// getters.js
export const getters = {
  filteredProducts (state, { filterCategory }) {
    return state.products.filter(product => {
      return product.category === filterCategory
    })
  }
}
```

```
// getters.spec.js
import { expect } from 'chai'
import { getters } from './getters'

describe('getters', () => {
  it('filteredProducts', () => {
    // mock state
    const state = {
      products: [
        { id: 1, title: 'Apple', category: 'fruit' },
        { id: 2, title: 'Orange', category: 'fruit' },
        { id: 3, title: 'Carrot', category: 'vegetable' }
      ]
    }
    // mock getter
    const filterCategory = 'fruit'

    // get the result from the getter
    const result = getters.filteredProducts(state, { filterCategory })

    // assert the result
    expect(result).to.deep.equal([
      { id: 1, title: 'Apple', category: 'fruit' },
      { id: 2, title: 'Orange', category: 'fruit' }
    ])
  })
})
```

#### 3.6.4 Running Tests

If your mutations and actions are written properly, the tests should have no direct dependency on Browser APIs after proper mocking. Thus you can simply bundle the tests with webpack and run it directly in Node. Alternatively, you can use `mocha-loader` or Karma + `karma-webpack` to run the tests in real browsers.

##### Running in Node

Create the following webpack config (together with proper [`.babelrc`](https://babeljs.io/docs/usage/babelrc/)):

```
// webpack.config.js
module.exports = {
  entry: './test.js',
  output: {
    path: __dirname,
    filename: 'test-bundle.js'
  },
  module: {
    loaders: [
      {
        test: /\.js$/,
        loader: 'babel-loader',
        exclude: /node_modules/
      }
    ]
  }
}
```

Then:

```
webpack
mocha test-bundle.js
```

##### Running in Browser

1. Install `mocha-loader`.
2. Change the `entry` from the webpack config above to `'mocha-loader!babel-loader!./test.js'`.
3. Start `webpack-dev-server` using the config.
4. Go to `localhost:8080/webpack-dev-server/test-bundle`.

##### Running in Browser with Karma + karma-webpack

Consult the setup in [vue-loader documentation](https://vue-loader.vuejs.org/en/workflow/testing.html).

### 3.7 Hot Reloading

Vuex supports hot-reloading mutations, modules, actions and getters during development, using webpack's [Hot Module Replacement API](https://webpack.js.org/guides/hot-module-replacement/). You can also use it in Browserify with the [browserify-hmr](https://github.com/AgentME/browserify-hmr/) plugin.

For mutations and modules, you need to use the `store.hotUpdate()` API method:

```
// store.js
import { createStore } from 'vuex'
import mutations from './mutations'
import moduleA from './modules/a'

const state = { ... }

const store = createStore({
  state,
  mutations,
  modules: {
    a: moduleA
  }
})

if (module.hot) {
  // accept actions and mutations as hot modules
  module.hot.accept(['./mutations', './modules/a'], () => {
    // require the updated modules
    // have to add .default here due to babel 6 module output
    const newMutations = require('./mutations').default
    const newModuleA = require('./modules/a').default
    // swap in the new modules and mutations
    store.hotUpdate({
      mutations: newMutations,
      modules: {
        a: newModuleA
      }
    })
  })
}
```

Checkout the [counter-hot example](https://github.com/vuejs/vuex/tree/dev/examples/counter-hot) to play with hot-reload.

#### Dynamic module hot reloading

If you use modules exclusively, you can use `require.context` to load and hot reload all modules dynamically.

```
// store.js
import { createStore } from 'vuex'

// Load all modules.
function loadModules() {
  const context = require.context("./modules", false, /([a-z_]+)\.js$/i)

  const modules = context
    .keys()
    .map((key) => ({ key, name: key.match(/([a-z_]+)\.js$/i)[1] }))
    .reduce(
      (modules, { key, name }) => ({
        ...modules,
        [name]: context(key).default
      }),
      {}
    )

  return { context, modules }
}

const { context, modules } = loadModules()

const store = createStore({
  modules
})

if (module.hot) {
  // Hot reload whenever any module changes.
  module.hot.accept(context.id, () => {
    const { modules } = loadModules()

    store.hotUpdate({
      modules
    })
  })
}
```

### 3.8 TypeScript Support

Vuex provides its typings so you can use TypeScript to write a store definition. You don't need any special TypeScript configuration for Vuex. Please follow [Vue's basic TypeScript setup](https://v3.vuejs.org/guide/typescript-support.html) to configure your project.

However, if you're writing your Vue components in TypeScript, there're a few steps to follow that require for you to correctly provide typings for a store.

#### 3.8.1 Typing `$store` Property in Vue Component

Vuex doesn't provide typings for `this.$store` property out of the box. When used with TypeScript, you must declare your own module augmentation.

To do so, declare custom typings for Vue's `ComponentCustomProperties` by adding a declaration file in your project folder:

```
// vuex.d.ts
import { ComponentCustomProperties } from 'vue'
import { Store } from 'vuex'

declare module '@vue/runtime-core' {
  // declare your own store states
  interface State {
    count: number
  }

  // provide typings for `this.$store`
  interface ComponentCustomProperties {
    $store: Store<State>
  }
}
```

#### 3.8.2 Typing `useStore` Composition Function

When you're writing your Vue component in Composition API, you will most likely want `useStore` to return the typed store. For `useStore` to correctly return the typed store, you must:

1. Define the typed `InjectionKey`.
2. Provide the typed `InjectionKey` when installing a store to the Vue app.
3. Pass the typed `InjectionKey` to the `useStore` method.

Let's tackle this step by step. First, define the key using Vue's `InjectionKey` interface along with your own store typing definition:

```
// store.ts
import { InjectionKey } from 'vue'
import { createStore, Store } from 'vuex'

// define your typings for the store state
export interface State {
  count: number
}

// define injection key
export const key: InjectionKey<Store<State>> = Symbol()

export const store = createStore<State>({
  state: {
    count: 0
  }
})
```

Next, pass the defined injection key when installing the store to the Vue app:

```
// main.ts
import { createApp } from 'vue'
import { store, key } from './store'

const app = createApp({ ... })

// pass the injection key
app.use(store, key)

app.mount('#app')
```

Finally, you can pass the key to the `useStore` method to retrieve the typed store.

```
// in a vue component
import { useStore } from 'vuex'
import { key } from './store'

export default {
  setup () {
    const store = useStore(key)

    store.state.count // typed as number
  }
}
```

Under the hood, Vuex installs the store to the Vue app using Vue's [Provide/Inject](https://v3.vuejs.org/api/composition-api.html#provide-inject) feature which is why the injection key is an important factor.

#####  Simplifying `useStore` usage

Having to import `InjectionKey` and passing it to `useStore` everywhere it's used can quickly become a repetitive task. To simplify matters, you can define your own composable function to retrieve a typed store:

```
// store.ts
import { InjectionKey } from 'vue'
import { createStore, useStore as baseUseStore, Store } from 'vuex'

export interface State {
  count: number
}

export const key: InjectionKey<Store<State>> = Symbol()

export const store = createStore<State>({
  state: {
    count: 0
  }
})

// define your own `useStore` composition function
export function useStore () {
  return baseUseStore(key)
}
```

Now, by importing your own composable function, you can retrieve the typed store **without** having to provide the injection key and its typing:

```
// in a vue component
import { useStore } from './store'

export default {
  setup () {
    const store = useStore()

    store.state.count // typed as number
  }
}
```



## 4. **Migration Guide**

### Migrating to 4.0 from 3.x

Almost all Vuex 4 APIs have remained unchanged from Vuex 3. However, there are still a few breaking changes that you must fix.

- Breaking Changes
  - [Installation process](https://next.vuex.vuejs.org/guide/migrating-to-4-0-from-3-x.html#installation-process)
  - [TypeScript support](https://next.vuex.vuejs.org/guide/migrating-to-4-0-from-3-x.html#typescript-support)
  - [Bundles are now aligned with Vue 3](https://next.vuex.vuejs.org/guide/migrating-to-4-0-from-3-x.html#bundles-are-now-aligned-with-vue-3)
  - ["createLogger" function is exported from the core module](https://next.vuex.vuejs.org/guide/migrating-to-4-0-from-3-x.html#createlogger-function-is-exported-from-the-core-module)
- New Features
  - [New "useStore" composition function](https://next.vuex.vuejs.org/guide/migrating-to-4-0-from-3-x.html#new-usestore-composition-function)

#### 4.1 Breaking Changes

#####  Installation process

To align with the new Vue 3 initialization process, the installation process of Vuex has changed. To create a new store, users are now encouraged to use the newly introduced createStore function.

```
import { createStore } from 'vuex'

export const store = createStore({
  state () {
    return {
      count: 1
    }
  }
})
```

To install Vuex to a Vue instance, pass the `store` instead of Vuex.

```
import { createApp } from 'vue'
import { store } from './store'
import App from './App.vue'

const app = createApp(App)

app.use(store)

app.mount('#app')
```

**NOTE**

*Whilst this is not technically a breaking change, you may still use the `new Store(...)` syntax, we recommend this approach to align with Vue 3 and Vue Router Next.*

##### TypeScript support

Vuex 4 removes its global typings for `this.$store` within a Vue component to solve [issue #994](https://github.com/vuejs/vuex/issues/994). When used with TypeScript, you must declare your own module augmentation.

Place the following code in your project to allow `this.$store` to be typed correctly:

```
// vuex-shim.d.ts

import { ComponentCustomProperties } from 'vue'
import { Store } from 'vuex'

declare module '@vue/runtime-core' {
  // Declare your own store states.
  interface State {
    count: number
  }

  interface ComponentCustomProperties {
    $store: Store<State>
  }
}
```

You can learn more in the [TypeScript Support](https://next.vuex.vuejs.org/guide/typescript-support.html) section.

##### Bundles are now aligned with Vue 3

The following bundles are generated to align with Vue 3 bundles:

- ```
  vuex.global(.prod).js
  ```

  - For direct use with `<script src="...">` in the browser. Exposes the Vuex global.
  - Global build is built as IIFE, and not UMD, and is only meant for direct use with `<script src="...">`.
  - Contains hard-coded prod/dev branches and the prod build is pre-minified. Use the `.prod.js` files for production.

- ```
  vuex.esm-browser(.prod).js
  ```

  - For use with native ES module imports (including module supporting browsers via `<script type="module">`.

- ```
  vuex.esm-bundler.js
  ```

  - For use with bundlers such as `webpack`, `rollup` and `parcel`.
  - Leaves prod/dev branches with `process.env.NODE_ENV` guards (must be replaced by bundler).
  - Does not ship minified builds (to be done together with the rest of the code after bundling).

- ```
  vuex.cjs.js
  ```

  - For use in Node.js server-side rendering with `require()`.

##### `createLogger` function is exported from the core module

In Vuex 3, `createLogger` function was exported from `vuex/dist/logger` but it's now included in the core package. The function should be imported directly from the `vuex` package.

```
import { createLogger } from 'vuex'
```

#### 4.2 New Features

##### New `useStore` composition function

Vuex 4 introduces a new API to interact with the store in Composition API. You can use the `useStore` composition function to retrieve the store within the component `setup` hook.

```
import { useStore } from 'vuex'

export default {
  setup () {
    const store = useStore()
  }
}
```

You can learn more in the [Composition API](https://next.vuex.vuejs.org/guide/composition-api.html) section.





## 5. Vuex Api

https://next.vuex.vuejs.org/api/



- Store
  - [createStore](https://next.vuex.vuejs.org/api/#createstore)
- Store Constructor Options
  - [state](https://next.vuex.vuejs.org/api/#state)
  - [mutations](https://next.vuex.vuejs.org/api/#mutations)
  - [actions](https://next.vuex.vuejs.org/api/#actions)
  - [getters](https://next.vuex.vuejs.org/api/#getters)
  - [modules](https://next.vuex.vuejs.org/api/#modules)
  - [plugins](https://next.vuex.vuejs.org/api/#plugins)
  - [strict](https://next.vuex.vuejs.org/api/#strict)
  - [devtools](https://next.vuex.vuejs.org/api/#devtools)
- Store Instance Properties
  - [state](https://next.vuex.vuejs.org/api/#state-2)
  - [getters](https://next.vuex.vuejs.org/api/#getters-2)
- Store Instance Methods
  - [commit](https://next.vuex.vuejs.org/api/#commit)
  - [dispatch](https://next.vuex.vuejs.org/api/#dispatch)
  - [replaceState](https://next.vuex.vuejs.org/api/#replacestate)
  - [watch](https://next.vuex.vuejs.org/api/#watch)
  - [subscribe](https://next.vuex.vuejs.org/api/#subscribe)
  - [subscribeAction](https://next.vuex.vuejs.org/api/#subscribeaction)
  - [registerModule](https://next.vuex.vuejs.org/api/#registermodule)
  - [unregisterModule](https://next.vuex.vuejs.org/api/#unregistermodule)
  - [hasModule](https://next.vuex.vuejs.org/api/#hasmodule)
  - [hotUpdate](https://next.vuex.vuejs.org/api/#hotupdate)
- Component Binding Helpers
  - [mapState](https://next.vuex.vuejs.org/api/#mapstate)
  - [mapGetters](https://next.vuex.vuejs.org/api/#mapgetters)
  - [mapActions](https://next.vuex.vuejs.org/api/#mapactions)
  - [mapMutations](https://next.vuex.vuejs.org/api/#mapmutations)
  - [createNamespacedHelpers](https://next.vuex.vuejs.org/api/#createnamespacedhelpers)
- Composable Functions
  - [useStore](https://next.vuex.vuejs.org/api/#usestore)



































































































































































































































































# Vuex     结束



