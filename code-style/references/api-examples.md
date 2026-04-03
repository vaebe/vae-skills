# API 接口定义示例

## 示例 1: 标准 CRUD API 模块（推荐）

```ts
// src/api/mod/tag.ts
import type { ResultData, ResultPageListData } from '../base'
import type { PaginationParameter } from '@/types'
import Api from '../base'

// 1. 列表查询参数类型
export interface GetModTagListReq extends Partial<PaginationParameter> {
  keywords?: string // 关键词搜索，支持字段：标签名称
  status?: number // 状态：0-禁用 1-启用，不传查全部
}

// 2. 数据实体类型（用于表单提交和列表展示）
export interface ModTag {
  id?: number // 标签id
  name: string // 名称
  color: string // 色值
  sort: number // 排序权重
  status?: number // 状态
  mod_count?: number // mod数量统计
  mod_group_count?: number // 模组数量统计
  create_time?: string
}

// 3. API 函数
export function getModTagList(params: GetModTagListReq): Promise<ResultPageListData<ModTag[]>> {
  return Api.get('/admin/mod/tag', { params })
}

export function createModTag(data: ModTag): Promise<ResultData> {
  return Api.post('/admin/mod/tag', data)
}

export function editModTag(data: ModTag): Promise<ResultData> {
  const { id, ...info } = data
  return Api.put(`/admin/mod/tag/${id}`, info)
}

export function deleteModTag(params: { id: number }): Promise<ResultData> {
  return Api.delete(`/admin/mod/tag/${params.id}`)
}

export function updateModTagStatus(data: { id: number }): Promise<ResultData<{ status: number }>> {
  return Api.put(`/admin/mod/tag/status/${data.id}`)
}
```

## 示例 2: 复杂业务 API 模块

```ts
// src/api/mod/config.ts
import type { ResultData, ResultPageListData } from '../base'
import type { PaginationParameter } from '@/types'
import Api from '../base'

export interface SaveModConfigPayload {
  id?: number // mod 版本 id
  game_id: number // mod所属游戏ID
  title: string // MOD名称
  description: string // MOD描述，富文本
  remark: string // MOD备注,最多200字符
  cover_image: string // 封面
  creator_id: number // mod作者 id
  creator_name: string // mod作者
  is_online_mod: number // 是否联机MOD: 1否 2是 3其他
  is_hot: number // 是否热门: 0否 1是
  is_recommended: number // 是否上MOD推荐榜: 0否 1是
  sort: number // 排序权重
  tags: number[] // 分类标签
}

interface Game {
  id: number
  name: string
  cover: string
}

export interface Tag {
  id: number // 标签id
  name: string // 标签名称
}

export interface ModConfigListItem extends Omit<SaveModConfigPayload, 'tags'> {
  create_time: string
  download_count: number // 下载数量
  game: Game // 所属游戏
  id: number // MOD id
  is_online: number // 上下架状态 0-下架 1-上架
  mod_group_count: number // 关联模组数量
  tags: Tag[] // 所属标签
  uncompress: string // 解压路径
  update_time: string
  visit_times: number // 访问数量
}

export interface GetModConfigListParams extends PaginationParameter {
  keyword?: string
  is_online?: number
  is_hot?: number
  is_recommended?: number
}

export function getModConfigList(params: GetModConfigListParams): Promise<ResultPageListData<ModConfigListItem[]>> {
  return Api.get('/admin/mod/config/list', { params })
}

export function getModConfigDetail(params: { id: number }): Promise<ResultData<ModConfigListItem>> {
  return Api.get('/admin/mod/config/detail', { params })
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

export function onlineModConfig(params: { id: number }): Promise<ResultData> {
  return Api.post('/admin/mod/config/online', null, { params })
}

export function offlineModConfig(params: { id: number }): Promise<ResultData> {
  return Api.post('/admin/mod/config/offline', null, { params })
}
```

## 示例 2: Composables

### useResetReactive

```ts
// src/composables/useResetReactive.ts
import { cloneDeep } from 'lodash-es'

// 重置 reactive 为默认值
export function useResetReactive<T extends object>(
  value: T,
  clone: (value: T) => T = cloneDeep,
) {
  const state = reactive(clone(value)) as T

  const reset = () => {
    const newState = clone(value)
    Object.keys(state).forEach(key => delete state[key as keyof T])
    Object.assign(state, newState)
  }

  return [state, reset] as [typeof state, typeof reset]
}
```

### usePageList

