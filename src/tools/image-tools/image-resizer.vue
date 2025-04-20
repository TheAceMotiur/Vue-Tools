<script setup>
import { ref, computed, watch } from 'vue';
import ToolContainer from '../../components/ToolContainer.vue';

const selectedFile = ref(null);
const resizedImage = ref(null);
const previewUrl = ref('');
const resizedUrl = ref('');
const originalWidth = ref(0);
const originalHeight = ref(0);
const newWidth = ref(0);
const newHeight = ref(0);
const maintainAspectRatio = ref(true);
const aspectRatio = ref(1);
const resizeMethod = ref('dimensions'); // 'dimensions', 'percentage', 'preset'
const resizePercentage = ref(50);
const isProcessing = ref(false);
const error = ref('');

const presetSizes = [
  { name: 'Instagram Post', width: 1080, height: 1080 },
  { name: 'Instagram Story', width: 1080, height: 1920 },
  { name: 'Facebook Post', width: 1200, height: 630 },
  { name: 'Twitter Post', width: 1200, height: 675 },
  { name: 'LinkedIn Post', width: 1200, height: 627 },
  { name: 'YouTube Thumbnail', width: 1280, height: 720 },
  { name: 'HD (1080p)', width: 1920, height: 1080 },
  { name: '4K UHD', width: 3840, height: 2160 },
];

const resizePercentageText = computed(() => {
  return `${resizePercentage.value}%`;
});

function handleFileChange(event) {
  const file = event.target.files[0];
  if (!file) return;
  
  // Check if file is an image
  if (!file.type.match('image.*')) {
    error.value = 'Please select an image file (JPEG, PNG, WebP, GIF)';
    return;
  }
  
  // Reset values
  error.value = '';
  selectedFile.value = file;
  resizedImage.value = null;
  resizedUrl.value = '';
  
  // Load image to get dimensions
  const img = new Image();
  img.onload = () => {
    originalWidth.value = img.width;
    originalHeight.value = img.height;
    aspectRatio.value = img.width / img.height;
    
    // Set initial resize values
    newWidth.value = Math.round(img.width / 2);
    newHeight.value = Math.round(img.height / 2);
  };
  img.onerror = () => {
    error.value = 'Error loading image';
  };
  
  // Create preview URL
  const reader = new FileReader();
  reader.onload = (e) => {
    previewUrl.value = e.target.result;
    img.src = e.target.result;
  };
  reader.readAsDataURL(file);
}

// Update height when width changes (if maintaining aspect ratio)
watch(newWidth, (width) => {
  if (maintainAspectRatio.value && aspectRatio.value) {
    newHeight.value = Math.round(width / aspectRatio.value);
  }
});

// Update width when height changes (if maintaining aspect ratio)
watch(newHeight, (height) => {
  if (maintainAspectRatio.value && aspectRatio.value) {
    newWidth.value = Math.round(height * aspectRatio.value);
  }
});

// Update dimensions when percentage changes
watch(resizePercentage, (percentage) => {
  if (resizeMethod.value === 'percentage' && originalWidth.value && originalHeight.value) {
    newWidth.value = Math.round((originalWidth.value * percentage) / 100);
    newHeight.value = Math.round((originalHeight.value * percentage) / 100);
  }
});

function applyPreset(preset) {
  newWidth.value = preset.width;
  newHeight.value = preset.height;
  maintainAspectRatio.value = false;
  resizeMethod.value = 'dimensions';
}

function resizeImage() {
  if (!selectedFile.value) {
    error.value = 'Please select an image file first';
    return;
  }
  
  if (newWidth.value <= 0 || newHeight.value <= 0) {
    error.value = 'Width and height must be greater than zero';
    return;
  }
  
  isProcessing.value = true;
  error.value = '';
  
  // Create an image object
  const img = new Image();
  img.onload = () => {
    // Create canvas
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d');
    
    // Set dimensions
    canvas.width = newWidth.value;
    canvas.height = newHeight.value;
    
    // Draw resized image on canvas
    ctx.drawImage(img, 0, 0, newWidth.value, newHeight.value);
    
    // Convert to blob
    canvas.toBlob(
      (blob) => {
        if (blob) {
          resizedImage.value = new File([blob], selectedFile.value.name, {
            type: blob.type,
          });
          resizedUrl.value = URL.createObjectURL(blob);
        } else {
          error.value = 'Failed to resize the image';
        }
        isProcessing.value = false;
      },
      selectedFile.value.type
    );
  };
  
  img.onerror = () => {
    error.value = 'Error loading image';
    isProcessing.value = false;
  };
  
  img.src = previewUrl.value;
}

function downloadResizedImage() {
  if (!resizedImage.value) return;
  
  const link = document.createElement('a');
  link.href = resizedUrl.value;
  link.download = `resized-${resizedImage.value.name}`;
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
}
</script>

