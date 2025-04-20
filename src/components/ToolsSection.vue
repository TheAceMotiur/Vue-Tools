<script setup>
import ToolCard from './ToolCard.vue';

defineProps({
  category: {
    type: String,
    required: true
  },
  tools: {
    type: Array,
    required: true
  }
})
</script>

<template>
  <div class="mb-8">
    <h2 class="text-2xl font-bold text-gray-800 mb-4">{{ category }}</h2>
    <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-4">
      <ToolCard
        v-for="tool in tools"
        :key="tool.name"
        :icon="tool.icon"
        :name="tool.name"
        :description="tool.description"
        :category-slug="tool.categorySlug"
        :slug="tool.slug"
      />
    </div>
  </div>
</template>
