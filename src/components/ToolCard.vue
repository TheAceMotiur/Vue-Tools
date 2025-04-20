<script setup>
defineProps({
  icon: {
    type: String,
    required: true
  },
  name: {
    type: String,
    required: true
  },
  description: {
    type: String,
    default: ''
  },
  categorySlug: {
    type: String,
    required: true
  },
  slug: {
    type: String,
    required: true
  }
})
</script>

<template>
  <div class="bg-white rounded-lg shadow-md hover:shadow-lg transition-shadow p-4 flex flex-col h-full">
    <div class="flex items-center mb-3">
      <div class="bg-indigo-100 p-2 rounded-md mr-3">
        <div class="text-indigo-600" v-html="icon"></div>
      </div>
      <h3 class="font-medium text-lg text-gray-900">{{ name }}</h3>
    </div>
    <p v-if="description" class="text-gray-600 text-sm flex-grow">{{ description }}</p>
    <router-link 
      :to="`/tools/${categorySlug}/${slug}`" 
      class="mt-3 text-indigo-600 hover:text-indigo-800 text-sm font-medium"
    >
      Use Tool →
    </router-link>
  </div>
</template>
