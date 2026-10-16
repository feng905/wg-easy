<template>
  <main v-if="data">
    <FormElement @submit.prevent="submit">
      <FormGroup>
        <FormNumberField
          id="session"
          v-model="data.sessionTimeout"
          :label="$t('admin.general.sessionTimeout')"
          :description="$t('admin.general.sessionTimeoutDesc')"
        />
      </FormGroup>
      <FormGroup>
        <FormHeading>{{ $t('admin.general.metrics') }}</FormHeading>
        <FormNullTextField
          id="password"
          v-model="data.metricsPassword"
          :label="$t('admin.general.metricsPassword')"
          :description="$t('admin.general.metricsPasswordDesc')"
        />
        <FormSwitchField
          id="prometheus"
          v-model="data.metricsPrometheus"
          :label="$t('admin.general.prometheus')"
          :description="$t('admin.general.prometheusDesc')"
        />
        <FormSwitchField
          id="json"
          v-model="data.metricsJson"
          :label="$t('admin.general.json')"
          :description="$t('admin.general.jsonDesc')"
        />
      </FormGroup>
      <FormGroup>
        <FormHeading>{{ $t('admin.general.liveWireGuard') }}</FormHeading>
        <FormSwitchField
          id="liveWireGuard"
          v-model="data.liveWireGuardEnabled"
          :label="$t('admin.general.liveWireGuardEnabled')"
          :description="$t('admin.general.liveWireGuardEnabledDesc')"
        />
        <FormTextField
          id="liveWireGuardAudience"
          v-model="data.liveWireGuardAudience"
          :label="$t('admin.general.liveWireGuardAudience')"
          :description="$t('admin.general.liveWireGuardAudienceDesc')"
        />
        <FormInfoField
          id="liveWireGuardEndpoint"
          :data="liveWireGuardEndpoint"
          :label="$t('admin.general.liveWireGuardEndpoint')"
          :description="$t('admin.general.liveWireGuardEndpointDesc')"
        />
      </FormGroup>
      <FormGroup>
        <FormHeading>{{ $t('form.actions') }}</FormHeading>
        <FormPrimaryActionField type="submit" :label="$t('form.save')" />
        <FormSecondaryActionField :label="$t('form.revert')" @click="revert" />
      </FormGroup>
    </FormElement>
  </main>
</template>

<script setup lang="ts">
const { data: _data, refresh } = await useFetch(`/api/admin/general`, {
  method: 'get',
});
const data = toRef(_data.value);

// Provisioning URL handed to the LiveWireGuard app; only meaningful in the browser
const liveWireGuardEndpoint = ref('');
onMounted(() => {
  liveWireGuardEndpoint.value = `${window.location.origin}/api/livewireguard/config`;
});

const _submit = useSubmit(
  (data) =>
    $fetch(`/api/admin/general`, {
      method: 'post',
      body: data,
    }),
  { revert }
);

function submit() {
  return _submit(data.value);
}

async function revert() {
  await refresh();
  data.value = toRef(_data.value).value;
}
</script>
