<template>
  <o-form-item key="data" label="示例数据">
    <div class="justify-center flex-row flex-nowrap flex items-center">
      <o-input placeholder="点击预览" :readonly="true" @click="isShow = true" />
      <o-button type="primary" @click="isShow = true"> 预览 </o-button>
    </div>
  </o-form-item>
  <o-modal v-model:show="isShow">
    <o-card
      title="示例数据"
      :bordered="false"
      size="small"
      role="dialog"
      closable
      aria-modal="true"
      @close="isShow = false"
    >
      <OCodeEditor v-model:value="formData.data" />
    </o-card>
  </o-modal>
</template>

<script lang="ts" setup>
import { cloneDeep } from 'lodash-es'
import type { Slotter } from 'open-data-v/data/type'
import { OButton, OCard, OFormItem, OInput, OModal } from 'open-data-v/ui'
import { onMounted, reactive, ref, watch } from 'vue'

import DataHandler from './handler'

interface CutomeSlotter extends Slotter {
  exampleData: any
  propValue: any
}

const props = defineProps<{
  slotter: CutomeSlotter
}>()
const isShow = ref<boolean>(false)

const formData = reactive<{
  data: string
}>({
  data: ''
})

onMounted(async () => {
  await initData()
})

const initData = async () => {
  const dataConfig = props.slotter.dataConfig

  if (dataConfig && dataConfig.type === 'DEMO') {
    const demoRequest = props.slotter.dataConfig?.dataInstance as DataHandler
    if (demoRequest) {
      const resp = await demoRequest.getRespData({ propValue: props.slotter.propValue })
      formData.data = JSON.stringify(resp.data, null, '\t')
    }
  } else {
    const exampleData = props.slotter.exampleData
    const dataConfig = {
      type: 'DEMO',
      dataInstance: new DataHandler({
        data: cloneDeep(exampleData)
      })
    }
    await props.slotter.changeDataConfig(dataConfig)
  }
}

watch(
  () => props.slotter,
  (value) => {
    if (value) {
      initData()
    }
  },
  { immediate: true }
)
</script>

<style lang="less" scoped></style>
