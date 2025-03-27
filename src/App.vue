<template>
  <div class="wrapper">
    <Map :history="data ?? []" />
  </div>
</template>

<script setup lang="ts">
import axios from "axios";
import Map from "./components/Map.vue";
import type { HistoryType } from "@/types/history-type";
import { ref } from "vue";

const data = ref<HistoryType[]>();

axios
  .get("/data/history.json")
  .then((response) => {
    data.value = response.data;
  })
  .catch((error) => {
    console.error("Помилка завантаження даних:", error);
  });
</script>

<style scoped>
.wrapper {
  width: fit-content;
  margin: 50px auto;
  position: relative;
}
</style>
