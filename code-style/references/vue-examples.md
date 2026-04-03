# Vue 组件完整示例

## 示例 1: 表单编辑页面

```vue
<script setup lang="ts">
import type { FormInstance, FormRules } from 'element-plus'
import type { SaveModConfigPayload } from '@/api/mod/config'
import type { UserInfo } from '@/api/user'
import { createModConfig, getModConfigDetail, updateModConfig } from '@/api/mod/config'
import { assignByKeys, getFullImageUrl, getRelativeUrl } from '@/utils/tool'

const ImageUpload = defineAsyncComponent(() => import('@/components/ImageUpload.vue'))
const RichText = defineAsyncComponent(() => import('@/components/RichText.vue'))
const UserSelector = defineAsyncComponent(() => import('@/components/UserSelector.vue'))

const route = useRoute()

const id = computed(() => Number(route.params.id) || undefined)
const isEdit = computed(() => !!id.value)

const [form] = useResetReactive<SaveModConfigPayload>({
  id: undefined,
  game_id: 0,
  title: '',
  description: '',
  remark: '',
  cover_image: '',
  creator_id: 0,
  creator_name: '',
  is_online_mod: 1,
  is_hot: 0,
  is_recommended: 0,
  sort: 0,
  tags: [],
})

const { fetchGameOptions, gameOptions, fetchModTags, modTags, OnlineModTypeEnums } = useEnums()
fetchGameOptions().then(() => {
  if (gameOptions.value.length && !isEdit.value) {
    form.game_id = gameOptions.value[0].id
  }
})
fetchModTags()

const rules: FormRules<SaveModConfigPayload> = {
  title: [{ required: true, message: '请输入 Mod 名称', trigger: 'blur' }],
  tags: [{ required: true, type: 'array', message: '请选择 Mod 类型', trigger: 'change' }],
  game_id: [{ required: true, message: '请选择所属游戏', trigger: 'change' }],
  creator_name: [{ required: true, message: '请输入输入 Mod 作者', trigger: 'blur' }],
  is_online_mod: [{ required: true, message: '请选择是否联机MOD', trigger: 'change' }],
  cover_image: [{ required: true, message: '请上传封面图', trigger: 'change' }],
}

async function loadEditData(editId: number) {
  const res = await getModConfigDetail({ id: editId })
  if (res.code !== 200) {
    return
  }

  const { tags = [], cover_image, ...info } = res.data ?? {}

  assignByKeys(form, {
    ...info,
    cover_image: getFullImageUrl(cover_image),
    tags: tags.map(item => item.id),
  })
}

if (isEdit.value && id.value) {
  loadEditData(id.value)
}

const userSelectorRef = useTemplateRef('userSelectorRef')
function openSelUserDialog() {
  userSelectorRef.value?.openDialog()
}

function setCreator(data: UserInfo) {
  form.creator_id = data.id
  form.creator_name = data.nickname
}

const router = useRouter()
function goBack() {
  router.push('/mod/config/list')
}

const formRef = useTemplateRef<FormInstance>('formRef')
const submitLoading = shallowRef(false)

function submitForm() {
  formRef.value?.validate(async (valid) => {
    if (!valid) {
      ElMessage.warning('请检查必填项')
      return
    }

    submitLoading.value = true

    // 提交：完整URL提取相对路径保存给后端
    const submitData = { ...form }
    submitData.cover_image = getRelativeUrl(submitData.cover_image)

    let api
    if (isEdit.value && form.id) {
      api = updateModConfig({ ...submitData, id: submitData.id! })
    }
    else {
      api = createModConfig(submitData)
    }

    api.then((res) => {
      if (res.code === 200) {
        ElMessage.success('提交成功')
        router.push({ name: 'mod.config.list' })
      }
    }).finally(() => {
      submitLoading.value = false
    })
  })
}
</script>

<template>
  <div>
    <el-card>
      <template #header>
        <div class="flex items-center justify-between">
          <span class="text-base font-medium">
            {{ isEdit ? '编辑 Mod 配置' : '新建 Mod 配置' }}
          </span>
          <el-button link type="primary" @click="goBack">
            返回列表
          </el-button>
        </div>
      </template>

      <el-form ref="formRef" :model="form" :rules="rules" label-position="top">
        <el-row :gutter="20">
          <el-col :span="12">
            <el-form-item label="Mod 名称" prop="title">
              <el-input v-model="form.title" placeholder="请输入 Mod 名称" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="Mod 分类标签" prop="tags">
              <el-select
                v-model="form.tags"
                multiple
                filterable
                collapse-tags
                collapse-tags-tooltip
                placeholder="请选择 Mod 类型"
                class="w-full"
              >
                <el-option
                  v-for="item in modTags"
                  :key="item.id"
                  :label="item.name"
                  :value="item.id"
                />
              </el-select>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>

      <template #footer>
        <div class="flex items-center justify-end gap-2">
          <el-button @click="goBack">
            取消
          </el-button>
          <el-button type="primary" :loading="submitLoading" @click="submitForm">
            {{ isEdit ? '保存' : '提交' }}
          </el-button>
        </div>
      </template>
    </el-card>

    <UserSelector ref="userSelectorRef" @select="setCreator" />
  </div>
</template>
```

