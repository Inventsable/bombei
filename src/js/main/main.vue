<script setup lang="ts">
import { onMounted, onBeforeMount } from "vue";
import Panel from "../lib/Volt/panel.vue";
import { RouterLink, RouterView, useRouter } from "vue-router";
import { useSettings } from "../stores/settings";
import { useHelp } from "../stores/help";
import { useCore } from "../stores/core";
import { csi, evalES } from "../lib/utils/bolt";
const settings = useSettings(),
  help = useHelp(),
  core = useCore();

const root = `${csi.getSystemPath("extension")}/`,
  isPopup = /settings/i.test(window.__adobe_cep__.getExtensionId() + ""),
  isMainPanel = /main/i.test(window.__adobe_cep__.getExtensionId() + "");

onBeforeMount(async () => {
  if (!isPopup) {
    await settings.init();
    await core.init();
  } else {
    await help.init();
  }
});
onMounted(async () => {
  const router = useRouter();
  if (isPopup)
    router.push({
      path: `/settings/main`,
    });
  else {
    settings.preloadHelpPages();
  }
  csi.addEventListener("com.ping", (data: any) => {
    console.log(data);
  });
});
</script>

<template>
  <div class="app">
    <Panel>
      <router-view />
    </Panel>
  </div>
</template>

<style>
:root {
  --color-base: #413f4f;
  --color-blue: #8083ff;
  --color-green: #00ecb7;
  --color-alert: #f5bd00;
  --size-icon: 16px;
  --quad: cubic-bezier(0.48, 0.04, 0.52, 0.96);
  --quart: cubic-bezier(0.76, 0, 0.24, 1);
  --quint: cubic-bezier(0.86, 0, 0.16, 1);
  --drawer-width: 200px;
  --size-footer: 42px;
  --size-toolbar: 30px;
}
</style>
