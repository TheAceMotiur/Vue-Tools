<script setup>
import { ref, computed } from 'vue';
import ToolContainer from '../../components/ToolContainer.vue';

const selectedFile = ref(null);
const compressedImage = ref(null);
const quality = ref(80);
const originalSize = ref(0);
const compressedSize = ref(0);
const isProcessing = ref(false);
const error = ref('');
const previewUrl = ref('');
const compressedUrl = ref('');

const compressionRatio = computed(() => {
  if (!originalSize.value || !compressedSize.value) return 0;
  return Math.round((1 - (compressedSize.value / originalSize.value)) * 100);
});

function handleFileChange(event) {
  const file = event.target.files[0];
  if (!file) return;
  
  // Check if file is an image
  if (!file.type.match('image.*')) {
    error.value = 'Please select an image file (JPEG, PNG, WebP)';
    return;
  }
  
  // Reset values
  error.value = '';
  selectedFile.value = file;
  originalSize.value = file.size;
  compressedImage.value = null;
  compressedSize.value = 0;
  compressedUrl.value = '';
  
  // Create preview
  const reader = new FileReader();
  reader.onload = (e) => {
    previewUrl.value = e.target.result;
  };
  reader.readAsDataURL(file);
}

function compressImage() {
  if (!selectedFile.value) {
    error.value = 'Please select an image file first';
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
    canvas.width = img.width;
    canvas.height = img.height;
    
    // Draw image on canvas
    ctx.drawImage(img, 0, 0);
    
    // Convert to blob and get compressed image
    canvas.toBlob(
      (blob) => {
        if (blob) {
          compressedImage.value = new File([blob], selectedFile.value.name, {
            type: blob.type,
          });
          compressedSize.value = blob.size;
          compressedUrl.value = URL.createObjectURL(blob);
        } else {
          error.value = 'Failed to compress the image';
        }
        isProcessing.value = false;
      },
      selectedFile.value.type,
      quality.value / 100
    );
  };
  
  img.onerror = () => {
    error.value = 'Error loading image';
    isProcessing.value = false;
  };
  
  img.src = previewUrl.value;
}

function downloadCompressedImage() {
  if (!compressedImage.value) return;
  
  const link = document.createElement('a');
  link.href = compressedUrl.value;
  link.download = `compressed-${compressedImage.value.name}`;
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
}

function formatFileSize(bytes) {
  if (bytes === 0) return '0 Bytes';
  const k = 1024;
  const sizes = ['Bytes', 'KB', 'MB'];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
}
</script>

<template>
  <ToolContainer 
    title="Image Compressor" 
    description="Reduce image file size while preserving quality for faster website loading."
  >
    <div class="space-y-6">
      <div>
        <label for="image-file" class="block text-sm font-medium text-gray-700 mb-2">
          Select an image to compress:
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
          <label for="quality" class="block text-sm font-medium text-gray-700 mb-2">
            Compression Quality: {{ quality }}%
          </label>
          <input
            id="quality"
            type="range"
            v-model.number="quality"
            min="1"
            max="100"
            class="block w-full"
          />
          <div class="flex justify-between text-xs text-gray-500 mt-1">
            <span>Smaller file</span>
            <span>Better quality</span>
          </div>
        </div>
        
        <div class="flex justify-center">
          <button
            @click="compressImage"
            class="px-4 py-2 bg-indigo-600 text-white rounded-md hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2"
            :disabled="isProcessing"
          >
            <template v-if="isProcessing">
              <svg class="animate-spin -ml-1 mr-2 h-4 w-4 text-white inline-block" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
              </svg>
              Compressing...
            </template>
            <template v-else>
              Compress Image
            </template>
          </button>
        </div>
        
        <div v-if="previewUrl" class="grid grid-cols-1 md:grid-cols-2 gap-6">
          <div>
            <h3 class="text-sm font-medium text-gray-700 mb-2">Original Image:</h3>
            <div class="border rounded-md overflow-hidden bg-gray-50">
              <img :src="previewUrl" class="max-w-full max-h-[300px] mx-auto" alt="Original Image Preview" />
            </div>
            <p class="text-sm text-gray-500 mt-1">Size: {{ formatFileSize(originalSize) }}</p>
          </div>
          
          <div v-if="compressedUrl">
            <h3 class="text-sm font-medium text-gray-700 mb-2">Compressed Image:</h3>
            <div class="border rounded-md overflow-hidden bg-gray-50">
              <img :src="compressedUrl" class="max-w-full max-h-[300px] mx-auto" alt="Compressed Image Preview" />
            </div>
            <p class="text-sm text-gray-500 mt-1">
              Size: {{ formatFileSize(compressedSize) }} 
              <span class="text-green-600 font-medium ml-1">
                ({{ compressionRatio }}% smaller)
              </span>
            </p>
          </div>
        </div>
        
        <div v-if="compressedImage" class="flex justify-center">
          <button
            @click="downloadCompressedImage"
            class="px-4 py-2 bg-green-600 text-white rounded-md hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-green-500 focus:ring-offset-2"
          >
            Download Compressed Image
          </button>
        </div>
      </div>
    </div>
    
    <template #instructions>
      <ol class="list-decimal list-inside space-y-2 text-gray-600">
        <li>Click "Select an image" to upload your image file</li>
        <li>Use the quality slider to adjust the compression level (lower quality = smaller file size)</li>
        <li>Click "Compress Image" to process your image</li>
        <li>Preview both the original and compressed versions side by side</li>
        <li>Click "Download Compressed Image" to save the optimized file</li>
      </ol>
    </template>
  </ToolContainer>
</template>