<template>
  <ToolContainer 
    title="Image Resizer" 
    description="Resize your images to specific dimensions or proportions."
  >
    <div class="space-y-6">
      <div>
        <label for="image-file" class="block text-sm font-medium text-gray-700 mb-2">
          Select an image to resize:
        </label>
        <input
          id="image-file"
          type="file"
          accept="image/*"
          @change="handleFileChange"
          class="block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded file:border-0 file:text-sm file:font-semibold file:bg-indigo-50 file:text-indigo-700 hover:file:bg-indigo-100"
        />
      </div>
      
      <div v-if="error" class="bg-red-50 border-l-4 border-red-400 p-4 text-red-700">
        {{ error }}
      </div>
      
      <div v-if="selectedFile" class="space-y-6">
        <div>
          <h3 class="text-sm font-medium text-gray-700 mb-2">Original Dimensions:</h3>
          <p class="text-sm text-gray-600">{{ originalWidth }} × {{ originalHeight }} pixels</p>
        </div>
        
        <div class="space-y-4">
          <div class="flex space-x-4">
            <label class="inline-flex items-center">
              <input
                type="radio"
                v-model="resizeMethod"
                value="dimensions"
                class="form-radio text-indigo-600"
              />
              <span class="ml-2">Custom Dimensions</span>
            </label>
            <label class="inline-flex items-center">
              <input
                type="radio"
                v-model="resizeMethod"
                value="percentage"
                class="form-radio text-indigo-600"
              />
              <span class="ml-2">Percentage</span>
            </label>
            <label class="inline-flex items-center">
              <input
                type="radio"
                v-model="resizeMethod"
                value="preset"
                class="form-radio text-indigo-600"
              />
              <span class="ml-2">Presets</span>
            </label>
          </div>
          
          <div v-if="resizeMethod === 'dimensions'" class="space-y-4">
            <div class="flex items-center space-x-4">
              <div class="w-1/2">
                <label for="new-width" class="block text-xs text-gray-500 mb-1">Width (pixels)</label>
                <input
                  id="new-width"
                  v-model.number="newWidth"
                  type="number"
                  min="1"
                  class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                />
              </div>
              <div class="w-1/2">
                <label for="new-height" class="block text-xs text-gray-500 mb-1">Height (pixels)</label>
                <input
                  id="new-height"
                  v-model.number="newHeight"
                  type="number"
                  min="1"
                  class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                />
              </div>
            </div>
            
            <div>
              <label class="inline-flex items-center">
                <input
                  type="checkbox"
                  v-model="maintainAspectRatio"
                  class="form-checkbox text-indigo-600"
                />
                <span class="ml-2 text-sm text-gray-700">Maintain aspect ratio</span>
              </label>
            </div>
          </div>
          
          <div v-if="resizeMethod === 'percentage'" class="space-y-4">
            <label for="resize-percentage" class="block text-sm font-medium text-gray-700">
              Resize to: {{ resizePercentageText }}
            </label>
            <input
              id="resize-percentage"
              type="range"
              v-model.number="resizePercentage"
              min="1"
              max="200"
              class="block w-full"
            />
            <div class="flex justify-between text-xs text-gray-500">
              <span>1%</span>
              <span>100%</span>
              <span>200%</span>
            </div>
            <div class="text-sm text-gray-600">
              New dimensions: {{ newWidth }} × {{ newHeight }} pixels
            </div>
          </div>
          
          <div v-if="resizeMethod === 'preset'" class="space-y-4">
            <h4 class="text-sm font-medium text-gray-700">Common Size Presets:</h4>
            <div class="grid grid-cols-2 gap-2">
              <button
                v-for="preset in presetSizes"
                :key="preset.name"
                @click="applyPreset(preset)"
                class="px-3 py-2 border border-gray-300 rounded text-sm text-left hover:bg-gray-50"
              >
                <span class="font-medium">{{ preset.name }}</span>
                <span class="text-xs text-gray-500 block">{{ preset.width }}×{{ preset.height }}</span>
              </button>
            </div>
          </div>
        </div>
        
        <div class="flex justify-center">
          <button
            @click="resizeImage"
            class="px-4 py-2 bg-indigo-600 text-white rounded-md hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2"
            :disabled="isProcessing"
          >
            <template v-if="isProcessing">
              <svg class="animate-spin -ml-1 mr-2 h-4 w-4 text-white inline-block" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
              </svg>
              Resizing...
            </template>
            <template v-else>
              Resize Image
            </template>
          </button>
        </div>
        
        <div v-if="previewUrl" class="grid grid-cols-1 md:grid-cols-2 gap-6">
          <div>
            <h3 class="text-sm font-medium text-gray-700 mb-2">Original Image:</h3>
            <div class="border rounded-md overflow-hidden bg-gray-50">
              <img :src="previewUrl" class="max-w-full max-h-[300px] mx-auto" alt="Original Image Preview" />
            </div>
            <p class="text-sm text-gray-500 mt-1">{{ originalWidth }}×{{ originalHeight }} pixels</p>
          </div>
          
          <div v-if="resizedUrl">
            <h3 class="text-sm font-medium text-gray-700 mb-2">Resized Image:</h3>
            <div class="border rounded-md overflow-hidden bg-gray-50">
              <img :src="resizedUrl" class="max-w-full max-h-[300px] mx-auto" alt="Resized Image Preview" />
            </div>
            <p class="text-sm text-gray-500 mt-1">{{ newWidth }}×{{ newHeight }} pixels</p>
          </div>
        </div>
        
        <div v-if="resizedImage" class="flex justify-center">
          <button
            @click="downloadResizedImage"
            class="px-4 py-2 bg-green-600 text-white rounded-md hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-green-500 focus:ring-offset-2"
          >
            Download Resized Image
          </button>
        </div>
      </div>
    </div>
    
    <template #instructions>
      <ol class="list-decimal list-inside space-y-2 text-gray-600">
        <li>Click "Select an image" to upload your image file</li>
        <li>Choose your resize method: Custom Dimensions, Percentage, or Preset sizes</li>
        <li>Set your desired dimensions or select a common preset size</li>
        <li>Click "Resize Image" to process your image</li>
        <li>Preview both the original and resized versions side by side</li>
        <li>Click "Download Resized Image" to save the resized file</li>
      </ol>
    </template>
  </ToolContainer>
</template>
