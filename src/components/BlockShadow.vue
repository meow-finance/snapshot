<script setup>
defineProps({
  title: String,
  counter: Number,
  slim: Boolean,
  icon: String,
  loading: Boolean
});

defineEmits(['submit']);
</script>

<template>
   <div class="content-box">
        <div class="content-box-head">
            <div class="header-menu" >
              <span>{{title}}</span>
          </div>
        </div>
        <div class="content-box-detail">
              
          <div v-if="loading" class="block px-4 py-4">
            <div
              class="bg-skin-text rounded-md animate-pulse-fast mb-2"
              style="width: 80%; height: 20px"
            />
            <div
              class="bg-skin-text rounded-md animate-pulse-fast"
              style="width: 50%; height: 20px"
            />
          </div>
          <div v-else :class="!slim && 'p-4'" class="leading-6">
            <slot />
          </div>

        </div>
      </div>
</template>

<style scoped lang="scss">
  .content-box-head
  {
    height: 50px;
  }
   .header-menu
  { 
      &:hover {
        cursor: default;
      }
  }
  .header-menu
  {
      width: 100%;
      justify-content: flex-start;
  }
</style>