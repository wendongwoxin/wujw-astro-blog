---
title: "From Vue2 to Vue3"
description: "详细对比Vue2和Vue3的主要区别，以及一些新特性、生态系统的更新"
pubDate: "2024-06-19"
heroImage: "/images/from-vue2-to-vue3.png"
---

## 核心架构变化

### 1. 响应式系统

- **Vue2**: 基于 Object.defineProperty

  - 只能监听对象已存在的属性
  - 需要使用 `Vue.set()` 动态添加响应式属性
  - 数组变更检测存在限制

- **Vue3**: 采用 Proxy
  - 可以监听动态添加的属性
  - 可以监听数组索引和长度变化
  - 性能更好，内存占用更低

### 2. 组合式 API vs 选项式 API

- **Vue2**: 主要使用选项式 API

  ```js
  export default {
    data() {
      return { count: 0 };
    },
    methods: {
      increment() {
        this.count++;
      },
    },
  };
  ```

- **Vue3**: 引入组合式 API

  ```js
  import { ref } from "vue";

  export default {
    setup() {
      const count = ref(0);
      const increment = () => count.value++;

      return { count, increment };
    },
  };
  ```

## 性能优化

### 1. 虚拟 DOM 重写

- **Vue3**:
  - 编译时优化
  - 静态树提升
  - 静态属性提升
  - 基于 Proxy 的响应式系统

### 2. 包体积优化

- **Vue3**:
  - 更好的 tree-shaking 支持
  - 核心功能模块化
  - 按需引入

## 新特性介绍

### 1. Teleport 组件    

```vue
<teleport to="body">
    <div class="modal">
    <!-- 模态框内容 -->
    </div>
</teleport>
```

### 2. Fragments 支持
- **Vue3** 允许组件有多个根节点

```vue
<template>
    <header>...</header>
    <main>...</main>
    <footer>...</footer>
</template>
```

### 3. 更好的 TypeScript 支持
- Vue3 使用 TypeScript 重写
- 提供更好的类型推导
- 组合式 API 提供更好的类型支持

## 生态系统更新

### 1. 开发工具
- Vite 替代 Vue CLI
- Vue Devtools 升级

### 2. 状态管理
- Pinia 替代 Vuex
- 更好的 TypeScript 支持
- 更简单的 API


## 🎯 常见面试题

### 1. Composition API 和 Options API 的区别是什么？

**答案：**
主要区别在于：
- **代码组织方式**：
  - Options API 按选项类型组织代码（data、methods、computed等）
  - Composition API 按功能逻辑组织代码，相关代码可以放在一起
- **逻辑复用**：
  - Options API 主要通过 mixins，但容易造成命名冲突
  - Composition API 可以轻松地将逻辑提取到组合函数中复用
- **TypeScript 支持**：
  - Composition API 对 TypeScript 的类型推导更友好
- **性能**：
  - Composition API 的 tree-shaking 更好，没用到的代码不会打包

### 2. ref 和 reactive 的区别？什么时候用 ref，什么时候用 reactive？

**答案：**
- **ref**：
  - 用于基本类型数据
  - 需要通过 .value 访问
  - 可以直接重新赋值
```js
const count = ref(0)
count.value = 1
```

- **reactive**：
  - 用于对象类型数据
  - 直接访问属性
  - 不能直接重新赋值
```js
const state = reactive({ count: 0 })
state.count = 1
```

**使用建议：**
- 基本类型用 ref
- 对象类型用 reactive
- 如果需要解构或重新赋值，统一用 ref

### 3. 为什么 ref 需要使用 .value？

**答案：**
- JavaScript 的基本类型是按值传递的，需要一个包装对象来实现响应式
- .value 提供了一个响应式的引用，确保值的改变可以被追踪
- 在模板中使用时会自动解包，不需要 .value