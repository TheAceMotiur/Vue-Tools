<script setup>
import { ref, computed } from 'vue';
import AppHeader from '../components/AppHeader.vue';
import AppFooter from '../components/AppFooter.vue';
import ToolsSection from '../components/ToolsSection.vue';
import { toolCategories } from '../tools/index.js';

const searchQuery = ref('');

function handleSearch(query) {
  searchQuery.value = query.toLowerCase();
}

const filteredTools = computed(() => {
  if (!searchQuery.value) return toolCategories;
  
  return toolCategories.map(category => {
    const filteredItems = category.items.filter(tool => 
      tool.name.toLowerCase().includes(searchQuery.value) || 
      tool.description.toLowerCase().includes(searchQuery.value)
    );
    
    return {
      category: category.category,
      items: filteredItems
    };
  }).filter(category => category.items.length > 0);
});
</script>

<template>
  <div class="min-h-screen flex flex-col">
    <AppHeader @search="handleSearch" />
    
    <main class="flex-grow bg-gray-50">
      <div class="container mx-auto px-4 py-8">
        <div v-if="searchQuery" class="mb-6">
          <h2 class="text-lg font-semibold text-gray-700">
            Search results for: "{{ searchQuery }}"
          </h2>
        </div>
        
        <div v-for="category in filteredTools" :key="category.category">
          <ToolsSection 
            :category="category.category" 
            :tools="category.items"
          />
        </div>

        <div v-if="filteredTools.length === 0" class="text-center py-16">
          <svg xmlns="http://www.w3.org/2000/svg" class="mx-auto h-12 w-12 text-gray-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.172 16.172a4 4 0 015.656 0M9 10h.01M15 10h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
          </svg>
          <h3 class="mt-2 text-lg font-medium text-gray-900">No tools found</h3>
          <p class="mt-1 text-gray-500">Try searching for something else.</p>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
          <!-- Other tool cards will be here -->
        </div>
      </div>
    </main>
    
    <AppFooter />
  </div>
</template>
