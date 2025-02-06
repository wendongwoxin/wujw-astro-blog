---
title: "Pinia"
description: "详细介绍pinia的相关特性以及和Vuex的区别"
pubDate: "2024-06-25"
heroImage: "/images/pinia.webp"
---

# Pinia vs Vuex

Pinia 和 Vuex 都是 Vue.js 的状态管理库，但 Pinia 在设计上更为现代化，并且有着更简洁的 API。Pinia 是 Vue 3 的官方状态管理库，作为 Vuex 的继任者，Pinia 提供了一些 Vuex 所没有的特性，同时也简化了开发者的工作。

### 创建
Vuex 在 Vue中的使用方式是通过 **createStore** 创建一个 store：
```javascript
// store.js
import { createStore } from 'vuex'

export default createStore({
  state: {
    count: 0
  },
  mutations: {
    increment(state) {
      state.count++
    },
    decrement(state) {
      state.count--
    }
  },
  actions: {
    incrementAsync({ commit }) {
      setTimeout(() => {
        commit('increment')
      }, 1000)
    }
  }
})

```

Pinia 使用 **defineStore** 来定义 store：

```javascript
// stores/counter.js
import { defineStore } from 'pinia'

export const useCounterStore = defineStore('counter', {
  state: () => ({
    count: 0
  }),
  actions: {
    increment() {
      this.count++
    },
    decrement() {
      this.count--
    }
  }
})

```

### API设计比较

#### Vuex
1. Vuex 使用 state, mutations, actions, 和 getters 来管理状态。、
2. 使用 mutations 来修改 state，actions 可以处理异步操作，getters 用于派发计算属性。

#### Pinia

1. Pinia 的 API 设计简洁，只有 state, actions, 和 getters。
2. 没有 mutations，修改 state 直接通过 actions 来进行（与 Vue 3 的响应式设计更为契合）。
3. 支持 TypeScript 更好的类型推导。

### 响应式和性能

#### Vuex
1. Vuex 会通过 Vue 的响应式系统自动处理状态变化，但它需要使用 mutations 来修改状态，增加了代码复杂度。
2. 对于大型应用，Vuex 的 store 可能会变得比较臃肿，且对于某些操作可能会影响性能。

#### Pinia
1. Pinia 使用了 Vue 3 的 Composition API 和响应式系统，性能更好，API 更简洁。
2. Pinia 的 store 本质上是一个普通对象，采用 Vue 3 的 reactive 来实现响应式，而无需额外的 mutations。

### TypeScript 支持

#### Vuex
Vuex 支持 TypeScript，但 TypeScript 的类型推导并不完美，需要手动定义类型并进行配置。

#### Pinia
Pinia 完全支持 TypeScript，且支持自动推导类型。你只需在创建 store 时，TypeScript 会自动推导出 state、actions 和 getters 的类型。


Pinia 作为 Vuex 的替代品，带来了更现代化、简洁和高效的 API。如果你是 Vue 3 项目的开发者，推荐使用 Pinia 进行状态管理。