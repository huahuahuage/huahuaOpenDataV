<template>
  <div>
    <!-- 选中组件时的右侧面板 -->
    <o-tabs
      v-if="curComponent"
      v-model:value="activeKey"
      justify-content="flex-start"
      @update:value="
        (key) => {
          activeKey = key
        }
      "
    >
      <o-tab-pane name="style">
        <template #label>
          <div class="flex-nowrap flex flex-row items-center">
            <span class="inline-block">样式</span>
          </div>
        </template>
        <div class="attr-list o-scroll">
          <StyleList :curComponent="curComponent" />
        </div>
      </o-tab-pane>
      <o-tab-pane name="attr">
        <template #label>
          <div class="flex-nowrap flex flex-row items-center">
            <span class="inline-block">属性</span>
          </div>
        </template>
        <div class="attr-list o-scroll">
          <AttrList :curComponent="curComponent" />
        </div>
      </o-tab-pane>
    </o-tabs>
    <!-- 未选中 -->
    <o-tabs
      v-else
      v-model:value="canvasActiveKey"
      justify-content="flex-start"
      @update:value="
        (key) => {
          canvasActiveKey = key
        }
      "
    >
      <o-tab-pane name="canvas">
        <template #label>
          <div class="flex-nowrap flex flex-row items-center">
            <span>画布</span>
          </div>
        </template>
        <div class="attr-list">
          <Canvas />
        </div>
      </o-tab-pane>
    </o-tabs>
  </div>
</template>

<script setup lang="ts">
import { useCanvasState } from 'open-data-v/designer'
import { OTabPane, OTabs } from 'open-data-v/ui'
import { computed, ref } from 'vue'

import AttrList from './AttrModule'
import Canvas from './CanvasModules/Canvas.vue'
// import GlobalData from './CanvasModules/GlobalData'
// import DataList from './DataModule'
import StyleList from './StyleModule'
// 右侧属性列表
// const instance = getCurrentInstance()
// const XIcon = instance!.appContext.components['XIcon']

const activeKey = ref<string>('attr')

const canvasState = useCanvasState()

const curComponent = computed(() => canvasState.curComponent)

const canvasActiveKey = ref<string>('canvas')

// const collapsedTabPane = (key: string) => {
//   emits('update:iscollapsed', true)
//   activeKey.value = key
// }
</script>
<style scoped>
@import 'open-data-v/css/index.less';
.attr-list {
  @apply overflow-auto p-1 pt-0;
  backdrop-filter: blur(50px);
  height: calc(90vh - 28px);
  padding: 10px;
}
</style>
