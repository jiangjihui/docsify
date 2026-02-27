# Vue 3

Vue 3 是 Vue.js 的最新主要版本，于 2020 年 9 月正式发布。它带来了许多重要的新特性和改进，包括更好的性能、更小的 bundle 体积、更好的 TypeScript 支持以及全新的 Composition API。

## Vue 3 新特性

### 1. 性能提升

- **更小的体积**：Vue 3 的核心库经过 tree-shaking 优化，体积比 Vue 2 更小
- **更快的渲染**：使用 Proxy 替代 Object.defineProperty，响应式系统性能提升
- **更好的 tree-shaking**：大多数 API 都是 tree-shakeable 的
- **Fragments**：组件可以返回多个根节点

### 2. Composition API

Composition API 是 Vue 3 最重要的新特性，它提供了一种更灵活的方式来组织组件逻辑。

```vue
<script setup>
import { ref, computed, onMounted } from 'vue'

// 响应式状态
const count = ref(0)

// 计算属性
const doubled = computed(() => count.value * 2)

// 方法
function increment() {
  count.value++
}

// 生命周期钩子
onMounted(() => {
  console.log('组件已挂载')
})
</script>

<template>
  <button @click="increment">
    Count: {{ count }}, Doubled: {{ doubled }}
  </button>
</template>
```

### 3. Teleport

Teleport 允许将组件的模板部分渲染到 DOM 中的其他位置，非常适合模态框、对话框等场景。

```vue
<script setup>
import { ref } from 'vue'

const showModal = ref(false)
</script>

<template>
  <button @click="showModal = true">打开弹窗</button>

  <Teleport to="body">
    <div v-if="showModal" class="modal">
      <div class="content">
        <p>这是一个模态框</p>
        <button @click="showModal = false">关闭</button>
      </div>
    </div>
  </Teleport>
</template>

<style scoped>
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
}
.content {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: white;
  padding: 20px;
}
</style>
```

### 4. Suspense

Suspense 用于处理异步组件的加载状态。

```vue
<script setup>
import { defineAsyncComponent } from 'vue'

const AsyncComponent = defineAsyncComponent(() =>
  import('./AsyncComponent.vue')
)
</script>

<template>
  <Suspense>
    <template #default>
      <AsyncComponent />
    </template>
    <template #fallback>
      <div>加载中...</div>
    </template>
  </Suspense>
</template>
```

### 5. Fragments（片段）

Vue 3 组件可以返回多个根节点。

```vue
<template>
  <header>头部</header>
  <main>主体内容</main>
  <footer>底部</footer>
</template>
```

## 创建 Vue 3 工程

### 使用 Vite（推荐）

Vite 是 Vue 3 官方推荐的构建工具，开发体验极佳。

```bash
# 创建项目
npm create vue@latest my-vue-app

# 进入项目目录
cd my-vue-app

# 安装依赖
npm install

# 启动开发服务器
npm run dev
```

### 使用 Vue CLI

```bash
# 安装 Vue CLI
npm install -g @vue/cli

# 创建项目
vue create my-vue-app

# 进入项目目录
cd my-vue-app

# 启动项目
npm run serve
```

## Vue 3 响应式原理

Vue 3 使用 ES6 Proxy 来实现响应式系统，相比 Vue 2 的 Object.defineProperty 有以下优势：

- **可以直接监听数组的变化**
- **不需要递归遍历对象属性**
- **可以监听动态添加的属性**

```javascript
import { reactive, ref } from 'vue'

// 响应式对象
const state = reactive({
  name: 'Vue 3',
  count: 0
})

// ref 用于原始类型
const count = ref(0)

// 改变值
count.value = 1
state.count++
```

## Composition API 详解

### ref 和 reactive

```javascript
import { ref, reactive, toRefs } from 'vue'

// ref 用于基本类型
const name = ref('Vue 3')

// reactive 用于对象
const state = reactive({
  count: 0,
  user: {
    name: 'John',
    age: 30
  }
})

// 转换为 ref（解构响应式对象）
const { count, user } = toRefs(state)
```

### computed 和 watch

```javascript
import { ref, computed, watch, watchEffect } from 'vue'

const count = ref(0)

// 计算属性
const doubled = computed(() => count.value * 2)

// watch - 监听特定响应式引用
watch(count, (newVal, oldVal) => {
  console.log(`count 从 ${oldVal} 变为 ${newVal}`)
})

// watchEffect - 自动追踪依赖
watchEffect(() => {
  console.log(`count is: ${count.value}`)
})
```

### 生命周期钩子

在 Composition API 中，可以使用 `on` 前缀的函数来注册生命周期钩子：

| Vue 2 | Vue 3 Composition API |
|--------|----------------------|
| beforeCreate | - |
| created | - |
| beforeMount | onBeforeMount |
| mounted | onMounted |
| beforeUpdate | onBeforeUpdate |
| updated | onUpdated |
| beforeUnmount | onBeforeUnmount |
| unmounted | onUnmounted |
| errorCaptured | onErrorCaptured |
| renderTracked | onRenderTracked |
| renderTriggered | onRenderTriggered |

```javascript
import {
  onMounted,
  onUpdated,
  onUnmounted
} from 'vue'

onMounted(() => {
  console.log('组件挂载完成')
})

onUpdated(() => {
  console.log('组件更新完成')
})

onUnmounted(() => {
  console.log('组件卸载完成')
})
```

