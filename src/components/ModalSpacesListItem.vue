<script setup lang="ts">
const { formatCompactNumber } = useIntl();

defineProps<{
  space: any;
}>();
</script>

<template>
  <router-link :to="{ name: 'spaceProposals', params: { key: space.id } }">
    <BaseBlock class="">
      <div class="flex justify-between">
        <div class="flex min-w-0">
          <AvatarSpace :space="space" size="44" />
          <div class="ml-3 mr-3 truncate">
            <div class="flex items-center">
              <div class="truncate">
                {{ space.name }}
              </div>
              <IconVerifiedSpace
                class="ml-1 flex text-primary"
                :space-id="space.id"
                size="18"
              />
            </div>
            <div class="text-xs leading-5 text-skin-text">
              {{
                $tc('members', {
                  count: formatCompactNumber(space.followers || 0)
                })
              }}
            </div>
          </div>
        </div>
        <ButtonFollow class="!mb-0 !h-[44px] !w-[100px]" :space="space" />
      </div>
    </BaseBlock>
  </router-link>
</template>
