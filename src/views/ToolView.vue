<script setup>
import { ref, computed, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';
import AppHeader from '../components/AppHeader.vue';
import AppFooter from '../components/AppFooter.vue';
import { findTool } from '../tools/index.js';

const route = useRoute();
const router = useRouter();
const tool = ref(null);
const loading = ref(true);
const error = ref(null);

const categorySlug = computed(() => route.params.category);
const toolSlug = computed(() => route.params.tool);

onMounted(async () => {
  try {
    const foundTool = findTool(categorySlug.value, toolSlug.value);
    
    if (!foundTool) {
      error.value = 'Tool not found';
      loading.value = false;
      return;
    }
    
    // Dynamically import the tool component
    const toolComponent = await import(`../tools/${foundTool.categorySlug}/${foundTool.slug}.vue`);
    tool.value = {
      ...foundTool,
      component: toolComponent.default
    };
  } catch (err) {
    console.error(err);
    error.value = 'Failed to load tool';
  } finally {
    loading.value = false;
  }
});

function handleSearch(query) {
  if (query) {
    router.push('/');
  }
}
</script>

<template>
  <div class="min-h-screen flex flex-col">
    <AppHeader @search="handleSearch" />
    
    <main class="flex-grow bg-gray-50">
      <div class="container mx-auto px-4 py-8">
        <div v-if="loading" class="flex justify-center py-16">
          <svg class="animate-spin h-8 w-8 text-indigo-500" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
          </svg>
        </div>
        
        <div v-else-if="error" class="text-center py-16">
          <svg xmlns="http://www.w3.org/2000/svg" class="mx-auto h-12 w-12 text-red-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
          </svg>
          <h3 class="mt-2 text-lg font-medium text-gray-900">{{ error }}</h3>
          <button @click="router.push('/')" class="mt-4 px-4 py-2 bg-indigo-600 text-white rounded hover:bg-indigo-700">
            Return Home
          </button>
        </div>
        
        <template v-else>
          <div class="mb-6 flex items-center">
            <button @click="router.push('/')" class="flex items-center text-indigo-600 hover:text-indigo-800">
              <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-1" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M9.707 16.707a1 1 0 01-1.414 0l-6-6a1 1 0 010-1.414l6-6a1 1 0 011.414 1.414L5.414 9H17a1 1 0 110 2H5.414l4.293 4.293a1 1 0 010 1.414z" clip-rule="evenodd" />
              </svg>
              Back to Tools
            </button>
            
            <h1 class="text-2xl font-bold text-gray-800 ml-4">{{ tool.name }}</h1>
          </div>
          
          <div class="bg-white rounded-lg shadow-md p-6">
            <component :is="tool.component" />
          </div>
        </template>
      </div>
    </main>
    
    <AppFooter />
  </div>
</template>