## Vue 3 组件通信

### Props 和 Emit

```vue
<!-- ChildComponent.vue -->
<script setup>
defineProps({
  title: String,
  count: {
    type: Number,
    required: true
  }
})

const emit = defineEmits(['update', 'delete'])

function handleClick() {
  emit('update', '新的值')
}
</script>

<template>
  <button @click="handleClick">更新</button>
</template>
```

```vue
<!-- ParentComponent.vue -->
<script setup>
import { ref } from 'vue'
import ChildComponent from './ChildComponent.vue'

const message = ref('Hello')

function handleUpdate(newVal) {
  console.log('收到更新:', newVal)
}
</script>

<template>
  <ChildComponent
    title="标题"
    :count="10"
    @update="handleUpdate"
  />
</template>
```

### Provide / Inject

```javascript
// 父组件
import { provide, ref } from 'vue'

const count = ref(0)
provide('count', count)

// 子组件
import { inject } from 'vue'

const count = inject('count')
```

### v-model

Vue 3 对 v-model 进行了改进，支持多个 v-model 绑定。

```vue
<!-- 子组件 -->
<script setup>
defineProps(['modelValue'])
defineEmits(['update:modelValue'])
</script>

<template>
  <input
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>
```

```vue
<!-- 父组件 -->
<script setup>
import { ref } from 'vue'
import ChildComponent from './ChildComponent.vue'

const text = ref('Hello')
</script>

<template>
  <ChildComponent v-model="text" />
</template>
```

### 多个 v-model

```vue
<!-- 子组件 -->
<script setup>
defineProps(['firstName', 'lastName'])
defineEmits(['update:firstName', 'update:lastName'])
</script>

<template>
  <input
    :value="firstName"
    @input="$emit('update:firstName', $event.target.value)"
  />
  <input
    :value="lastName"
    @input="$emit('update:lastName', $event.target.value)"
  />
</template>
```

```vue
<!-- 父组件 -->
<script setup>
import { ref } from 'vue'
import ChildComponent from './ChildComponent.vue'

const firstName = ref('John')
const lastName = ref('Doe')
</script>

<template>
  <ChildComponent
    v-model:firstName="firstName"
    v-model:lastName="lastName"
  />
</template>
```

## Vue 3 插槽新语法

### 具名插槽

```vue
<!-- 子组件 -->
<template>
  <div class="container">
    <header>
      <slot name="header" />
    </header>
    <main>
      <slot />
    </main>
    <footer>
      <slot name="footer" />
    </footer>
  </div>
</template>
```

```vue
<!-- 父组件 -->
<template>
  <ChildComponent>
    <template #header>
      <h1>标题</h1>
    </template>
    <template #default>
      <p>主要内容</p>
    </template>
    <template #footer>
      <p>底部信息</p>
    </template>
  </ChildComponent>
</template>
```

### 作用域插槽

```vue
<!-- 子组件 -->
<template>
  <slot :user="user" :age="age" />
</template>

<script setup>
const user = { name: 'Vue 3' }
const age = 3
</script>
```

```vue
<!-- 父组件 -->
<template>
  <ChildComponent v-slot="{ user, age }">
    <p>{{ user.name }} 已经诞生 {{ age }} 年</p>
  </ChildComponent>
</template>
```

## Vue 3 变化汇总

### 全局 API 变化

| Vue 2 | Vue 3 |
|-------|-------|
| Vue.component | app.component |
| Vue.directive | app.directive |
| Vue.filter | app.filter |
| Vue.use | app.use |
| Vue.mixin | app.mixin |
| Vue.config | app.config |
| new Vue() | createApp() |

```javascript
// Vue 2
import Vue from 'vue'
Vue.config.productionTip = false

// Vue 3
import { createApp } from 'vue'
const app = createApp(App)
app.config.productionTip = false
```

### 模板语法变化

- **v-for 中的 key**：建议使用唯一 id 而不是 index
- **v-if 和 v-for**：不再建议一起使用，建议使用 computed 过滤
- **组件根节点**：可以多个根节点（Fragments）

### 自定义指令生命周期变化

| Vue 2 | Vue 3 |
|-------|-------|
| bind | beforeMount |
| inserted | mounted |
| update | updated |
| componentUpdated | updated |
| unbind | unmounted |

## TypeScript 支持

Vue 3 从一开始就使用 TypeScript 重写，提供了更好的类型支持。

```typescript
// 使用 TypeScript 定义组件
import { defineComponent, ref, computed } from 'vue'

interface User {
  name: string
  age: number
}

export default defineComponent({
  setup() {
    const user = ref<User>({ name: 'Vue', age: 3 })

    const greeting = computed(() => `Hello, ${user.value.name}`)

    return { user, greeting }
  }
})
```

## 迁移策略

从 Vue 2 迁移到 Vue 3 可以使用以下工具：

1. **Vue Migration Build**：Vue 2.7 提供了向 Vue 3 兼容的 API
2. **vue-codemod**：自动转换代码
3. **Vue 3 兼容性构建**：部分 Vue 2 功能可以在 Vue 3 中使用

```bash
# 安装迁移构建
npm install @vue/compat
```
