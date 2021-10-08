<script setup>
defineProps({
  name: String,
  size: String,
  lineHeight: String
});
</script>

<template>
  <i
    class="iconfont"
    :class="`icofont-${name}`"
    :style="size ? `font-size: ${size}px; line-height: ${lineHeight == undefined ? size : lineHeight }px;` : ''"
  />
</template>

<style scoped lang="scss">
.iconfont {
  font-size: 16px;
  line-height: 16px;
  vertical-align: baseline;
  padding-right: 4px;
}
</style>
