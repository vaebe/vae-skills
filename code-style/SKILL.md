---
name: code-style
description: Vue3/Vite + Element Plus 个人代码风格规范。强制使用 defineAsyncComponent 异步导入组件、defineModel 双向绑定、行内单行注释 //、if 必须带花括号等。适用于 Vue3 后台管理项目代码生成、组件开发、API 接口定义。关键词：Vue3, script setup, Composition API, Element Plus, 异步组件, defineModel, 代码规范
---

# 个人代码风格规范

**Iron Law**: 生成的代码必须严格遵循以下风格规范，不允许使用与规范冲突的写法。

## 工作流程

```bash
Code Style Skill Progress:

- [ ] Step 1: 识别项目类型
  - [ ] 检查 package.json 确认框架 (Vue3/Vite/Nuxt)
  - [ ] 检查 UI 组件库 (Element Plus)
- [ ] Step 2: 应用代码风格 ⛔ BLOCKING
  - [ ] Vue 单文件组件规范
  - [ ] TypeScript 类型定义规范
  - [ ] API 接口定义规范
  - [ ] 工具函数和 Composable 规范
- [ ] Step 3: 代码审查 ⚠️ REQUIRED
  - [ ] 检查是否违反任何风格规则
  - [ ] 如发现违规，向用户说明并请求确认修改
  - [ ] 确认使用推荐的写法
- [ ] Step 4: 输出代码
```

## 1. Vue 单文件组件规范

### 1.1 组件导入 - 必须使用异步导入

**禁止** - 直接静态导入：

```vue
<script setup>
import ResourceSelectDialog from './components/ResourceSelectDialog.vue'
import UserSelector from '@/components/UserSelector.vue'
</script>
```

**必须** - 使用 `defineAsyncComponent`：

```vue
<script setup>
const ResourceSelectDialog = defineAsyncComponent(() => import('./components/ResourceSelectDialog.vue'))
const UserSelector = defineAsyncComponent(() => import('@/components/UserSelector.vue'))
</script>
```

**例外**: 类型导入和工具函数可以静态导入：

```vue
<script setup>
import type { FormInstance } from 'element-plus'
import type { UserInfo } from '@/api/user'
import { Icon } from '@iconify/vue'
import { assignByKeys } from '@/utils/tool'
</script>
```

### 1.2 路由组件导入

**推荐** - 使用动态导入：

```ts
{
  path: '/mod/config/add',
  component: () => import('@/views/mod/config/addOrEdit.vue')
}
```

### 1.3 组件双向绑定

**禁止** - 使用传统的 modelValue：

```vue
<script setup>
interface Props {
  modelValue: string
}
const emit = defineEmits<{
  'update:modelValue': [value: string]
}>()
</script>
```

**必须** - 使用 `defineModel()`：

```vue
<script setup>
const modelValue = defineModel<string>()
// 或使用自定义名称
const dialogVisible = defineModel<boolean>('visible')
</script>
```

### 1.4 模板引用

**推荐** - 使用 `useTemplateRef`：

```vue
<script setup>
const formRef = useTemplateRef<FormInstance>('formRef')
const userSelectorRef = useTemplateRef('userSelectorRef')
</script>

<template>
  <el-form ref="formRef">
  <UserSelector ref="userSelectorRef" />
</template>
```

### 1.5 响应式数据

**推荐** - 根据场景选择合适的响应式 API：

```vue
<script setup>
// 简单布尔值/字符串/数字 - 使用 ref
const dialogVisible = ref(false)
const loading = ref(false)
const isEdit = ref(false)

// 对象/数组 - 使用 reactive 或 useResetReactive
const form = reactive({ name: '' })
// 或使用 useResetReactive（支持重置）
const [form, resetForm] = useResetReactive<FormData>({ name: '' })
</script>
```

### 1.6 组件事件定义

**推荐** - 使用元组类型定义事件：

```vue
<script setup>
const emit = defineEmits<{
  confirm: [resources: ModResource[]]
  refreshData: []
  select: [data: UserInfo]
}>()
</script>
```

### 1.7 组件暴露方法

**推荐** - 使用 `defineExpose`：

```vue
<script setup>
function openDialog(type: DialogType, id: number) {
  // ...
}

function closeDialog() {
  // ...
}

defineExpose({
  openDialog,
  closeDialog,
})
</script>
```

## 2. TypeScript 类型定义规范

### 2.1 接口字段注释 - 必须使用行内单行注释

**禁止** - 使用 JSDoc 多行注释：

```ts
export interface SaveModConfigPayload {
  /**
   * mod 版本 id
   */
  id?: number
  /**
   * 描述
   */
  description?: string
}
```

**必须** - 使用行内单行注释：

```ts
export interface SaveModConfigPayload {
  id?: number // mod 版本 id
  description?: string // 描述
  game_id: number // mod所属游戏ID
  title: string // MOD名称
  is_online_mod: number // 是否联机MOD: 1否 2是 3其他
}
```

### 2.2 类型命名规范

- 接口使用 `PascalCase`
- API 参数类型以 `Params` 结尾
- API 响应数据类型以 `Item` 或 `Data` 结尾
- 表单数据类型以 `Payload` 结尾

```ts
// API 参数
export interface GetModConfigListParams extends PaginationParameter {
  keyword?: string
  is_online?: number
}

// 列表项
export interface ModConfigListItem {
  id: number
  title: string
}

// 表单提交数据
export interface SaveModConfigPayload {
  id?: number
  title: string
}
```

