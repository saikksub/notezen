<template>
  <div class="fixed top-0 right-64 bottom-0 left-0 bg-neutral-100 overflow-y-auto">
    {{ triggerCounter }} - {{ triggerContentHash }}
    <div class="grid grid-cols-3 gap-3 p-4">
      <div v-for="item in 100" :key="item" class="col-span-1 aspect-video bg-white border border-neutral-200 rounded-md" />
    </div>
  </div>
  <div class="fixed top-0 right-0 bottom-0 w-64 border-l border-neutral-200 px-3 py-2">
    <input class="w-full border border-neutral-300 rounded-md px-2 py-1 text-sm" placeholder="Search notes">
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from "vue";
import { invoke } from "@tauri-apps/api/core";
import { readText } from '@tauri-apps/plugin-clipboard-manager'
import {
  startListening,
  onClipboardChange,
} from "tauri-plugin-clipboard-x-api";

const greetMsg = ref("");
const name = ref("");

const triggerContentHash = ref('')
const triggerCounter = ref(0)
let triggerTimer: ReturnType<typeof setTimeout> | null = null

async function greet() {
  // Learn more about Tauri commands at https://tauri.app/develop/calling-rust/
  greetMsg.value = await invoke("greet", { name: name.value });
}

function checksum(str) {
  let hash = 5381;
  for (let i = 0; i < str.length; i++) {
    hash = (hash * 33) ^ str.charCodeAt(i);
  }
  return (hash >>> 0).toString(16); // unsigned 32-bit hex
}

onMounted(async () => {
  await startListening();

  const unlisten = await onClipboardChange((result) => {
    const checksumValue = checksum(JSON.stringify(result))

    if (checksumValue === triggerContentHash.value) {
      triggerCounter.value += 1

      if (triggerCounter.value >= 2) {
        if (triggerTimer) clearTimeout(triggerTimer)
        triggerTimer = null
        triggerCounter.value = 0
        triggerContentHash.value = ''
        console.log('hit', result)
      }
    } else {
      if (triggerTimer) clearTimeout(triggerTimer)
      triggerContentHash.value = checksumValue
      triggerCounter.value = 1
      triggerTimer = setTimeout(() => {
        triggerCounter.value = 0
        triggerContentHash.value = ''
        triggerTimer = null
      }, 1000)
    }
  });
})
</script>
