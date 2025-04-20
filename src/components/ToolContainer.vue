<script setup>
defineProps({
  title: {
    type: String,
    required: true
  },
  description: {
    type: String,
    default: ''
  }
});
</script>

<template>
  <div class="tool-container">
    <div class="mb-6">
      <h2 class="text-2xl font-bold text-gray-800">{{ title }}</h2>
      <p v-if="description" class="mt-2 text-gray-600">{{ description }}</p>
    </div>
    
    <div class="bg-white rounded-lg shadow-md p-6">
      <slot></slot>
    </div>
    
    <div class="mt-6 bg-gray-50 rounded-lg p-4 border border-gray-200">
      <h3 class="text-lg font-semibold text-gray-700 mb-2">How to use</h3>
      <slot name="instructions"></slot>
    </div>
  </div>
</template>