### 2.3 枚举定义

**推荐** - 使用数组对象形式：

```ts
export interface EnumsItem {
  code: string | number
  name: string
}

const CommonStatusEnums: EnumsItem[] = [
  { code: 1, name: '上架' },
  { code: 0, name: '下架' },
]
```

## 3. JavaScript/TypeScript 代码规范

### 3.1 if 语句 - 不允许简写

**禁止** - 省略花括号：

```ts
if (!form.icon)
  form.icon = form.cover
```

**必须** - 使用花括号：

```ts
if (!form.icon) {
  form.icon = form.cover
}
```

### 3.2 条件判断中的常量

**推荐** - 常量放在左边：

```ts
// 推荐
if (200 === res.code) {
  // ...
}

// 避免
if (res.code === 200) {
  // ...
}
```

### 3.3 可选链和默认值

**推荐** - 使用简洁的写法：

```ts
const list = res.data?.list ?? []
const total = res?.data?.total ?? 0
```

### 3.4 解构赋值

**推荐** - 合理使用解构：

```ts
const { tags = [], cover_image, ...info } = res.data ?? {}

// 函数参数解构
function handleConfirm({ id, name }: UserInfo) {
  // ...
}
```

## 4. API 接口定义规范

### 4.1 文件组织

按功能模块组织 API 文件：

```bash
src/api/
  ├── base.ts           # 基础请求配置
  ├── user.ts           # 用户相关
  ├── mod/
  │   ├── config.ts     # Mod 配置
  │   ├── resource.ts   # Mod 资源
  │   └── tag.ts        # Mod 标签
  └── game/
      └── config.ts     # 游戏配置
```

### 4.2 接口定义格式

```ts
import type { ResultData, ResultPageListData } from '../base'
import type { PaginationParameter } from '@/types'
import Api from '../base'

// 1. 类型定义
export interface SaveModConfigPayload {
  id?: number // ID
  title: string // 名称
}

export interface GetModConfigListParams extends PaginationParameter {
  keyword?: string
  is_online?: number
}

// 2. API 函数
export function getModConfigList(params: GetModConfigListParams): Promise<ResultPageListData<ModConfigListItem[]>> {
  return Api.get('/admin/mod/config/list', { params })
}

export function createModConfig(data: SaveModConfigPayload): Promise<ResultData> {
  return Api.post('/admin/mod/config/create', data)
}

export function updateModConfig(data: SaveModConfigPayload & { id: number }): Promise<ResultData> {
  return Api.post('/admin/mod/config/update', data)
}

export function deleteModConfig(data: { id: number }): Promise<ResultData> {
  return Api.post('/admin/mod/config/delete', data)
}
```

### 4.3 响应类型命名

```ts
// 分页列表响应
ResultPageListData<T>

// 单条数据响应
ResultData<T>

// 无数据响应
ResultData
```

## 5. Composable 规范

### 5.1 useResetReactive

用于创建可重置的响应式状态：

```ts
const [form, resetForm] = useResetReactive<FormData>({
  id: undefined,
  name: '',
  status: 1,
})

// 重置
resetForm()
```

### 5.2 usePageList

用于列表页面：

```ts
const { page, tableData, listLoading, handleCurrentChange, handleSizeChange, reset, removeRow } = usePageList({
  searchForm,
  getListApi: getModConfigList,
  removeRowApi: deleteModConfig,
})
```

### 5.3 useEnums

用于获取枚举数据：

```ts
const { CommonStatusEnums, gameOptions, fetchGameOptions } = useEnums()
fetchGameOptions()
```

## 6. 模板规范

### 6.1 属性绑定

**推荐** - 简写形式：

```vue
<template>
  <el-input v-model="form.name" />
  <el-input v-model.trim="form.keyword" />
  <el-select v-model="form.status" clearable>
  <el-button :loading="submitLoading" @click="submitForm">
</template>
```

### 6.2 事件处理

**推荐** - 直接调用或内联：

```vue
<template>
  <el-button @click="submitForm">提交</el-button>
  <el-input @keyup.enter="handleSearch" />
  <el-pagination 
    @current-change="handleCurrentChange"
    @size-change="handleSizeChange"
  />
</template>
```

### 6.3 列表渲染

**推荐** - 使用简洁的 key：

```vue
<template>
  <el-option
    v-for="item in gameOptions"
    :key="item.id"
    :label="item.name"
    :value="item.id"
  />
</template>
```

## 7. 反模式清单（禁止的写法）

| 禁止 | 推荐 |
| ----- | ----- |
| `import Comp from './Comp.vue'` | `const Comp = defineAsyncComponent(() => import('./Comp.vue'))` |
| JSDoc 多行注释 `/** */` | 行内单行注释 `//` |
| `if (condition) statement` | `if (condition) { statement }` |
| `modelValue` + `emit('update:modelValue')` | `defineModel()` |
| `const props = defineProps()`（script 中无需访问 props 时） | 直接使用 `defineProps<{}>()`（不赋值给变量） |

## 8. 预交付检查清单

- [ ] 所有 Vue 组件使用 `defineAsyncComponent` 导入
- [ ] 接口字段使用行内单行注释 `//`
- [ ] 所有 `if` 语句使用花括号
- [ ] 双向绑定使用 `defineModel()`
- [ ] 模板引用使用 `useTemplateRef`
- [ ] API 函数返回类型声明完整
- [ ] 类型命名符合规范
