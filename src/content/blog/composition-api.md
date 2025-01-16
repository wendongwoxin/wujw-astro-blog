---
title: "Compositon Api"
description: "深入理解Vue 3 Composition API的核心概念、使用方法及最佳实践"
pubDate: "2024-06-20"
heroImage: "/images/vue-3-essentials.png"
---

# Composition API

## 什么是 Composition API？

Composition API 是 Vue 3 中引入的一种新的组件逻辑组织方式，它允许我们使用函数式的方式来组织组件的逻辑代码。相比于 Vue 2 的 Options API，它提供了更好的代码复用性和更灵活的逻辑组织方式。

## 核心概念

### 1. setup 函数

`setup` 是 Composition API 的入口点，它在组件创建之前执行：

```vue
<script setup>
    import { ref, onMounted } from 'vue'
    const count = ref(0)
    const increment = () => count.value++
    onMounted(() => {
        console.log('组件已挂载')
    })
</script>
```

### 2. 响应式系统

#### ref 和 reactive

```vue
<script setup>
    import { ref, reactive } from 'vue'
    // ref 用于基本类型
    const count = ref(0)
    // reactive 用于对象
    const state = reactive({
        name: '张三',
        age: 25
    })
</script>
```

### 3. 生命周期钩子
Vue 3 的 Composition API 提供了一套完整的生命周期钩子：
#### 常用的生命周期钩子
```vue
<script setup>
import { 
  onMounted, 
  onUpdated, 
  onUnmounted, 
  onBeforeMount 
} from 'vue'

// 组件挂载前
onBeforeMount(() => {
  console.log('组件挂载前')
  // 常用于：初始化一些数据，但不需要访问DOM
})

// 组件挂载后
onMounted(() => {
  console.log('组件挂载后')
  // 常用于：
  // 1. 发起API请求
  // 2. DOM操作
  // 3. 添加事件监听
  // 4. 第三方库初始化
})

// 组件更新后
onUpdated(() => {
  console.log('组件更新后')
  // 常用于：需要在DOM更新后进行操作
})

// 组件卸载前
onUnmounted(() => {
  console.log('组件卸载前')
  // 常用于：
  // 1. 清理定时器
  // 2. 取消事件监听
  // 3. 取消订阅
})
</script>
```

#### 其他生命周期钩子

```vue
<script setup>
import { 
  onBeforeUpdate,
  onActivated,
  onDeactivated,
  onErrorCaptured,
  onRenderTracked,
  onRenderTriggered,
  onServerPrefetch
} from 'vue'

// 组件更新前
onBeforeUpdate(() => {
  console.log('组件更新前')
  // 用于：在DOM更新之前访问状态
})

// keep-alive 组件激活时
onActivated(() => {
  console.log('组件被激活')
  // 用于：keep-alive 组件被激活时的逻辑
})

// keep-alive 组件停用时
onDeactivated(() => {
  console.log('组件被停用')
  // 用于：keep-alive 组件被停用时的逻辑
})

// 捕获后代组件错误
onErrorCaptured((err, instance, info) => {
  console.log('捕获到后代组件错误')
  // 用于：错误处理和日志记录
})

// 调试用：跟踪虚拟 DOM 重新渲染时收集的依赖
onRenderTracked((e) => {
  console.log('渲染依赖被跟踪')
  // 用于：调试，了解哪些依赖触发了组件重新渲染
})

// 调试用：跟踪虚拟 DOM 重新渲染的触发者
onRenderTriggered((e) => {
  console.log('渲染被触发')
  // 用于：调试，了解是什么触发了组件重新渲染
})

// SSR 专用：在服务器端渲染时预取数据
onServerPrefetch(async () => {
  // 用于：服务端渲染时等待异步数据
})
</script>
```

#### setup 特殊生命周期

`setup` 是一个特殊的生命周期钩子，它在组件创建之前执行，甚至在 `beforeCreate` 之前执行。这使得它成为使用 Composition API 的理想入口点。

```vue
<script>
import { ref, onMounted } from 'vue'

export default {
  // setup 函数的两个参数
  setup(props, context) {
    // props: 响应式的组件参数
    // context: 上下文对象，包含：
    //   - attrs: 非响应式的属性对象
    //   - slots: 插槽
    //   - emit: 触发事件的方法
    //   - expose: 暴露公共属性

    console.log('setup执行')
    const count = ref(0)

    // setup 会在所有生命周期钩子之前执行
    onMounted(() => {
      console.log('onMounted执行')
    })

    // 必须返回一个对象，对象中的属性可以在模板中使用
    return {
      count
    }
  }
}
</script>
```

#### `<script setup>` 语法糖

在 Vue 3.2+ 中，推荐使用 `<script setup>` 语法糖，它是更简洁的书写方式：

```vue
<script setup>
import { ref, onMounted } from 'vue'

// 无需返回值，顶层的变量和函数都会自动暴露给模板
const count = ref(0)

onMounted(() => {
  console.log('组件已挂载')
})
</script>
```

#### setup 的特点

1. **执行时机**
   - 在组件创建之前执行
   - 在 props 解析之后执行
   - 早于所有生命周期钩子

2. **访问限制**
   - 无法访问 `this`
   - 无法访问组件实例
   - 可以访问 props 和 context

3. **使用场景**
   - 初始化响应式数据
   - 定义方法和计算属性
   - 创建侦听器
   - 注册生命周期钩子
   
#### 完整的生命周期执行顺序

1. setup
2. beforeCreate (在 setup 中不需要)
3. created (在 setup 中不需要)
4. onBeforeMount
5. onMounted
6. onBeforeUpdate
7. onUpdated
8. onBeforeUnmount
9. onUnmounted

#### 使用建议

1. **最常用的钩子**：
   - `onMounted`: 初始化、API调用、DOM操作
   - `onUnmounted`: 清理工作
   - `onUpdated`: DOM更新后的操作

2. **调试场景**：
   - `onRenderTracked` 和 `onRenderTriggered` 主要用于开发调试

3. **keep-alive组件**：
   - 使用 `onActivated` 和 `onDeactivated` 来处理缓存组件的生命周期

4. **错误处理**：
   - `onErrorCaptured` 用于统一的错误处理和日志记录

## 为什么使用 Composition API？

1. **更好的代码组织**
   - 相关的逻辑可以组织在一起
   - 不再受限于 Options API 的选项分离

2. **更好的逻辑复用**
   - 可以轻松地将逻辑提取到独立的函数中
   - 通过组合函数实现逻辑复用

3. **更好的类型推导**
   - 对 TypeScript 的支持更好
   - 减少了类型标注的需求

## 最佳实践

1. **使用 `<script setup>`**
   - 更简洁的语法
   - 更好的运行时性能
   - 更好的 IDE 支持

2. **合理拆分组合函数**
   - 按照功能模块拆分
   - 保持单一职责原则

3. **响应式数据的选择**
   - 简单数据类型使用 `ref`
   - 复杂对象使用 `reactive`

## 总结
Composition API 是 Vue 3 中一个强大的特性，它提供了更灵活的代码组织方式和更好的逻辑复用能力。通过合理使用 Composition API，我们可以写出更易维护、更易测试的 Vue 应用。