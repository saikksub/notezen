<script setup lang="ts">
import { ref } from "vue";
import { invoke } from "@tauri-apps/api/core";

const greetMsg = ref("");
const name = ref("");

async function greet() {
  // Learn more about Tauri commands at https://tauri.app/develop/calling-rust/
  greetMsg.value = await invoke("greet", { name: name.value });
}
</script>

<template>
  <div class="fixed top-0 right-64 bottom-0 left-0 bg-neutral-100 overflow-y-auto">
    <div class="grid grid-cols-3 gap-3 p-4">
      <div v-for="item in 100" :key="item" class="col-span-1 aspect-video bg-white border border-neutral-200 rounded-md" />
    </div>
  </div>
  <div class="fixed top-0 right-0 bottom-0 w-64 border-l border-neutral-200 px-3 py-2">
    <input class="w-full border border-neutral-300 rounded-md px-2 py-1 text-sm" placeholder="Search notes">
  </div>
</template>
