<script setup>
import { isAddress } from '@ethersproject/address';

defineProps({ open: Boolean, strategies: Object, space: Object });

defineEmits(['close']);
</script>

<template>
  <UiModal :open="open" @close="$emit('close')">
    <template v-slot:header>
      <h3>{{ $t('strategiesPage') }}</h3>
    </template>
    <div class="m-4">
      <div
        v-for="(strategy, i) in strategies"
        :key="i"
        class="p-4 mb-3 border rounded-md link-color"
      >
        <h3 v-text="strategy.name" />
        <div>
          <div v-for="(option, key) in strategy.params" :key="key" class="flex">
            <span v-text="key" class="flex-auto text-color mr-1" />
            <a
              v-if="key === 'address' || isAddress(option)"
              :href="_explorer(space.network, option)"
              target="_blank"
              class="block"
            >
              <span v-text="_shorten(option)" />
              <Icon name="external-link" class="ml-1" />
            </a>
            <span
              v-else
              v-text="
                ['string', 'number', 'boolean'].includes(typeof option)
                  ? option
                  : typeof option
              "
            />
          </div>
        </div>
      </div>
    </div>
  </UiModal>
</template>
