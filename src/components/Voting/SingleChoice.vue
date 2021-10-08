<script setup>
import { ref } from 'vue';

defineProps({
  proposal: {
    type: Object,
    required: true
  }
});

const emit = defineEmits(['selectChoice']);

const selectedChoice = ref(null);

function selectChoice(i) {
  selectedChoice.value = i;
  emit('selectChoice', i);
}
</script>

<template>
  <div class="mb-3">
    <div  @click="selectChoice(i + 1)" v-for="(choice, i) in proposal.choices"   :key="i" class="flex">
      <UiMeowRadio :active="selectedChoice === i + 1" width="30" height="30"></UiMeowRadio>
      <UiButton
        class="block w-full mb-2"
        :class="selectedChoice === i + 1 && 'button--active'"
      >
        {{ _shorten(choice, 32) }}
        <PluginAragonGovern :proposal="proposal" />
      </UiButton>
    </div>
  </div>
</template>
