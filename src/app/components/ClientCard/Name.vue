<template>
  <div
    class="break-all text-sm text-gray-700 md:text-base dark:text-neutral-200"
    :title="$t('client.createdOn', { date: $d(new Date(client.createdAt)) })"
  >
    {{ client.name }}
    <span
      v-if="client.deviceId !== null"
      class="ml-1 rounded bg-gray-200 px-1.5 py-0.5 align-middle text-xs text-gray-600 dark:bg-neutral-700 dark:text-neutral-300"
      :title="$t('client.liveWireGuardDevice')"
    >
      LiveWireGuard
    </span>
  </div>
</template>

<script setup lang="ts">
defineProps<{
  client: LocalClient;
}>();
</script>
