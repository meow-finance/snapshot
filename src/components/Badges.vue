<script setup>
import { computed } from 'vue';

const props = defineProps({ address: String, members: Object });

const isCore = computed(() => {
  if (!props.members) return false;
  const members = props.members.map(address => address.toLowerCase());
  return members.includes(props.address.toLowerCase());
});
const coreClass = computed(() => {
  if (isCore.value) return 'core';
  else return 'community';
});
const iconClass = computed(() => {
  if (isCore.value) return 'brand-hallmark';
  else return 'users-social';
});
</script>


<template>
  <div class="State" :class="coreClass">
    <IconFont :name=iconClass size="16" lineHeight="16"/>
    <span
      v-text="$t(isCore ? 'isCore' : 'isCommunity')"
    />
  </div>
</template>

<style scoped lang="scss">
  .core
  {
      width: 80px;
      color: var(--button-bg);
      border-color: var(--button-bg);
      border-width: 2px
  }
  .community
  {
      width: 120px;
      color: var(--button-bg);
      border-color: var(--button-bg);
      border-width: 2px
  }
</style>