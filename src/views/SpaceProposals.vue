<script setup>
import { computed, onMounted, ref, watch } from 'vue';
import { useInfiniteLoader } from '@/composables/useInfiniteLoader';
import { useScrollMonitor } from '@/composables/useScrollMonitor';
import { useApolloQuery } from '@/composables/useApolloQuery';
import { PROPOSALS_QUERY } from '@/helpers/queries';
import { useProfiles } from '@/composables/useProfiles';
import { useUnseenProposals } from '@/composables/useUnseenProposals';
import { lsSet } from '@/helpers/utils';
import { useWeb3 } from '@/composables/useWeb3';

const props = defineProps({ space: Object, spaceId: String });

const { lastSeenProposals, updateLastSeenProposal } = useUnseenProposals();
const { web3 } = useWeb3();

const loading = ref(false);
const proposalsInit = ref([]);
const proposals = ref([]);
//active , closed , pending , all
const filterBy = ref('active');
//core , community , all
const createBy = ref('core');

const spaceMembers = computed(() =>
  props.space.members.length < 1 ? ['none'] : props.space.members
);
const web3Account = computed(() => web3.value.account);

const { loadBy, limit, loadingMore, stopLoadingMore, loadMore } =
  useInfiniteLoader();

const { endElement } = useScrollMonitor(() =>
  loadMore(() => loadProposals(limit.value), loading.value)
);

const { apolloQuery } = useApolloQuery();
async function loadProposals(skip = 0) {
  const proposalsObj = await apolloQuery(
    {
      query: PROPOSALS_QUERY,
      variables: {
        first: loadBy,
        skip,
        space: props.spaceId,
        state: filterBy.value ,
        author_in: []
      }
    },
    'proposals'
  );
  stopLoadingMore.value = proposalsObj?.length < loadBy;
  proposalsInit.value = proposalsInit.value.concat(proposalsObj);
  //wait for props.space.members
  let delayms = 0;
  if (props.space.members == undefined) {
    delayms = 1000;
  };
  await setTimeout(async () => {
    await filter();
  }, delayms);
  
}

onMounted(load());

async function load() {
  loading.value = true;
  await loadProposals();
  loading.value = false;
}

async function filter()
{ 
  proposals.value = proposalsInit.value.filter((proposal,inx) => 
  {
    if (createBy.value == "core")
    {
        return spaceMembers.value.includes(proposal.author) ? filterBy.value == proposal.state ? true : false : false;
    }
    else if (createBy.value == "community")
    {
      return !spaceMembers.value.includes(proposal.author) ? filterBy.value == proposal.state ? true : false  : false;
    }
    else
    {
      return filterBy.value == proposal.state ? true : false 
    }
  });
}

function selectState(e) {
  if ("core,community,all".indexOf(e) >= 0)
  {
    createBy.value = e;
    filter();
  }
  else
  {
    filterBy.value = e;
    proposalsInit.value = [];
    limit.value = loadBy;
    load();
  }
}

const { profiles, addressArray } = useProfiles();

watch(proposals, () => {
  if (proposals.value != null) addressArray.value = proposals.value.map(proposal => proposal.author);
});

watch(filterBy, () => {
  selectState(filterBy.value);
});

watch([proposals, web3Account], () => {
  if (web3Account.value) {
    lsSet(
      `lastSeenProposals.${web3Account.value.slice(0, 8).toLowerCase()}`,
      Object.assign(lastSeenProposals.value, {
        [props.spaceId]: new Date().getTime()
      })
    );
  }
  updateLastSeenProposal(web3Account.value);
});
</script>

<template>
  <Layout>
    <!-- <template #sidebar-left>
      <BlockSpace :space="space" />
    </template> -->
    <template #content-center>
      <div class="px-4 md:px-0 mb-3 flex">
        <div class="flex-auto">
          <!-- <div v-text="space.name" /> -->
          <div class="flex items-center flex-auto">
            <h1>{{ $t('proposals.header') }}</h1>
          </div>
        </div>
        <!-- <UiDropdown
          top="3.5rem"
          right="1.25rem"
          @select="selectState"
          :items="[
            { text: $t('proposals.states.all'), action: 'all' },
            { text: $t('proposals.states.active'), action: 'active' },
            { text: $t('proposals.states.pending'), action: 'pending' },
            { text: $t('proposals.states.closed'), action: 'closed' },
            { text: $t('proposals.states.core'), action: 'core' }
          ]"
        >
          <UiButton class="pr-3">
            {{ $t(`proposals.states.${filterBy}`) }}
            <Icon size="14" name="arrow-down" class="mt-1 mr-1" />
          </UiButton>
        </UiDropdown> -->
      </div>

      <div class="content-box">
        <div class="content-box-head">
          <div class="header-menu" @click="selectState('core')" :class="createBy == 'core' ? 'menu-active' : ''" >
            <IconFont name="brand-hallmark" size="22" lineHeight="13"/>
              <span>Core</span>
          </div>
           <div class="header-menu" @click="selectState('community')" :class="createBy == 'community' ? 'menu-active' : ''"  style="width:180px" >
              <IconFont name="users-social" size="22" lineHeight="13"/>
              <span>Community</span>
          </div>
           <div class="header-menu" @click="selectState('all')" :class="createBy == 'all' ? 'menu-active' : ''"  >
              <span>All</span>
          </div>
        </div>
        <div class="content-box-detail">
          <div class="content-filter">
            <UiMeowRadio class="meow-filter" height="25" width="25" v-model="filterBy" value="active" :text="$t(`proposals.states.active`)" />
            <UiMeowRadio class="meow-filter" height="25" width="25" v-model="filterBy" value="pending" :text="$t(`proposals.states.pending`)" />
            <UiMeowRadio class="meow-filter" height="25" width="25" v-model="filterBy" value="closed" :text="$t(`proposals.states.closed`)" />
          </div>
          <div>
              <BlockBottom v-if="loading" :slim="true">
                <RowLoading class="my-2" />
              </BlockBottom>
              <NoResults :block="true" v-else-if="proposals.length < 1" />
              <div v-else > 
                <BlockBottom :slim="true" v-for="(proposal, i) in proposals" :key="i" class="block">
                  <TimelineProposal :proposal="proposal" :profiles="profiles" />
                </BlockBottom>
              </div>
              <div
                style="height: 10px; width: 10px; position: absolute"
                ref="endElement"
              />
              <BlockBottom v-if="loadingMore && !loading" :slim="true">
                <RowLoading class="my-2" />
              </BlockBottom>
          </div>

        </div>
      </div>

    </template>
  </Layout>
</template>

<style scoped lang="scss">
.block:hover
{
  background-color: var(--focus);
  border-top-left-radius: 15px;
  border-top-right-radius: 15px;
}
.mb-4
{
  margin-bottom: 5px !important;
}
.meow-filter
{
  padding-right: 15px;
}
.content-filter
{
    display: flex;
    flex-direction: row;
    align-items: center;
    margin-bottom: 11px;
    border-bottom-width: 2px;
    padding-bottom: 6px;
}
 
 .menu-active
 {
    background: #FFFFFF;
    color: var(--text-head-color); 
 }

</style>
