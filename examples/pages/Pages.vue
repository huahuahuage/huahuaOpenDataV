<template>
  <div class="page-list">
    <n-card>
      <n-button type="primary" :loading="loading" @click="handleCreate"> 新建 </n-button>
    </n-card>

    <div v-for="item in layoutList" :key="item.id" class="card">
      <n-card :key="item.id" hoverable>
        <template #header>
          <n-ellipsis style="max-width: 8rem; font-size: 1rem">
            {{ item.name }}
          </n-ellipsis>
        </template>
        <template #cover>
          <img
            :src="previewIcon(item.thumbnail)"
            class="thumbnail-image"
            alt=""
            @click="handleView(item)"
          />
        </template>
        <template #action>
          <div class="options">
            <n-tooltip trigger="hover">
              <template #trigger>
                <n-button quaternary @click="handleEdit(item)">
                  <template #icon>
                    <x-icon name="edit" />
                  </template>
                </n-button>
              </template>
              <span>编辑</span>
            </n-tooltip>
            <n-dropdown
              trigger="hover"
              :options="options"
              :show-arrow="true"
              @select="(key) => handleSelect(key, item)"
            >
              <n-button quaternary>
                <template #icon>
                  <x-icon name="setting" />
                </template>
              </n-button>
            </n-dropdown>
          </div>
        </template>
      </n-card>
    </div>
  </div>
</template>

<script setup lang="ts">
import type { DropdownOption } from 'naive-ui'
import { NButton, NCard, NDropdown, NEllipsis, NTooltip } from 'naive-ui'
import { XIcon } from 'open-data-v/designer'
import { h, onMounted, ref } from 'vue'
import { useRouter } from 'vue-router'

import type { SimpleLayoutData } from '@/api/pages'
import { deletePageApi, getPageListApi } from '@/api/pages'
import defaultImg from '@/assets/default.png'
import { message } from '@/utils/message'
import { Logger } from '@/utils/utils'

const router = useRouter()
const loading = ref<boolean>(false)
const layoutList = ref<SimpleLayoutData[]>([])
const options: DropdownOption[] = [
  {
    key: 'preview',
    label: '预览',
    icon: renderIcon('previewOpen')
  },
  {
    key: 'delete',
    label: '删除',
    icon: renderIcon('delete')
  }
]

onMounted(async () => {
  await initUI()
})

function renderIcon(icon: string) {
  return () =>
    h(XIcon, {
      name: icon
    })
}

const initUI = async (): Promise<void> => {
  try {
    const resp = await getPageListApi()
    if (resp.data) {
      layoutList.value = resp.data
    }
  } catch (_) {
    message.error('页面数据请求异常')
  }
}

const handleSelect = async (key: string | number, item: SimpleLayoutData) => {
  if (key === 'preview') {
    await router.push({
      name: 'PageView',
      params: {
        index: item.id
      }
    })
  } else if (key === 'delete') {
    try {
      await deletePageApi(item.id as string)
      await initUI()
    } catch (e: any) {
      Logger.log(e?.message || e)
      message.error('删除失败')
    }
  }
}

const handleCreate = async () => {
  if (loading.value) {
    return false
  }

  loading.value = true
  await router.push({
    name: 'Create'
  })
}

const handleEdit = async (item: SimpleLayoutData) => {
  if (loading.value) {
    return false
  }

  loading.value = true
  await router.push({
    name: 'Editor',
    params: {
      index: item.id
    }
  })
}

const handleView = async (item: SimpleLayoutData) => {
  await router.push({
    name: 'PageView',
    params: {
      index: item.id
    }
  })
}

const previewIcon = (icon: string | undefined) => {
  return icon ? icon : defaultImg
}
</script>

<style lang="less" scoped>
.page-list {
  @apply flex h-full mx-auto flex-wrap relative;
  align-content: flex-start;

  .card {
    @apply p-2 w-60;
    padding-left: 0;
    transition: all 0.3s ease;
    transform-origin: center;
  }
}

.options {
  @apply flex justify-between items-center p-2;
}
:deep(.n-card__action) {
  padding: 0;
}

:deep(.n-card) {
  .n-card-header {
    padding: 5px 0;
    transform: translate(15%, 0);
  }
}

.thumbnail-image {
  width: 14vw;
  height: 16vh;
}
</style>
