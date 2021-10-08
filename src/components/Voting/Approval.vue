<script setup>
import { ref } from 'vue';

defineProps({
  proposal: {
    type: Object,
    required: true
  }
});

const emit = defineEmits(['selectChoice']);

const selectedChoices = ref([]);

function selectChoice(i) {
  if (selectedChoices.value.includes(i))
    selectedChoices.value.splice(selectedChoices.value.indexOf(i), 1);
  else selectedChoices.value.push(i);

  emit('selectChoice', selectedChoices.value);
}
</script>

<template>
  <div class="mb-3">
    <div  @click="selectChoice(i + 1)" v-for="(choice, i) in proposal.choices"   :key="i" class="flex">
        <UiMeowRadio :active="selectedChoices.includes(i + 1)" width="30" height="30"></UiMeowRadio>
        <UiButton
          class="block w-full mb-2"
          :class="selectedChoices.includes(i + 1) && 'button--active'"
        >
          {{ _shorten(choice, 32) }}
          <PluginAragonGovern :proposal="proposal" />
        </UiButton>
    </div>
  </div>
</template>

