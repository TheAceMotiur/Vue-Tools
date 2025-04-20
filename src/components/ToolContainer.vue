<script setup>
import { ref } from 'vue';
import AdComponent from './AdComponent.vue';

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

const showInstructions = ref(false);
</script>

<template>
  <div class="bg-white shadow-sm rounded-lg overflow-hidden">
    <div class="px-4 py-5 sm:px-6 border-b border-gray-200">
      <h1 class="text-xl font-semibold text-gray-900">{{ title }}</h1>
      <p v-if="description" class="mt-1 text-sm text-gray-600">{{ description }}</p>
    </div>
    
    <!-- Ad before content -->
    <AdComponent />
    
    <div class="px-4 py-5 sm:p-6">
      <slot></slot>
    </div>
    
    <!-- Ad after content -->
    <AdComponent />
    
    <div v-if="$slots.instructions" class="px-4 py-4 sm:px-6 bg-gray-50 border-t border-gray-200">
      <div class="flex items-center justify-between mb-3">
        <h3 class="text-lg font-medium text-gray-900">How to use this tool</h3>
        <button 
          @click="showInstructions = !showInstructions"
          class="text-sm text-indigo-600 hover:text-indigo-900"
        >
          {{ showInstructions ? 'Hide' : 'Show' }} instructions
        </button>
      </div>
      <div v-show="showInstructions" class="text-sm">
        <slot name="instructions"></slot>
      </div>
    </div>
  </div>
</template>
