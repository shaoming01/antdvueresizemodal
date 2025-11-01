# and vue3 可拖动、最大化、调整位置的独立组件（无入侵）

## 用法如下：
```html
<template>
  <AModal :open="true"
          :title="'标题'"
          :bodyStyle="{height: modalStyle.bodyHeight}"
          :style="modalStyle"
          wrap-class-name="resizeModal"
  >
    这是测试内容
    <ModalReSizeHandle v-model:style="modalStyle"></ModalReSizeHandle>
  </AModal>
</template>

<script lang="ts" setup>
const modalStyle = ref({
  width: '300px',
  bodyHeight: '200px',//
  paddingBottom: 0,
  maxWidth: '100%'
});

</script>

<style scoped lang="scss">

</style>


```