<script setup>

import {computed, ref } from 'vue';
import meowX from '@/assets/images/meowX.svg'
import meowO from '@/assets/images/meowO.svg'
const props = defineProps({
  modelValue: String,
  value: String,
  text: String,
  width: String,
  height: String,
  active: Boolean
});

const emit = defineEmits(['update:modelValue']);

const input = ref(false);

function handleToggle() {
  input.value = props.value;
  emit('update:modelValue', props.value);
}


</script>

<template>
<div  @click="handleToggle" class="pointer">
    <UiButtonIcon class="buttonicon"
        :imgsrc="(props.modelValue == props.value && value != undefined ) || active ? meowX : meowO " 
        :width="props.width"
        :heigth="props.height"
    />
    <span>{{props.text}}</span>
</div>

</template>


<style scoped lang="scss">
    .pointer
    {
        cursor: pointer;
    }
    .buttonicon
    {
        padding-right: 4px;
    }
</style>