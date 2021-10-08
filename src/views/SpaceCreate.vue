<script setup>
import { ref, watchEffect, computed, onMounted } from 'vue';
import { useRouter } from 'vue-router';
import draggable from 'vuedraggable';
import getProvider from '@snapshot-labs/snapshot.js/src/utils/provider';
import { getBlockNumber } from '@snapshot-labs/snapshot.js/src/utils/web3';
import { getInstance } from '@snapshot-labs/lock/plugins/vue3';
import { useModal } from '@/composables/useModal';
import { useTerms } from '@/composables/useTerms';
import { PROPOSAL_QUERY } from '@/helpers/queries';
import validations from '@snapshot-labs/snapshot.js/src/validations';
import { clone } from '@/helpers/utils';
import { useDomain } from '@/composables/useDomain';
import { useApolloQuery } from '@/composables/useApolloQuery';
import { useWeb3 } from '@/composables/useWeb3';
import { useClient } from '@/composables/useClient';

const props = defineProps({
  spaceId: String,
  space: Object,
  from: String
});

const router = useRouter();
const auth = getInstance();
const { domain } = useDomain();
const { web3 } = useWeb3();
const { send } = useClient();

const loading = ref(false);
const choices = ref([]);
const blockNumber = ref(-1);
const bodyLimit = ref(6400);
const form = ref({
  name: '',
  body: '',
  choices: [],
  start: 0,
  end: 0,
  snapshot: '',
  metadata: { plugins: {} },
  type: 'single-choice'
});
const modalOpen = ref(false);
const modalProposalPluginsOpen = ref(false);
const modalVotingTypeOpen = ref(false);
const selectedDate = ref('');
const counter = ref(0);
const nameForm = ref(null);
const passValidation = ref([true]);
const loadingSnapshot = ref(true);

const web3Account = computed(() => web3.value.account);
const proposal = computed(() =>
  Object.assign(form.value, { choices: choices.value })
);

// Check if account passes space validation
watchEffect(async () => {
  if (web3Account.value && auth.isAuthenticated.value) {
    const validationName = props.space.validation?.name ?? 'basic';
    const validationParams = props.space.validation?.params ?? {};
    const isValid = await validations[validationName](
      web3Account.value,
      clone(props.space),
      '',
      clone(validationParams)
    );
    passValidation.value = [isValid, validationName];
    console.log('Pass validation?', isValid, validationName);
  }
});

const isValid = computed(() => {
  // const ts = (Date.now() / 1e3).toFixed();
  const isSafeSnapPluginValid = form.value.metadata.plugins?.safeSnap
    ? form.value.metadata.plugins.safeSnap.valid
    : true;

  return (
    !loading.value &&
    form.value.name &&
    form.value.body.length <= bodyLimit.value &&
    form.value.start &&
    // form.value.start >= ts &&
    form.value.end &&
    form.value.end > form.value.start &&
    form.value.snapshot &&
    form.value.snapshot > blockNumber.value / 2 &&
    choices.value.length >= 2 &&
    !choices.value.some(a => a.text === '') &&
    passValidation.value[0] &&
    isSafeSnapPluginValid &&
    !web3.value.authLoading
  );
});

function addChoice(num) {
  for (let i = 1; i <= num; i++) {
    counter.value++;
    choices.value.push({ key: counter.value, text: '' });
  }
}

function removeChoice(i) {
  choices.value.splice(i, 1);
}

function setDate(ts) {
  if (selectedDate.value) {
    form.value[selectedDate.value] = ts;
  }
}

async function handleSubmit() {
  loading.value = true;
  form.value.snapshot = parseInt(form.value.snapshot);
  form.value.choices = choices.value.map(choice => choice.text);
  form.value.metadata.network = props.space.network;
  form.value.metadata.strategies = props.space.strategies;
  try {
    const { ipfsHash } = await send(props.space.id, 'proposal', form.value);
    router.push({
      name: 'spaceProposal',
      params: {
        key: props.spaceId,
        id: ipfsHash
      }
    });
  } catch (e) {
    console.error(e);
    loading.value = false;
  }
}

const { modalAccountOpen } = useModal();
const { modalTermsOpen, termsAccepted, acceptTerms } = useTerms(props.spaceId);

function clickSubmit() {
  !web3Account.value
    ? (modalAccountOpen.value = true)
    : !termsAccepted.value && props.space.terms
    ? (modalTermsOpen.value = true)
    : handleSubmit();
}

const { apolloQuery, queryLoading } = useApolloQuery();

async function loadProposal() {
  const proposal = await apolloQuery(
    {
      query: PROPOSAL_QUERY,
      variables: {
        id: props.from
      }
    },
    'proposal'
  );

  form.value = {
    name: proposal.title,
    body: proposal.body,
    choices: proposal.choices,
    start: proposal.start,
    end: proposal.end,
    snapshot: proposal.snapshot,
    type: proposal.type
  };

  const { network, strategies, plugins } = proposal;
  form.value.metadata = { network, strategies, plugins };

  choices.value = proposal.choices.map((text, key) => ({
    key,
    text
  }));
}

onMounted(async () => {
  // nameForm.value.focus();
  addChoice(2);

  if (props.from) loadProposal();
});

watchEffect(async () => {
  loadingSnapshot.value = true;
  if (props.space.network) {
    blockNumber.value = await getBlockNumber(getProvider(props.space.network));
    form.value.snapshot = blockNumber.value;
    loadingSnapshot.value = false;
  }
});
</script>

