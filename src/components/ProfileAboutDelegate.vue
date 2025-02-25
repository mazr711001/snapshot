<script setup lang="ts">
import { getDelegators } from '@/helpers/delegation';
import uniqBy from 'lodash/uniqBy';
import networks from '@snapshot-labs/snapshot.js/src/networks.json';
import { SNAPSHOT_SUBGRAPH_URL } from '@snapshot-labs/snapshot.js/src/utils';

const props = defineProps<{
  userAddress: string;
  followingSpaces: { space: { id: string } }[];
  loadingFollowedSpaces: boolean;
}>();

const { spaces, spacesLoaded } = useSpaces();
const { web3Account } = useWeb3();
const { networkKey } = useDelegate();
const { t } = useI18n();

const delegators = ref<{ delegator: string; space: string }[] | null>(null);

// Filter delegators by the spaces the user is following
const delegatorsFilteredBySpaces = computed(() => {
  if (!delegators.value) return [];

  const followingSpaceIds = props.followingSpaces.map(s => s.space.id);
  const delegatorSpaceIds = delegators.value
    .map(d => d.space)
    .filter(d => d !== '');

  return uniqBy(delegatorSpaceIds.filter(d => followingSpaceIds.includes(d)));
});

const loadingDelegators = ref<boolean | 'notSupportedNetwork'>(false);

async function loadDelegatorsByNetwork() {
  loadingDelegators.value = true;
  if (SNAPSHOT_SUBGRAPH_URL[networkKey.value]) {
    const res = await getDelegators(networkKey.value, props.userAddress);
    delegators.value = res.delegations ?? [];
    return (loadingDelegators.value = false);
  }
  loadingDelegators.value = 'notSupportedNetwork';
}

watch(
  networkKey,
  async () => {
    if (!web3Account.value) return;
    loadDelegatorsByNetwork();
  },
  { immediate: true }
);

// Delegate modal
const modalDelegateOpen = ref(false);
const delegateSpaceId = ref('');

function clickDelegate(id) {
  delegateSpaceId.value = id;
  modalDelegateOpen.value = true;
}

const networkString = computed(() => {
  return (
    networks?.[networkKey.value]?.shortName ??
    networks?.[networkKey.value]?.name ??
    t('theCurrentNetwork')
  );
});
</script>

<template>
  <div>
    <BaseBlock
      :loading="
        !spacesLoaded || loadingFollowedSpaces || loadingDelegators === true
      "
      :title="$t('profile.about.delegateFor')"
      :counter="delegatorsFilteredBySpaces.length"
      :label="networkString"
      :label-tooltip="$t('profile.about.delegatorNetworkInfo')"
      hide-bottom-border
      slim
    >
      <ProfileAboutDelegateListItem
        v-if="delegatorsFilteredBySpaces.length && spacesLoaded && delegators"
        :spaces="spaces"
        :delegators-filtered-by-spaces="delegatorsFilteredBySpaces"
        :delegators="delegators"
        :user-address="userAddress"
        :web3-account="web3Account"
        @delegate="clickDelegate"
      />
      <div v-else class="border-t p-4">
        {{
          loadingDelegators === 'notSupportedNetwork'
            ? $t('profile.about.notSupportedNetwork', {
                network: networkString
              })
            : $t('profile.about.noDelegatorsMessage', {
                network: networkString
              })
        }}
      </div>
    </BaseBlock>
  </div>
  <Teleport to="#modal">
    <ModalDelegate
      :open="modalDelegateOpen"
      :user-address="userAddress"
      :space-id="delegateSpaceId"
      @close="modalDelegateOpen = false"
      @reload="loadDelegatorsByNetwork"
    />
  </Teleport>
</template>
