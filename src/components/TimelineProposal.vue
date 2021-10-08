<script setup>
import { watchEffect, computed } from 'vue';
import { useUsername } from '@/composables/useUsername';
import removeMd from 'remove-markdown';

const props = defineProps({
  proposal: Object,
  profiles: Object
});

const body = computed(() => removeMd(props.proposal.body));

const period = computed(() => {
  if (props.proposal.state === 'closed') return 'endedAgo';
  if (props.proposal.state === 'active') return 'endIn';
  return 'startIn';
});

const _date = (_mstime)=>
{ 
    const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' , hour: '2-digit', minute: '2-digit'};
    const d = (new Date(_mstime*1000)).toLocaleString(undefined, options);
    return `${d}`;
}

const { address, profile, username } = useUsername();

watchEffect(() => {
  address.value = props.proposal.author;
  profile.value = props.profiles[props.proposal.author];
});
</script>

<template>
  <router-link
    class="p-3 block text-color"
    :to="{
      name: 'spaceProposal',
      params: { key: proposal.space.id, id: proposal.id }
    }"
  >
    <div>
      <!-- <div class="mb-2">
        <Token :space="proposal.space" size="28" />
        <span class="ml-2" v-text="proposal.space.name" />
        {{ $tc('proposalBy', [username]) }}
        <Badges :address="proposal.author" :members="proposal.space.members" />
        <UiState :state="proposal.state" class="inline-block float-right" />
      </div> -->
      <h3 v-text="_shorten(proposal.title, 124)" class="mt-1 mb-1" />
      <!-- <p v-text="_shorten(body, 140)" class="break-words mb-1 text-md" /> -->
      <div>
        {{ $tc(period, [_date(proposal.start), _date(proposal.end)]) }} 
      </div>
      <div style="display: flex;flex-direction: row; align-items: center;align-content: center;">
        <UiState  style="margin-right: 9px;text-align: center;" :state="proposal.state" />
        <Badges  style="ext-align: center;" :address="proposal.author" :members="proposal.space.members" />
      </div> 
    </div>
  </router-link>
</template>

