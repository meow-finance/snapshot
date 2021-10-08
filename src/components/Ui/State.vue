<script setup>
import { computed } from 'vue';

const props = defineProps({
  state: String
});

const stateClass = computed(() => {
  if (props.state === 'closed') return 'bg-gray-500';
  if (props.state === 'active') return 'votenow ';
  return 'votenow';
});
const iconClass = computed(() => {
  if (props.state === 'closed') return 'ban';
  if (props.state === 'active') return 'law-order';
  return '';
});
</script>

<template>
  <div class="State text-white" :class="stateClass">
    <IconFont :name=iconClass size="16" lineHeight="16"/>
    <span
      v-text="$t(`proposals.states.${state}`)"
    />
  </div>
</template>

<style lang="scss">
  .State{
    width: 90px;
  }
  .votenow
  {
    width: 110px;
    background-color: var(--button-bg);
  }
</style>