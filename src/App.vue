<template>
  <div class="h-full flex flex-col bg-white shadow-sm">
    <div class="flex-1 overflow-y-auto p-4">
      <MessageComponent />
    </div>
  </div>
  <!-- main div end -->
</template>

<script setup>
import MessageComponent from "./components/Message.vue";
import { ref } from "vue";

const props = ref("None");
</script>
