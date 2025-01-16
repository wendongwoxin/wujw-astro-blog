---
title: "深入理解 Vue 3 响应式系统"
description: "详细解析 Vue 3 响应式系统的实现原理、工作流程及核心概念"
pubDate: "2024-06-21"
heroImage: "/images/vue-3-reactivity.png"
---

## 响应式系统概述

Vue 3 的响应式系统是基于 Proxy 实现的，它能够追踪数据的变化并自动更新相关的视图。让我们从最简单的实现开始，逐步深入理解它的工作原理。

### 基础实现

```js
// 最简单的响应式实现
function reactive(obj) {
  return new Proxy(obj, {
    get(target, key) {
      track(target, key)
      return target[key]
    },
    set(target, key, value) {
      target[key] = value
      trigger(target, key)
      return true
    }
  })
}
```

## 核心概念

### 1. 依赖收集（track）

```js
// 存储依赖关系
const targetMap = new WeakMap()

function track(target, key) {
  if (!activeEffect) return
  
  let depsMap = targetMap.get(target)
  if (!depsMap) {
    targetMap.set(target, (depsMap = new Map()))
  }
  
  let dep = depsMap.get(key)
  if (!dep) {
    depsMap.set(key, (dep = new Set()))
  }
  
  dep.add(activeEffect)
}
```

### 2. 触发更新（trigger）

```js
function trigger(target, key) {
  const depsMap = targetMap.get(target)
  if (!depsMap) return
  
  const dep = depsMap.get(key)
  if (dep) {
    dep.forEach(effect => effect())
  }
}
```


### 3. 副作用函数（effect）

```js
let activeEffect = null

function effect(fn) {
  const _effect = function() {
    activeEffect = _effect
    fn()
    activeEffect = null
  }
  
  _effect()
  return _effect
}
```

## 响应式 API

### 1. ref

```js
function ref(value) {
  const refObject = {
    get value() {
      track(refObject, 'value')
      return value
    },
    set value(newValue) {
      value = newValue
      trigger(refObject, 'value')
    }
  }
  return refObject
}
```

### 2. reactive

```js
function reactive(target) {
  // 处理嵌套对象
  const handler = {
    get(target, key, receiver) {
      const result = Reflect.get(target, key, receiver)
      track(target, key)
      return typeof result === 'object' ? reactive(result) : result
    },
    set(target, key, value, receiver) {
      const oldValue = target[key]
      const result = Reflect.set(target, key, value, receiver)
      if (oldValue !== value) {
        trigger(target, key)
      }
      return result
    }
  }
  
  return new Proxy(target, handler)
}
```

## 工作流程

1. **依赖收集阶段**：
```js
const counter = reactive({ count: 0 })

effect(() => {
  console.log(counter.count) // 访问属性时触发 track
})
```

2. **更新触发阶段**：
```js
counter.count++ // 修改属性时触发 trigger
```

## 进阶特性

### 1. 计算属性

```js
function computed(getter) {
  let value
  let dirty = true
  
  const effect = createEffect(getter, {
    lazy: true,
    scheduler: () => {
      dirty = true
    }
  })
  
  return {
    get value() {
      if (dirty) {
        value = effect()
        dirty = false
      }
      return value
    }
  }
}
```

### 2. 监听器

```js
function watch(source, callback) {
  effect(() => {
    const newValue = source()
    callback(newValue)
  })
}
```

## 性能优化

### 1. 依赖收集优化

```js
// 使用 WeakMap 避免内存泄漏
const targetMap = new WeakMap()

// 使用 Set 去重
const dep = new Set()
```

### 2. 调度优化

```js
function trigger(target, key) {
  const effects = new Set()
  const computedEffects = new Set()
  
  // 分开处理普通效果和计算属性
  const add = (effectsToAdd) => {
    if (effectsToAdd) {
      effectsToAdd.forEach(effect => {
        if (effect.computed) {
          computedEffects.add(effect)
        } else {
          effects.add(effect)
        }
      })
    }
  }
  
  // 先触发计算属性，再触发普通效果
  computedEffects.forEach(effect => effect())
  effects.forEach(effect => effect())
}
```

## 常见问题解析

### 1. 为什么需要 ref？

ref 主要用于处理基本类型的响应式：
```js
const count = ref(0)
// 等价于
const count = reactive({ value: 0 })
```

### 2. reactive 的局限性

```js
const obj = reactive({ count: 0 })
let { count } = obj // 解构后失去响应性
```

### 3. 嵌套响应式

```js
const state = reactive({
  user: {
    name: 'John',
    age: 20
  }
})
// 嵌套对象也是响应式的
```

## 最佳实践

1. **选择合适的响应式 API**：
```js
// 基本类型用 ref
const count = ref(0)

// 对象类型用 reactive
const state = reactive({ count: 0 })
```

2. **避免响应式丢失**：
```js
// 错误方式
const { count } = reactive({ count: 0 })

// 正确方式
const state = reactive({ count: 0 })
const count = computed(() => state.count)
```

3. **性能优化**：
```js
// 大数据列表考虑使用 shallowRef 或 shallowReactive
const list = shallowRef([...largeArray])
```

## 调试技巧

```js
// 追踪依赖收集
effect(() => {
  console.log('effect run')
  console.trace() // 查看调用栈
  counter.count
})
```

通过深入理解 Vue 3 的响应式系统，我们可以更好地使用这些特性，写出更高效的代码。记住，响应式系统是 Vue 的核心特性之一，掌握它对于理解 Vue 的工作原理至关重要。


## 🎯 常见面试题

### 1. Vue 3 为什么使用 Proxy 代替 Object.defineProperty？

**答案：**
Proxy 相比 Object.defineProperty 有以下优势：
1. **能够监听数组变化**：不需要重写数组方法
2. **能够监听对象属性的添加和删除**：不需要通过 Vue.set/delete
3. **能够监听嵌套对象**：不需要递归遍历
4. **性能更好**：懒处理，访问时才进行代理

```js
// Object.defineProperty 方式
Object.defineProperty(obj, 'key', {
  get() {},
  set() {}
})

// Proxy 方式
new Proxy(obj, {
  get() {},
  set() {}
})
```


### 2. computed 和 watch 的区别？

**答案：**
1. **功能定位**：
   - computed 用于数据计算，有缓存
   - watch 用于副作用操作，无缓存

2. **执行时机**：
   - computed 懒执行，值被访问时才更新
   - watch 立即执行，数据变化时就触发

```js
// computed 示例
const fullName = computed(() => firstName.value + ' ' + lastName.value)

// watch 示例
watch(source, (newValue, oldValue) => {
  // 执行副作用
})
```

### 3. Vue 3 响应式系统的性能优化策略有哪些？

**答案：**
1. **懒收集依赖**：
   - 只在访问时才收集依赖
   - 避免不必要的依赖收集

2. **调度优化**：
   - 使用微任务队列
   - 合并多次更新

3. **内存优化**：
   - 使用 WeakMap 存储依赖
   - 及时清理不需要的依赖
