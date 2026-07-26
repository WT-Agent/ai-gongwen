<template>
  <NomadsShowcase @apply-template="handleApply" />
</template>

<script setup lang="ts">
import NomadsShowcase from './NomadsShowcase.vue';

const emit = defineEmits(['apply-template']);

function handleApply(payload: any) {
  emit('apply-template', payload);
}
</script>
