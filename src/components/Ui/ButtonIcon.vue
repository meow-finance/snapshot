<script setup>
import { ref } from 'vue';
defineProps({
  address: String,
  width: String,
  heigth: String,
  imgsrc: String,
  space: Object,
});

const error = ref(false);

</script>

<template>
  <span class="flex-shrink-0">
    <img
      v-if="imgsrc && !error"
      :src="imgsrc"
      :style="{
        width: `${parseInt(width) || 22}px`,
        height: `${parseInt(heigth) || 22}px`,
        'padding-right': '3px'
      }"
      @error="error = true"
      :class="[
        space?.skin ? `${space?.skin} bg-[color:var(--bg-color)]` : ''
      ]"
      class="inline-block !align-middle leading-none"
    />
    <UiBlockie
      v-else
      :seed="address"
      class="inline-block !align-middle"
      :style="{
        'line-height': 0,
        width: `${parseInt(width) || 22}px`,
        height: `${parseInt(heigth) || 22}px`,
        'padding-right': '3px'
      }"
    />
  </span>
</template>
