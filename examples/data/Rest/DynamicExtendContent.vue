<template>
  <RestContent v-model:options="formData" @update:rest-options="formChange" @change="formChange">
    <template #data-select>
      <div class="flex flex-nowrap flex-row">
        <n-select
          :options="restDataList"
          :value="formData.id"
          class="selected"
          clearable
          placeholder="请选择数据"
          @update:value="selectedChange"
          @clear="clear"
        />
        <n-input v-if="mode === 'debug'" v-model:value="formData.title" class="title">
          <template #prefix>
            <x-icon name="api" />
          </template>
        </n-input>
        <n-space v-if="mode === 'debug'">
          <n-button-group class="save">
            <n-button @click="formData.id ? handleUpdate() : handleSave()">保存</n-button>
          </n-button-group>
        </n-space>
      </div>
    </template>
  </RestContent>
</template>
<script setup lang="ts">
import type { AxiosResponse } from 'axios'
import type { SelectOption } from 'naive-ui'
import { NButton, NButtonGroup, NInput, NSelect, NSpace } from 'naive-ui'
import { StaticKey, useEventBus } from 'open-data-v/base'
import { useRequest } from 'open-data-v/data/hooks/http'
import type { RestOption, RestResponse } from 'open-data-v/data/rest'
import {
  KVToRecordable,
  recordabletoKV,
  RequestMethod,
  requestOptionsToStore,
  RestContent
} from 'open-data-v/data/rest'
import { onMounted, reactive, ref } from 'vue'

import {
  createRestDataApi,
  getRestDataApi,
  getRestDataListApi,
  updateRestDataApi
} from '@/api/data'
import type { RestDataDetail } from '@/api/data/type'
import { message } from '@/utils/message'
import { Logger, uuid } from '@/utils/utils'

const getEmptyParams = () => {
  return [{ key: '', value: '', disable: false, id: uuid() }]
}

const props = withDefaults(
  defineProps<{
    options?: RestOption
    mode?: 'debug' | 'use'
  }>(),
  {
    options: () => {
      return {
        method: RequestMethod.GET,
        url: '/getRiskArea',
        headers: [{ key: '', value: '', disable: false, id: uuid() }],
        params: [{ key: '', value: '', disable: false, id: uuid() }],
        data: [{ key: '', value: '', disable: false, id: uuid() }],
        otherConfig: {
          isRepeat: false,
          interval: 1000
        }
      }
    },
    mode: 'use'
  }
)

const restDataList = ref<Array<SelectOption>>([])
const loadRestList = async () => {
  try {
    const resp = await getRestDataListApi()
    if (resp.status === 200) {
      restDataList.value = resp.data.map((el: RestDataDetail) => {
        return {
          label: el.name,
          value: el.id
        }
      })
    }
  } catch (err: any) {
    Logger.log(err || err.message)
  }
}

const clear = () => {
  formData.id = undefined
  formData.title = undefined
  formData.headers = [{ key: '', value: '', disable: false, id: uuid() }]
  formData.data = [{ key: '', value: '', disable: false, id: uuid() }]
  formData.params = [{ key: '', value: '', disable: false, id: uuid() }]
  formData.method = RequestMethod.GET
  formData.url = ''
}
interface ErrorResponse extends Error {
  config: Record<string, any>
  code?: number | undefined
  response: AxiosResponse
  isAxiosError: boolean

  toJSON?: () => {
    message: string
    name: string
    // Microsoft
    description?: string
    number?: string
    // Mozill
    fileName?: string
    lineNumber?: string
    columnNumber?: string
    stack?: string
    // Axios
    config: Record<string, any>
    code?: number
    status?: number
  }
}
let snapShot
if (props.mode === 'debug') {
  useEventBus(StaticKey.REST_KEY, async (id) => {
    await loadRestData(id as string)
    await send()
  })
}

const loadRestData = async (id: string) => {
  try {
    const resp = await getRestDataApi(id)
    if (resp.status === 200) {
      const data: RestDataDetail = resp.data
      formData.method = data.method
      formData.url = data.url
      if (!data.data) {
        return
      }
      const body = recordabletoKV(data.data || {})
      formData.data = body.length > 0 ? body : getEmptyParams()
      const params = recordabletoKV(data.params || {})
      formData.params = params.length > 0 ? params : getEmptyParams()
      formData.title = data.name
      formData.id = data.id!

      const headers = recordabletoKV(data.headers || {})
      formData.headers = headers.length > 0 ? headers : getEmptyParams()
    }
  } catch (err) {
    return undefined
  }
}
const emits = defineEmits<{
  (e: 'update:options', value: RestOption): void
  (e: 'change', value: RestOption): void
}>()

interface RequestDataOption extends RestOption {
  title?: string
  id?: string
}
const formData = reactive<RequestDataOption>(props.options)
const response = ref<RestResponse>({
  status: 0,
  data: '',
  headers: {}
})
const requestInstance = useRequest()
const send = async () => {
  try {
    const resp = await requestInstance.request(requestOptionsToStore(formData))
    response.value.status = resp.status
    response.value.data = JSON.stringify(resp.data, null, '\t')

    response.value.headers = resp.headers
    formData.id && snapShot && snapShot.save(formData)
  } catch (err: any) {
    err as ErrorResponse
    const result = err.response || (err.toJSON ? err.toJSON() : {})
    response.value.status = result.status
    response.value.data = err.stack || err.message
    response.value.headers = result.headers || result?.config?.headers || {}
  }
}
const formChange = () => {
  emits('change', formData)
  emits('update:options', formData)
}

const selectedChange = async (id: string) => {
  await loadRestData(id)
  await send()
  emits('update:options', formData)
  emits('change', formData)
}
const handleSave = async () => {
  try {
    const headers = KVToRecordable(formData.headers)
    const params = KVToRecordable(formData.headers)
    const data = KVToRecordable(formData.data)
    const resp = await createRestDataApi({
      data: data,
      name: formData.title || '未命名',
      url: formData.url,
      method: formData.method,
      headers: headers,
      params: params
    })
    if (resp.status === 201) {
      const data: RestDataDetail = resp.data
      formData.id = data.id!
      formData.title = data.name
      message.info('数据保存成功')
    } else {
      message.warning('数据保存失败')
    }
  } catch (err) {
    message.warning('数据保存失败')
  }
}
const handleUpdate = async () => {
  try {
    const headers = KVToRecordable(formData.headers)
    const params = KVToRecordable(formData.headers)
    const data = KVToRecordable(formData.data)
    const resp = await updateRestDataApi(formData.id!, {
      data: data,
      name: formData.title || '未命名',
      url: formData.url,
      method: formData.method,
      headers: headers,
      params: params
    })
    if (resp.status === 200) {
      message.info('数据更新成功')
    } else {
      message.warning('数据更新失败')
    }
  } catch (err) {
    message.warning('数据更新失败')
  }
}

onMounted(async () => {
  await send()
  await loadRestList()
})
</script>

<style scoped lang="less">
.rest-data {
  display: flex;
  margin-bottom: 5px;
  .selected {
    flex: 4;
  }
  .title {
    flex: 8;
  }
}
.api {
  display: flex;
  .method {
    min-width: 110px;
    flex: 8;
  }
  .url {
    flex: 90;
  }
}
.response {
  .resp-fail {
    color: #f76560;
    margin-left: 10px;
  }
  .resp-code.resp-success {
    color: #18a058;
    margin-left: 10px;
  }
}
</style>