<template>
  <Layout v-bind="$attrs" style="padding-top: 1rem;">
    <template #content-left>
      <!-- <div class="px-4 md:px-0 mb-3">
        <router-link
          :to="domain ? { path: '/' } : { name: 'spaceProposals' }"
          class="text-color"
        >
          <Icon name="back" size="22" class="!align-middle" />
          {{ space.name }}
        </router-link>
      </div> -->

      <!-- Hint validation -->
      <Block v-if="passValidation[0] === false">
        <Icon name="warning" class="mr-1" />
        <span v-if="passValidation[1] === 'basic'">
          {{
            space.validation?.params.minScore || space?.filters.minScore
              ? $tc('create.validationWarning.basic.minScore', [
                  _n(space.filters.minScore),
                  space.symbol
                ])
              : $t('create.validationWarning.basic.member')
          }}
        </span>
        <span v-else>
          {{
            $t(
              space.validation.params.rules ||
                'create.validationWarning.customValidation'
            )
          }}
        </span>
      </Block>


      <!-- Input -->
      <div class="px-4 md:px-0">
        <div class="flex flex-col mb-6">
          <h4>Title</h4>
          <UiInputBorder
            v-model="form.name"
            maxlength="128"
            class="text-2xl font-bold mb-2 input"
            additionalClass="input-border"
          />
          <h4>Content</h4>
          <TextareaAutosize
            v-model="form.body"
            class="input pt-1"
            additionalClass="input-border p-12"
          />
          <div class="mb-6">
            <p v-if="form.body.length > bodyLimit" class="!text-red mt-4">
              -{{ _n(-(bodyLimit - form.body.length)) }}
            </p>
          </div>
          <!-- Preview -->
          <div v-if="form.body">
             <BlockShadow :title="$t('create.preview')">
                <UiMarkdown :body="form.body" />
             </BlockShadow>
          </div>
        </div>
      </div>

      <!-- Choices -->
      <BlockShadow :title="$t('create.choices')">
        <div v-if="choices.length > 0" class="overflow-hidden mb-2">
          <draggable
            v-model="choices"
            :component-data="{ name: 'list' }"
            item-key="id"
          >
            <template #item="{ element, index }">
              <UiInput
                border="false"
                v-model="element.text"
                maxlength="32"
                additionalClass="input-border"
                >
                <template v-slot:label>
                  <span class="text-skin-link">{{ index + 1 }}</span>
                </template>
                <template v-slot:info>
                  <span @click="removeChoice(index)" class="icon-delete">
                    <IconFont name="close" size="22" />
                  </span>
                </template>
              </UiInput>
            </template>
          </draggable>
        </div>
        <div class="flex-center">
          <UiButton id="add" @click="addChoice(1)" class="h-40">
          {{ $t('create.addChoice') }}
          </UiButton>
        </div>
      </BlockShadow>
      <PluginSafeSnapConfig
        v-if="space?.plugins?.safeSnap"
        :create="true"
        :proposal="proposal"
        :moduleAddress="space.plugins?.safeSnap?.address"
        :network="space.network"
        v-model="form.metadata.plugins.safeSnap"
      />
    </template>


    <!-- Action -->
    <template #sidebar-right>
      <BlockShadow
        :title="$t('actions')"
        :icon="
          space.plugins && Object.keys(space.plugins).length > 0
            ? 'stars'
            : undefined
        "
        @submit="modalProposalPluginsOpen = true"
      >
        <div class="mb-2">
          <UiButton class="w-full mb-2" @click="modalVotingTypeOpen = true">
            <span>{{ $t(`voting.${form.type}`) }}</span>
          </UiButton>
          <UiButton
            @click="(modalOpen = true), (selectedDate = 'start')"
            class="w-full mb-2"
          >
            <span v-if="!form.start">{{ $t('create.startDate') }}</span>
            <span v-else v-text="$d(form.start * 1e3, 'short', 'en-US')" />
          </UiButton>
          <UiButton
            @click="(modalOpen = true), (selectedDate = 'end')"
            class="w-full mb-2"
          >
            <span v-if="!form.end">{{ $t('create.endDate') }}</span>
            <span v-else v-text="$d(form.end * 1e3, 'short', 'en-US')" />
          </UiButton>
          <UiButton :loading="loadingSnapshot" class="w-full mb-2">
            <input
              v-model="form.snapshot"
              type="number"
              class="input w-full text-center"
              :placeholder="$t('create.snapshotBlock')"
              style="color: #ffffff;"
            />
          </UiButton>
        </div>
        <UiButton
          @click="clickSubmit"
          :disabled="!isValid"
          :loading="loading || queryLoading"
          class="w-full button--submit"
        >
          {{ $t('create.publish') }}
        </UiButton>
      </BlockShadow>
    </template>


  </Layout>
  <teleport to="#modal">
    <ModalSelectDate
      :value="form[selectedDate]"
      :selectedDate="selectedDate"
      :open="modalOpen"
      @close="modalOpen = false"
      @input="setDate"
    />
    <ModalProposalPlugins
      :space="space"
      :proposal="proposal"
      v-model="form.metadata.plugins"
      :open="modalProposalPluginsOpen"
      @close="modalProposalPluginsOpen = false"
    />
    <ModalTerms
      :open="modalTermsOpen"
      :space="space"
      @close="modalTermsOpen = false"
      @accept="acceptTerms(), handleSubmit()"
    />

    <ModalVotingType
      :open="modalVotingTypeOpen"
      @close="modalVotingTypeOpen = false"
      v-model="form.type"
    />
  </teleport>
</template>

<style scoped>
.list-leave-active,
.list-enter-active {
  transition: all 0.3s;
}
.list-move {
  transition: transform 0.3s;
}
.list-enter,
.list-leave-to {
  opacity: 0;
}
.p-12
{
  padding-top: 12px !important;
}
.icon-delete
{
  padding-left: 10px;
  color: rgb(194, 6, 6);
  cursor: pointer;
}
.flex-center
{
  display: flex;
  justify-content: center;
}
</style>
