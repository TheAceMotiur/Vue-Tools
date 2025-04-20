<script setup>
import { ref } from 'vue';
import ToolContainer from '../../components/ToolContainer.vue';

const imageFile = ref(null);
const previewUrl = ref('');
const convertedImage = ref(null);
const convertedUrl = ref('');
const originalFormat = ref('');
const targetFormat = ref('webp');
const quality = ref(80);
const error = ref('');

function handleFileUpload(event) {
  const file = event.target.files[0];
  if (!file) return;

  imageFile.value = file;
  originalFormat.value = file.type.split('/')[1];
  previewUrl.value = URL.createObjectURL(file);
}

function convertImage() {
  if (!imageFile.value) {
    error.value = 'Please upload an image file first.';
    return;
  }

  error.value = '';

  const canvas = document.createElement('canvas');
  const ctx = canvas.getContext('2d');
  const img = new Image();

  img.onload = () => {
    canvas.width = img.width;
    canvas.height = img.height;
    ctx.drawImage(img, 0, 0);

    const mimeType = targetFormat.value === 'jpeg' ? 'image/jpeg' : `image/${targetFormat.value}`;
    const convertedDataUrl = canvas.toDataURL(mimeType, quality.value / 100);

    convertedImage.value = convertedDataUrl;
    convertedUrl.value = convertedDataUrl;
  };

  img.onerror = () => {
    error.value = 'Error converting image. Please try again.';
  };

  img.src = previewUrl.value;
}

function downloadConvertedImage() {
  if (!convertedImage.value) return;

  const link = document.createElement('a');
  link.href = convertedImage.value;
  link.download = `converted-image.${targetFormat.value}`;
  link.click();
}
</script>

<template>
  <ToolContainer 
    title="Image Converter" 
    description="Convert images between different formats (JPG, PNG, WebP)."
  >
    <div class="space-y-6">
      <div>
        <label for="image-upload" class="block text-sm font-medium text-gray-700 mb-2">
          Select an image:
        </label>
        <input
          id="image-upload"
          type="file"
          accept="image/*"
          @change="handleFileUpload"
          class="block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-md file:border file:border-gray-300 file:text-sm file:font-medium file:bg-gray-50 file:text-gray-700 hover:file:bg-gray-100"
        />
      </div>

      <div>
        <label for="format-select" class="block text-sm font-medium text-gray-700 mb-2">
          Convert to format:
        </label>
        <select
          id="format-select"
          v-model="targetFormat"
          class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
        >
          <option value="webp">WebP</option>
          <option value="jpeg">JPEG</option>
          <option value="png">PNG</option>
        </select>
      </div>

      <div v-if="targetFormat === 'jpeg' || targetFormat === 'webp'">
        <label for="quality-range" class="block text-sm font-medium text-gray-700 mb-2">
          Quality ({{ quality }}):
        </label>
        <input
          id="quality-range"
          type="range"
          v-model="quality"
          min="1"
          max="100"
          class="w-full"
        />
      </div>

      <div class="flex justify-center">
        <button
          @click="convertImage"
          class="px-4 py-2 bg-indigo-600 text-white rounded-md hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2"
        >
          Convert Image
        </button>
      </div>

      <div v-if="error" class="bg-red-50 border-l-4 border-red-400 p-4 text-red-700">
        {{ error }}
      </div>

      <div v-if="previewUrl" class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div>
          <h3 class="text-sm font-medium text-gray-700 mb-2">Original Image ({{ originalFormat }}):</h3>
          <div class="border rounded-md overflow-hidden bg-gray-50">
            <img :src="previewUrl" class="max-w-full max-h-[300px] mx-auto" alt="Original Image Preview" />
          </div>
        </div>
        
        <div v-if="convertedUrl">
          <h3 class="text-sm font-medium text-gray-700 mb-2">Converted Image ({{ targetFormat.toUpperCase() }}):</h3>
          <div class="border rounded-md overflow-hidden bg-gray-50">
            <img :src="convertedUrl" class="max-w-full max-h-[300px] mx-auto" alt="Converted Image Preview" />
          </div>
        </div>
      </div>
      
      <div v-if="convertedImage" class="flex justify-center">
        <button
          @click="downloadConvertedImage"
          class="px-4 py-2 bg-green-600 text-white rounded-md hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-green-500 focus:ring-offset-2"
        >
          Download {{ targetFormat.toUpperCase() }} Image
        </button>
      </div>
    </div>
    
    <template #instructions>
      <ol class="list-decimal list-inside space-y-2 text-gray-600">
        <li>Click "Select an image" to upload your image file</li>
        <li>Choose the format you want to convert to (WebP, JPEG, or PNG)</li>
        <li>For JPEG and WebP, you can adjust the quality (higher quality = larger file)</li>
        <li>Click "Convert Image" to process your image</li>
        <li>Preview both the original and converted versions side by side</li>
        <li>Click "Download" to save the converted image to your device</li>
      </ol>
    </template>
  </ToolContainer>
</template>