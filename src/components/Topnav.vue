<script setup>
import { ref, computed, watch, onMounted } from 'vue';
import { useRoute } from 'vue-router';
import { useModal } from '@/composables/useModal';
import { useDomain } from '@/composables/useDomain';
import { useApp } from '@/composables/useApp';
import { useWeb3 } from '@/composables/useWeb3';
import { useTxStatus } from '@/composables/useTxStatus';
import { useUserSkin } from '@/composables/useUserSkin';
import addImg from '@/assets/images/add.png';
const { pendingCount } = useTxStatus();
const { modalAccountOpen } = useModal();
const { env, domain } = useDomain();
const route = useRoute();

const { explore } = useApp();
const { login, web3 } = useWeb3();
const { toggleSkin, getSkinIcon } = useUserSkin();

const loading = ref(false);
const modalWalletNotice = ref(false);

const space = computed(() => {
  const key = domain || route.params.key;
  return explore.value.space?.[key];
});


function setTitle() {
  document.title = space.value?.name ?? 'Meow Governance';
}

async function handleLogin(connector) {
  modalAccountOpen.value = false;
  loading.value = true;
  await login(connector);
  loading.value = false;
}

watch(space, () => {
  setTitle();
});

const walletConnectType = computed(() => web3.value.walletConnectType);

watch(walletConnectType, val => {
  if (val === 'Gnosis Safe Multisig') modalWalletNotice.value = true;
});

onMounted(() => setTitle());
</script>

<template>
  <div class="mb-4">
    <div
      v-if="env === 'develop'"
      class="p-3 text-center bg-blue"
      style="color: white; font-size: 20px"
    >
      {{ $t('demoSite') }}
    </div>
    <nav id="topnav" class="border-b w-full">
      <Container>
        <div class="flex" style="height: auto;margin-top: 15px;margin-bottom: 10px">
          <div class="flex-auto flex items-center" 
          style="display: flex;
              align-items: flex-start;
              flex-direction: column;">
            <div
              :to="{ path: '/' }"
              class="flex items-center topic-text"
              style="font-weight: 800;font-size: 60px; padding-top: 12px;;line-height:2.5rem"
            >
             {{route.path != '/create' ? 'Voting' : 'New Proposal'}}
            </div>
            <label style="padding-top: 10px;" >Have your say in the future of the Meow Finance Ecosystem</label>
             <UiButton 
              v-if="route.path == '/'"
             >
              <UiButtonIcon
                    :imgsrc="addImg"
                    width="16"
                    heigth="16"
                />
                <router-link :to="{ path: 'create'}" class="button-text" >
                    Make a Proposal
                </router-link>
             </UiButton>

              <div class="px-4 md:px-0 mb-3" style="padding-top: 0.5rem;"
                v-if="route.path != '/'"
              >
              <router-link
                :to="domain ? { path: '/' } : { name: 'spaceProposals' }"
                class="text-color"
              >
                <Icon name="back" size="22" class="!align-middle" />
                {{ 'Voting' }}
              </router-link>
            </div>
          </div>
          <div :key="web3.account">
            <template v-if="$auth.isAuthenticated.value">
              <UiButton
                @click="modalAccountOpen = true"
                :loading="web3.authLoading"
                class="flex items-center float-left"
              >
                <UiAvatar
                  :imgsrc="
                    web3.profile?.image ? _getUrl(web3.profile.image) : ''
                  "
                  :address="web3.account"
                  size="16"
                  class="-mr-1 sm:mr-2 md:mr-2 lg:mr-2 xl:mr-2 -ml-1"
                />
                <span
                  v-if="web3.profile?.name || web3.profile?.ens"
                  v-text="web3.profile.name || web3.profile.ens"
                  class="hidden sm:block"
                />
                <span
                  v-else
                  v-text="_shorten(web3.account)"
                  class="hidden sm:block"
                />
              </UiButton>
            </template>
            <UiButton
              v-if="!$auth.isAuthenticated.value"
              @click="modalAccountOpen = true"
              :loading="loading || web3.authLoading"
            >
              <span class="hidden sm:block" v-text="$t('connectWallet')" />
              <Icon
                name="login"
                size="20"
                class="sm:hidden -ml-2 -mr-2 block align-text-bottom"
              />
            </UiButton>
            <UiSidebarButton
              v-if="!domain"
              @click="toggleSkin"
              class="float-right ml-2"
            >
              <Icon size="20" class="link-color" :name="getSkinIcon()" />
            </UiSidebarButton>
          </div>
        </div>
      </Container>
    </nav>
    <div class="bg-blue text-white text-center py-2" v-if="pendingCount > 0">
      <UiLoading :fill-white="true" class="mr-2" />
      {{ $tc('delegate.pendingTransaction', pendingCount) }}
    </div>
    <teleport to="#modal">
      <ModalAccount
        :open="modalAccountOpen"
        @close="modalAccountOpen = false"
        @login="handleLogin"
      />
      <ModalWalletNotice
        :open="modalWalletNotice"
        @close="modalWalletNotice = false"
      />
    </teleport>
  </div>
</template>


<style scoped lang="scss">
  .topic-text
  {
    color: var(--text-head-color) !important;
  }
  .create-color
  {
    color: var(--link-color) !important;
  }
  .button-text
  {
    color: var(--button-text) !important;
  }
</style>
