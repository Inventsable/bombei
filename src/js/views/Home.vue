<script setup lang="ts">
import { onMounted, ref, computed, watch, onBeforeMount, nextTick } from "vue";
import { useSettings } from "../stores/settings";
import { evalES, csi } from "../lib/utils/utils";
import type { Ref } from "vue";
import flyoutMenu from "../lib/Volt/flyout-menu.vue";
import contextMenu from "../lib/Volt/context-menu.vue";
import Button from "../lib/Volt/button.vue";
import type {
  FlyoutMenuItem,
  FlyoutMenu,
  ContextMenuItem,
  ContextMenu,
} from "../lib/Volt/types";
import type { ContextMenuItemProp } from "../lib/Volt/types/props";
import checkbox from "../lib/Volt/checkbox.vue";
import { path } from "../lib/cep/node";
import { exists } from "../lib/utils/fs";
import type { ColorValue } from "../../shared/shared";

// @ts-ignore - These don't have explicit types since the file is from Adobe
import { SystemPath } from "../lib/cep/csinterface";
import { appThemeChanged } from "../lib/utils/theme-manager";

const settings = useSettings();

const workerExample = async () => {
  const selectionWorker = new Worker(
    new URL("../worker/colorSelectionFilter.js", import.meta.url)
  );
  const docWorker = new Worker(
    new URL("../worker/documentScanMerger.js", import.meta.url)
  );
  // console.log("Scan document")
  // const result = JSON.parse(
  //   await evalES(`deepScan('${JSON.stringify(settings.deepScanOptions)}')`)
  // );
  if (window.Worker) {
    const timeStart = Date.now();
    // Pass fills and strokes from JSX into a ./worker/worker.js:
    const payload = JSON.stringify(result);
    docWorker.postMessage(payload);

    docWorker.onmessage = (response: any) => {
      const parsedResult = JSON.parse(response.data as string);
      if (parsedResult.error) {
        // Something went wrong
        console.error(parsedResult.error);
      } else {
        const timeTotal = `${parsedResult.timeEnd - timeStart}ms`;
        console.log(`DEEP SCAN @${timeTotal}:`, parsedResult.list);
        settings.setHardList(parsedResult.list);
      }
    };
  } else {
    // In theory this should never be triggered unless some false-positive triggers on webworker availability
    console.error("No webworker is available in this environment");
  }
};

function forcePopup() {
  function openPopup() {
    csi.requestOpenExtension("com.warlock.cep.settings", "");
  }
  openPopup();
  setTimeout(() => {
    openPopup();
  }, 1000);
}

/** CONTEXT / FLYOUT MENUS */
const buildContextMenu = () => {
  return [
    {
      label: "Show indicator",
      checked: showIndicator.value,
      checkable: true,
      enabled: true,
      id: "showIndicator",
    },
    {
      label: "Show help",
      enabled: true,
      id: "showHelp",
      callback: () => {
        console.log("Popup?");
        forcePopup();
      },
    },
    {
      label: "Filters",
      menu: [
        {
          label: "Actives always on top",
          checkable: true,
          checked: settings.filters.indicatorsOnTop,
          callback: () => {
            settings.filters.indicatorsOnTop =
              !settings.filters.indicatorsOnTop;
          },
        },
        {
          label: "---",
        },
        {
          label: "Sort by hue",
          checkable: true,
          checked: settings.filters.byHue,
          callback: () => {
            settings.toggleSortByHue(!settings.filters.byHue);
          },
        },
        {
          label: "Sort by saturation",
          checkable: true,
          checked: settings.filters.bySaturation,
          callback: () => {
            settings.toggleSortBySaturation(!settings.filters.bySaturation);
          },
        },
        {
          label: "Sort by frequency",
          checkable: true,
          checked: settings.filters.byFrequency,
          callback: () => {
            settings.toggleSortByFrequency(!settings.filters.byFrequency);
          },
        },
      ],
    },
    {
      label: "---",
    },
    {
      label: "Delete AppData",
      callback: () => {
        settings.deleteSettings();
        console.log("Local settings deleted");
      },
    },
    {
      label: "Delete local storage",
      callback: () => {
        window.localStorage.removeItem("settings");
        console.log("Local storage deleted");
      },
    },
  ] as ContextMenuItem[];
};

const contextMenuRef = ref(buildContextMenu());

/**
 * Used to bypass update:modelValue failing to echo with computed getter
 */
const checkClick = (item: ContextMenuItem | ContextMenuItemProp) => {
  if (item.id && /indicator/i.test(item.id)) {
    showIndicator.value = !item.checked;
  } else {
    // This is stupid, but I can't get it to react to my Pinia state otherwise
    contextMenuRef.value = [];
    // So clear the value completely
    nextTick(() => {
      // And on nextTick, repopulate the menu with updated values to force a redraw
      contextMenuRef.value = buildContextMenu();
    });
  }
};

/** LIFECYCLE HOOKS */
onBeforeMount(async () => {
  //
});

onMounted(async () => {
  console.log("Sync to indicator");
  await syncToAppIndicator();
  await shallowScan();
  console.log("Mounted");
  // For testing
  // const result = JSON.parse(await evalES(`getActiveFillColor()`))
  // console.log(result);
});
</script>
<template>
  <flyoutMenu refresh />
  <contextMenu refresh v-model="contextMenuRef" @click="checkClick" />
  <div class="home-content">Hello world</div>
</template>

<style>
:root {
  --color-warning: #ffee00;
}

.home-content {
  overflow-y: hidden;
}

.panel-content {
  margin: 6px 0px;
}

/* Slim */
@media screen and (max-width: 249px) {
}
</style>
