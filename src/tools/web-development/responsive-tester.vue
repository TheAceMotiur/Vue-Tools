<script setup>
import { ref, computed, onMounted } from 'vue';
import ToolContainer from '../../components/ToolContainer.vue';

const url = ref('');
const viewportWidth = ref(375);
const viewportHeight = ref(667);
const scale = ref(1);
const isValidUrl = ref(true);
const isLoading = ref(false);
const error = ref('');

const commonDevices = [
  { name: 'iPhone SE', width: 375, height: 667 },
  { name: 'iPhone 12/13', width: 390, height: 844 },
  { name: 'iPhone 12/13 Pro Max', width: 428, height: 926 },
  { name: 'iPad', width: 810, height: 1080 },
  { name: 'iPad Pro', width: 1024, height: 1366 },
  { name: 'Galaxy S22', width: 360, height: 780 },
  { name: 'Pixel 6', width: 412, height: 915 },
  { name: 'Laptop', width: 1366, height: 768 },
  { name: 'Desktop', width: 1920, height: 1080 }
];

const commonOrientations = [
  { name: 'Portrait', icon: '↕️' },
  { name: 'Landscape', icon: '↔️' }
];

const frameUrl = computed(() => {
  if (!url.value) return '';
  
  try {
    // Check if URL is valid by creating a URL object
    new URL(url.value.startsWith('http') ? url.value : `https://${url.value}`);
    isValidUrl.value = true;
    
    // Return properly formatted URL
    return url.value.startsWith('http') ? url.value : `https://${url.value}`;
  } catch (e) {
    isValidUrl.value = false;
    return '';
  }
});

const iframeStyles = computed(() => {
  return {
    width: `${viewportWidth.value}px`,
    height: `${viewportHeight.value}px`,
    transform: `scale(${scale.value})`,
    transformOrigin: 'top left',
    border: '1px solid #ccc'
  };
});

const containerStyles = computed(() => {
  return {
    width: `${viewportWidth.value * scale.value}px`,
    height: `${viewportHeight.value * scale.value}px`,
    overflow: 'hidden'
  };
});

function setDevice(device) {
  viewportWidth.value = device.width;
  viewportHeight.value = device.height;
  
  // Adjust scale to fit in the container if needed
  adjustScale();
}

function swapDimensions() {
  const temp = viewportWidth.value;
  viewportWidth.value = viewportHeight.value;
  viewportHeight.value = temp;
  
  // Adjust scale to fit in the container if needed
  adjustScale();
}

function adjustScale() {
  // Get the container's maximum width (using a reasonable maximum size)
  const maxContainerWidth = 800;
  const maxContainerHeight = 600;
  
  // Calculate scale to fit within the container
  const widthScale = maxContainerWidth / viewportWidth.value;
  const heightScale = maxContainerHeight / viewportHeight.value;
  
  // Use the smaller scale to ensure it fits in both dimensions
  scale.value = Math.min(1, widthScale, heightScale);
}

function loadUrl() {
  if (!frameUrl.value) {
    error.value = 'Please enter a valid URL';
    return;
  }
  
  isLoading.value = true;
  error.value = '';
  
  // Simulate loading for demo purposes
  setTimeout(() => {
    isLoading.value = false;
  }, 1500);
}

onMounted(() => {
  adjustScale();
});
</script>

