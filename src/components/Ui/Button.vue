<script setup>
defineProps({
  loading: Boolean,
  type: String,
  disable: {
    type: Boolean,
    default: true
  }
});
</script>

<template>
  <button
    :type="type || 'button'"
    class="button px-[24px] focus-within:border-skin-link"
    :disabled="disable ? loading : false"
  >
    <UiLoading v-if="loading" />
    <slot v-else />
  </button>
</template>

<style scoped lang="scss">
.button {
  border: 1px solid var(--border-color);
  background-color: var(--button-bg);
  color: var(--button-text);
  border-radius: 23px;
  outline: none;
  line-height: 24px;
  height: 35px;
  font-size: 18px;

  &.button--submit {
    color: white;
    background-color: var(--primary-color);
    border: 1px solid var(--primary-color);

    &:hover {
      color: white;
      background-color: var(--primary-color);
      border: 1px solid var(--primary-color);
    }

    &:disabled {
      color: var(--button-text) !important;
      border: 1px solid var(--border-color);
      background-color: var(--border-color);
    }
  }

  &.button--active {
    border-color: var(--border-color) !important;
    background-color: #986be9;

    &:hover {
      color: white;
      background-color: #986be9;
      border: 1px solid var(--primary-color);
    }
  }

  &:hover {
    color: var(--button-text);
    border-color: var(--button-text);
    background-color: var(--button-hover);
  }

  &:disabled {
    color: var(--border-color) !important;
    cursor: not-allowed;
  }
}
</style>