```ts
// src/composables/usePageList.ts
import type { AnyObject, PaginationParameter } from '@/types'
import { dataClone } from '@/utils/tool'

interface PageOptions {
  searchForm?: AnyObject
  getListApi: (params: any) => Promise<AnyObject>
  removeRowApi?: (params: any) => Promise<AnyObject>
  customQueryParameters?: () => Record<string, any>
  resetFunc?: () => void
  sizeChangeFunc?: () => void
  currentChangeFunc?: () => void
}

// 列表
export function usePageList(opts: PageOptions) {
  const {
    searchForm = {},
    getListApi,
    removeRowApi,
    customQueryParameters = () => ({}),
    resetFunc,
    sizeChangeFunc,
    currentChangeFunc,
  } = opts

  const page = reactive<PaginationParameter>({
    page_size: 10,
    page: 1,
    total: 0,
  })

  // 格式化查询参数
  const formattedQueryParameter = () => {
    const formInfo: AnyObject = {}
    Object.keys(searchForm).forEach((item) => {
      const val = searchForm[item]
      if (!!val || val === 0 || val === false) {
        formInfo[item] = val
      }
    })
    return formInfo
  }

  const listLoading = ref(false)
  const listResponse = ref<AnyObject>()
  const tableData = ref<any[]>([])

  function getList() {
    listLoading.value = true

    const params = {
      ...page,
      ...formattedQueryParameter(),
      ...customQueryParameters(),
    }

    delete params.total

    getListApi(params).then((res) => {
      listResponse.value = res

      if (res.code === 200) {
        const { list = [], total = 0 } = res?.data
        tableData.value = list
        page.total = total
      }
    }).catch(() => {
      tableData.value = []
      page.total = 0
      listResponse.value = undefined
    }).finally(() => {
      listLoading.value = false
    })
  }

  function handleSizeChange(pageSize: number) {
    page.page_size = pageSize
    sizeChangeFunc?.()
    getList()
  }

  function handleCurrentChange(pageNo: number) {
    page.page = pageNo
    currentChangeFunc?.()
    getList()
  }

  const rawSearchForm = dataClone(searchForm)

  function reset() {
    Object.keys(searchForm).forEach(key => delete searchForm[key])
    Object.assign(searchForm, rawSearchForm)
    resetFunc?.()
    handleCurrentChange(1)
  }

  function removeRow(params: any, infoText?: string, delSuccessInfo?: string) {
    if (!removeRowApi) {
      ElMessage.warning('请配置 removeRowApi 调用')
      return
    }

    ElMessageBox.confirm(
      infoText ?? '此操作将永久删除该数据, 是否继续?',
      '提示',
      {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      },
    )
      .then(async () => {
        const res = await removeRowApi?.(params)
        if (res?.code === 200) {
          ElMessage.success(delSuccessInfo ?? '删除成功')
          handleCurrentChange(1)
        }
      })
  }

  return {
    listLoading,
    listResponse,
    reset,
    page,
    tableData,
    handleSizeChange,
    handleCurrentChange,
    removeRow,
  }
}

export const DialogTypeObj = {
  add: '新增',
  edit: '编辑',
  view: '查看',
}
```

### useEnums

```ts
// src/composables/useEnums.ts
import type { ModGameOption } from '@/api/game/config'
import type { ModTag } from '@/api/mod/tag'
import type { AnyObject } from '@/types'
import { getModGameOptions } from '@/api/game/config'
import { getModTagList } from '@/api/mod/tag'

export interface EnumsItem {
  code: string | number
  name: string
}

// Banner 管理：跳转类型
const bannerLinkTypeEnums: EnumsItem[] = [
  { code: 1, name: '内部模块' },
  { code: 2, name: '内部浏览器' },
  { code: 3, name: '外部浏览器' },
]

// 通用状态：1=上架，0=下架
const CommonStatusEnums: EnumsItem[] = [
  { code: 1, name: '上架' },
  { code: 0, name: '下架' },
]

// 通用是否：1=是，0=否
const YesNoEnums: EnumsItem[] = [
  { code: 1, name: '是' },
  { code: 0, name: '否' },
]

// 是否联机MOD：1=否，2=是，3=其他
const OnlineModTypeEnums: EnumsItem[] = [
  { code: 1, name: '否' },
  { code: 2, name: '是' },
  { code: 3, name: '其他' },
]

// MOD资源审核状态：-1=全部，0=待审核，1=已通过，2=已拒绝，3=已发布
const ModResourceStatusEnums: EnumsItem[] = [
  { code: -1, name: '全部状态' },
  { code: 0, name: '待审核' },
  { code: 1, name: '已通过' },
  { code: 2, name: '已拒绝' },
  { code: 3, name: '已发布' },
]

// 枚举数据转换成以指定 key 为键的对象
function arrEnumsToObjEnums(list: AnyObject[] | readonly AnyObject[], key = 'code') {
  const obj = {} as typeof list[0]
  list.forEach((item) => {
    obj[item[key]] = item
  })
  return obj
}

interface GetEnumNameParams {
  key: string | number
  list: Record<string, any>[] | Readonly<Record<string, any>[]>
  code?: string
  name?: string
}

// 转换枚举-获取枚举名称
function getEnumName(config: GetEnumNameParams) {
  const { key, list, code = 'code', name = 'name' } = config

  if (!list) {
    return '-'
  }

  const res = list.find(item => item[code] === key)?.[name]
  return res || '-'
}

// 游戏选项
const gameOptions = ref<ModGameOption[]>([])
// 获取游戏选项
async function fetchGameOptions() {
  gameOptions.value = []
  return getModGameOptions().then((res) => {
    if (res.code === 200) {
      gameOptions.value = res?.data ?? []
    }
  })
}

// mod 分类标签
const modTags = ref<ModTag[]>([])

// 获取 mod 分类
async function fetchModTags() {
  modTags.value = []

  return getModTagList({ page: 1, page_size: 999999, status: 1 }).then((res) => {
    if (res.code === 200) {
      modTags.value = res?.data?.list ?? []
    }
  })
}

export function useEnums() {
  return {
    bannerLinkTypeEnums,
    CommonStatusEnums,
    YesNoEnums,
    OnlineModTypeEnums,
    ModResourceStatusEnums,
    arrEnumsToObjEnums,
    getEnumName,
    gameOptions,
    fetchGameOptions,
    modTags,
    fetchModTags,
  }
}
```
