<template>
  <o-tabs
    v-model:value="activeKey"
    justify-content="flex-start"
    @update:value="
      (key) => {
        activeKey = key
      }
    "
  >
    <o-tab-pane name="components" display-directive="show:lazy">
      <template #label>
        <div class="flex-nowrap flex flex-row items-center">
          <!-- <x-icon name="components" class="inline-block" /> -->
          <span class="inline-block">组件</span>
        </div>
      </template>
      <ComponentList class="components" />
    </o-tab-pane>
    <o-tab-pane name="layer" display-directive="show:lazy">
      <template #label>
        <div class="flex-nowrap flex flex-row items-center">
          <!-- <x-icon name="layers" class="inline-block" /> -->
          <span class="inline-block">图层</span>
        </div>
      </template>
      <Layer class="layers" />
    </o-tab-pane>
  </o-tabs>
</template>

<script lang="ts" setup>
import { OTabPane, OTabs } from 'open-data-v/ui'
import { ref } from 'vue'

import ComponentList from './Component/ComponentList.vue'
import Layer from './Layer/Layer.vue'

const activeKey = ref<string>('components')
</script>
<style scoped lang="less">
.components {
  height: calc(90vh - 28px);
}
.layers {
  height: calc(90vh - 28px);
}
</style>
