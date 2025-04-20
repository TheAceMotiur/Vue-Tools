<script setup>
import { ref } from 'vue';

const searchQuery = ref('');
const emit = defineEmits(['search']);

function handleSearch() {
  emit('search', searchQuery.value);
}
</script>

<template>
  <header class="bg-indigo-600 shadow-md">
    <div class="container mx-auto px-4 py-3">
      <div class="flex items-center justify-between">
        <div class="flex items-center">
          <h1 class="text-white text-2xl font-bold">FreeNetly</h1>
        </div>
        
        <div class="w-full max-w-md mx-4">
          <div class="relative">
            <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
              <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-gray-300" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M8 4a4 4 0 100 8 4 4 0 000-8zM2 8a6 6 0 1110.89 3.476l4.817 4.817a1 1 0 01-1.414 1.414l-4.816-4.816A6 6 0 012 8z" clip-rule="evenodd" />
              </svg>
            </div>
            <input
              v-model="searchQuery"
              @input="handleSearch"
              type="text"
              class="block w-full pl-10 pr-3 py-2 border border-transparent rounded-md leading-5 bg-indigo-400 bg-opacity-25 text-white placeholder-gray-300 focus:outline-none focus:bg-white focus:placeholder-gray-500 focus:text-gray-900 focus:ring-2 focus:ring-indigo-300"
              placeholder="Live search tools..."
            />
          </div>
        </div>

        <div class="flex items-center">
          <a href="#" class="text-white text-sm hover:underline">About</a>
          <a href="#" class="text-white text-sm ml-4 hover:underline">Contact</a>
        </div>
      </div>
    </div>
  </header>
</template>
