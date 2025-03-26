<template>
  <div>

  </div>
</template>

<script setup lang="ts">
import type { HistoryType } from '@/types/history-type';

const props = withDefaults(
  defineProps<{
    history: HistoryType[];
  }>(), {
  }
)
</script>

<style scoped>

</style>