## 示例 2: 对话框组件

```vue
<script setup lang="ts">
import type { FormInstance, FormRules } from 'element-plus'
import type { ModConfigVersion, SaveModConfigVersion } from '@/api/mod/configVersion'
import type { ModResource } from '@/api/mod/resource'
import type { DialogType } from '@/types'
import { Icon } from '@iconify/vue'
import { createModConfigVersion, editModConfigVersion, getModConfigVersionResource } from '@/api/mod/configVersion'
import { assignByKeys } from '@/utils/tool'

const emit = defineEmits<{
  refreshData: []
}>()

const ModResourceList = defineAsyncComponent(() => import('@/components/mod/ResourceList.vue'))
const ResourceSelectDialog = defineAsyncComponent(() => import('./ResourceSelectDialog.vue'))

const [form, resetForm] = useResetReactive<SaveModConfigVersion>({
  id: undefined,
  mod_id: 0,
  version: '',
  description: '',
  sort: 0,
  resource: [],
})

const rules: FormRules<typeof form> = {
  version: [{ required: true, message: '请输入版本号', trigger: 'blur' }],
  description: [{ required: true, message: '请输入版本描述', trigger: 'blur' }],
  resource: [{ required: true, type: 'array', message: '请添加资源', trigger: 'change' }],
}

// 已选择的资源
const selectedResources = ref<ModResource[]>([])
function getVersionResourceList(id: number) {
  getModConfigVersionResource({ id }).then((res) => {
    if (res.code === 200) {
      selectedResources.value = res.data?.list ?? []
    }
  })
}

const dialogVisible = shallowRef(false)
const formRef = useTemplateRef<FormInstance>('formRef')
const modId = shallowRef<number>(0)
const isEdit = shallowRef(false)

function openDialog(type: DialogType, mId: number, data?: ModConfigVersion) {
  resetForm()
  isEdit.value = type !== 'add'
  modId.value = mId
  selectedResources.value = []

  if (type === 'edit' && data?.id) {
    assignByKeys(form, data)
    getVersionResourceList(data.id)
  }

  dialogVisible.value = true
  nextTick(() => {
    formRef.value?.clearValidate()
  })
}

// 资源选择弹窗
const resourceSelectDialogRef = useTemplateRef('resourceSelectDialogRef')
function openResourceDialog() {
  resourceSelectDialogRef.value?.openDialog(selectedResources.value)
}

function handleResourceConfirm(resources: ModResource[]) {
  selectedResources.value = resources
  form.resource = resources.map(v => v.id).filter((id): id is number => id !== undefined)
}

// 移除资源
function removeResource(id: number) {
  selectedResources.value = selectedResources.value.filter(item => item.id !== id)
  form.resource = selectedResources.value.map(item => item.id).filter((id): id is number => id !== undefined)
}

const loading = ref(false)

// 提交
async function submitForm() {
  if (!formRef.value) {
    return
  }

  await formRef.value.validate(async (valid) => {
    if (!valid) {
      return
    }

    loading.value = true
    form.mod_id = modId.value

    let api
    if (isEdit.value && form.id) {
      api = editModConfigVersion({ ...form, id: form.id })
    }
    else {
      api = createModConfigVersion(form)
    }

    api.then((res) => {
      if (res.code === 200) {
        ElMessage.success(isEdit.value ? '编辑成功' : '创建成功')
        dialogVisible.value = false
        emit('refreshData')
      }
    }).finally(() => {
      loading.value = false
    })
  })
}

defineExpose({
  openDialog,
})
</script>

<template>
  <el-dialog
    v-model="dialogVisible"
    :title="isEdit ? '编辑版本' : '新建版本'"
    width="880px"
    :close-on-click-modal="false"
    destroy-on-close
  >
    <el-form
      ref="formRef"
      :model="form"
      :rules="rules"
      label-position="top"
    >
      <el-form-item label="版本描述" prop="description">
        <el-input
          v-model.trim="form.description"
          type="textarea"
          :autosize="{ minRows: 2, maxRows: 10 }"
          placeholder="请输入版本描述"
        />
      </el-form-item>
    </el-form>

    <template #footer>
      <div class="flex justify-end gap-2">
        <el-button @click="dialogVisible = false">
          取消
        </el-button>
        <el-button type="primary" :loading="loading" @click="submitForm">
          提交
        </el-button>
      </div>
    </template>

    <ResourceSelectDialog ref="resourceSelectDialogRef" @confirm="handleResourceConfirm" />
  </el-dialog>
</template>
```
