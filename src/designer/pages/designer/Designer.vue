<template>
  <div class="main h-fit flex-col flex flex-nowrap">
    <div class="top w-full h-14 align-middle">
      <ToolBar :toolbars="toolbars" />
    </div>
    <div class="content flex flex-nowrap flex-row">
      <div class="left o-scroll">
        <LeftSideBar />
      </div>
      <Canvas class="canvas border border-gray-500" />
      <div class="right o-scroll">
        <RightSideBar :iscollapsed="true" />
      </div>
    </div>
  </div>
</template>
<script setup lang="ts">
import { useData, useProp } from 'open-data-v/base'
import type { ToolBarItemType } from 'open-data-v/designer'
import { useCanvasState } from 'open-data-v/designer'
import { onUnmounted, provide, readonly } from 'vue'

import type { LayoutData } from '../../../designer/state/type'
import LeftSideBar from '../../Pane/LeftSideBar'
import RightSideBar from '../../Pane/RightSideBar'
import ToolBar from '../../Pane/Toolsbar'
import Canvas from './Canvas.vue'

withDefaults(
  defineProps<{
    toolbars?: ToolBarItemType[]
  }>(),
  {
    toolbars: () => []
  }
)
const canvasState = useCanvasState()

provide('HOOKS', readonly({ useProp, useData }))
const setLayoutData = (data: LayoutData) => {
  canvasState.setLayoutData(data)
}
defineExpose({ setLayoutData })

onUnmounted(() => {
  canvasState.clearCanvas()
})
</script>
<style scoped lang="less">
@import 'open-data-v/css/index.less';
.main {
  .top {
    padding: 0 10px;
    border-bottom: 0.5px solid #e5e7eb;
  }
  .content {
    width: 100vw;
    height: calc(100vh - 56px);
    .canvas {
      transition-property: width;
      transition-duration: 0.5s;
      width: calc(100vw - 540px);
      height: calc(100vh - 103px);
    }
    .left {
      transition-property: width;
      transition-duration: 0.5s;
      width: 240px;
      padding-top: 10px;
      // border: 1px solid #e5e7eb;
      overflow: hidden;
    }
    .right {
      transition-property: width;
      transition-duration: 0.5s;
      width: 300px;
      height: 100%;
      padding-top: 10px;
      // border: 1px solid #e5e7eb;
      overflow: hidden;
    }
  }
}
</style>