<template>
  <ToolContainer 
    title="Responsive Tester" 
    description="Test how your website looks on different device screen sizes."
  >
    <div class="space-y-6">
      <div>
        <label for="site-url" class="block text-sm font-medium text-gray-700 mb-2">
          Enter website URL:
        </label>
        <div class="flex">
          <input
            id="site-url"
            v-model="url"
            type="text"
            class="block w-full rounded-l-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
            placeholder="example.com"
            :class="{ 'border-red-300': !isValidUrl }"
          />
          <button
            @click="loadUrl"
            class="inline-flex items-center px-4 py-2 border border-transparent rounded-r-md shadow-sm text-sm font-medium text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
            :disabled="isLoading"
          >
            <template v-if="isLoading">
              <svg class="animate-spin -ml-1 mr-2 h-4 w-4 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
              </svg>
              Loading...
            </template>
            <template v-else>
              Load
            </template>
          </button>
        </div>
        <p v-if="!isValidUrl && url" class="mt-1 text-sm text-red-600">
          Please enter a valid URL
        </p>
      </div>
      
      <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
        <div class="space-y-4">
          <div>
            <h3 class="text-sm font-medium text-gray-700 mb-2">Device Presets:</h3>
            <div class="grid grid-cols-2 md:grid-cols-1 gap-2">
              <button
                v-for="device in commonDevices"
                :key="device.name"
                @click="setDevice(device)"
                class="px-3 py-2 border border-gray-300 rounded text-sm text-left hover:bg-gray-50"
              >
                <span class="font-medium">{{ device.name }}</span>
                <span class="text-xs text-gray-500 block">{{ device.width }}×{{ device.height }}</span>
              </button>
            </div>
          </div>
          
          <div>
            <h3 class="text-sm font-medium text-gray-700 mb-2">Orientation:</h3>
            <div class="flex space-x-2">
              <button
                @click="swapDimensions"
                class="flex items-center justify-center w-10 h-10 border border-gray-300 rounded hover:bg-gray-50"
                title="Swap orientation"
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7h12m0 0l-4-4m4 4l-4 4m0 6H4m0 0l4 4m-4-4l4-4" />
                </svg>
              </button>
            </div>
          </div>
          
          <div>
            <h3 class="text-sm font-medium text-gray-700 mb-2">Custom Size:</h3>
            <div class="grid grid-cols-2 gap-2">
              <div>
                <label for="width" class="block text-xs text-gray-500">Width (px)</label>
                <input
                  id="width"
                  v-model.number="viewportWidth"
                  type="number"
                  min="200"
                  max="2560"
                  class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                  @change="adjustScale"
                />
              </div>
              <div>
                <label for="height" class="block text-xs text-gray-500">Height (px)</label>
                <input
                  id="height"
                  v-model.number="viewportHeight"
                  type="number"
                  min="200"
                  max="2560"
                  class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                  @change="adjustScale"
                />
              </div>
            </div>
          </div>
          
          <div>
            <label for="scale" class="block text-sm font-medium text-gray-700 mb-2">
              Scale: {{ Math.round(scale.value * 100) }}%
            </label>
            <input
              id="scale"
              v-model.number="scale"
              type="range"
              min="0.1"
              max="1"
              step="0.05"
              class="block w-full"
            />
          </div>
        </div>
        
        <div class="md:col-span-2">
          <div class="bg-gray-100 p-4 rounded-lg flex items-start justify-center min-h-[400px]">
            <div v-if="error" class="text-center text-red-500">
              {{ error }}
            </div>
            
            <div v-else-if="!frameUrl" class="text-center text-gray-500">
              <svg xmlns="http://www.w3.org/2000/svg" class="h-16 w-16 mx-auto mb-4 text-gray-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 18h.01M8 21h8a2 2 0 002-2V5a2 2 0 00-2-2H8a2 2 0 00-2 2v14a2 2 0 002 2z" />
              </svg>
              <p>Enter a URL above and click "Load" to preview</p>
            </div>
            
            <div v-else :style="containerStyles" class="bg-white overflow-hidden">
              <div v-if="isLoading" class="flex items-center justify-center h-full">
                <svg class="animate-spin h-10 w-10 text-indigo-500" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                  <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                  <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                </svg>
              </div>
              
              <iframe
                v-else
                :src="frameUrl"
                :style="iframeStyles"
                frameborder="0"
                title="Website preview"
                sandbox="allow-same-origin allow-scripts"
              ></iframe>
            </div>
          </div>
          
          <div class="flex justify-between mt-2 text-xs text-gray-500">
            <span>Current viewport: {{ viewportWidth }} × {{ viewportHeight }}</span>
            <span>Scale: {{ Math.round(scale * 100) }}%</span>
          </div>
        </div>
      </div>
    </div>
    
    <template #instructions>
      <ol class="list-decimal list-inside space-y-2 text-gray-600">
        <li>Enter the website URL you want to test</li>
        <li>Click the "Load" button to preview the site</li>
        <li>Select a device preset or set custom dimensions</li>
        <li>Toggle orientation to switch between portrait and landscape</li>
        <li>Adjust the scale slider if needed to fit the preview in your screen</li>
        <li>Note: Some websites may block embedding in iframes for security reasons</li>
      </ol>
    </template>
  </ToolContainer>
</template>
