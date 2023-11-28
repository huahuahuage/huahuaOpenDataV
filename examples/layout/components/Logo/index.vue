<template>
  <div class="logo">
    <img :src="Logo" alt="" class="mr-2" />
    <div>
      <h2 class="title">{{ title }}</h2>
      <div class="sub-title">Base on OpenDataV</div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue'

import Logo from '@/assets/logo.png'

const title = ref<string>(import.meta.env.VITE_APP_TITLE)
</script>

<style lang="less" scoped>
.logo {
  @apply flex items-center justify-center overflow-hidden whitespace-nowrap;
  height: @header-height;
  line-height: @header-height;

  img {
    @apply w-auto;
    height: 35px;
  }

  .title {
    @apply mb-0;
    height: 20px;
    margin: 0;
    padding: 0;
    font-size: 16px;
  }

  .sub-title {
    font-size: 12px;
  }
}
</style>
