<script setup>
import { ref, computed } from 'vue';
import ToolContainer from '../../components/ToolContainer.vue';

const selectedColor = ref('#4f46e5'); // Default indigo color
const colorFormat = ref('hex');

const colorFormats = computed(() => {
  const hex = selectedColor.value;
  
  // Convert hex to RGB
  const r = parseInt(hex.substring(1, 3), 16);
  const g = parseInt(hex.substring(3, 5), 16);
  const b = parseInt(hex.substring(5, 7), 16);
  
  // Convert RGB to HSL
  const r1 = r / 255;
  const g1 = g / 255;
  const b1 = b / 255;
  
  const max = Math.max(r1, g1, b1);
  const min = Math.min(r1, g1, b1);
  let h, s, l = (max + min) / 2;
  
  if (max === min) {
    h = s = 0; // achromatic
  } else {
    const d = max - min;
    s = l > 0.5 ? d / (2 - max - min) : d / (max + min);
    
    switch (max) {
      case r1: h = (g1 - b1) / d + (g1 < b1 ? 6 : 0); break;
      case g1: h = (b1 - r1) / d + 2; break;
      case b1: h = (r1 - g1) / d + 4; break;
    }
    
    h = Math.round(h * 60);
  }
  
  s = Math.round(s * 100);
  l = Math.round(l * 100);
  
  return {
    hex: hex,
    rgb: `rgb(${r}, ${g}, ${b})`,
    hsl: `hsl(${h}, ${s}%, ${l}%)`
  };
});

function copyColorCode() {
  let colorCode = '';
  
  switch (colorFormat.value) {
    case 'hex':
      colorCode = colorFormats.value.hex;
      break;
    case 'rgb':
      colorCode = colorFormats.value.rgb;
      break;
    case 'hsl':
      colorCode = colorFormats.value.hsl;
      break;
  }
  
  navigator.clipboard.writeText(colorCode);
  alert(`Copied ${colorCode} to clipboard!`);
}
</script>

<template>
  <ToolContainer 
    title="Color Picker" 
    description="Select colors and get their values in different formats"
  >
    <div class="space-y-6">
      <div class="flex flex-col sm:flex-row gap-6">
        <div class="flex-1">
          <label for="color-picker" class="block text-sm font-medium text-gray-700 mb-2">
            Pick a color:
          </label>
          <input 
            id="color-picker"
            type="color"
            v-model="selectedColor"
            class="h-40 w-full p-1 border border-gray-300 rounded-md cursor-pointer"
          />
        </div>
        
        <div class="flex-1">
          <label class="block text-sm font-medium text-gray-700 mb-2">
            Selected color preview:
          </label>
          <div 
            class="h-40 rounded-md border border-gray-300"
            :style="{ backgroundColor: selectedColor }"
          ></div>
        </div>
      </div>
      
      <div>
        <label class="block text-sm font-medium text-gray-700 mb-2">
          Color format:
        </label>
        <div class="flex flex-wrap gap-3">
          <label class="inline-flex items-center">
            <input type="radio" v-model="colorFormat" value="hex" class="form-radio text-indigo-600">
            <span class="ml-2">HEX</span>
          </label>
          <label class="inline-flex items-center">
            <input type="radio" v-model="colorFormat" value="rgb" class="form-radio text-indigo-600">
            <span class="ml-2">RGB</span>
          </label>
          <label class="inline-flex items-center">
            <input type="radio" v-model="colorFormat" value="hsl" class="form-radio text-indigo-600">
            <span class="ml-2">HSL</span>
          </label>
        </div>
      </div>
      
      <div>
        <label class="block text-sm font-medium text-gray-700 mb-2">
          Color value:
        </label>
        <div class="relative">
          <input
            type="text"
            readonly
            class="block w-full rounded-md border-gray-300 bg-gray-50 shadow-sm py-2 px-3"
            :value="colorFormat === 'hex' ? colorFormats.hex : (colorFormat === 'rgb' ? colorFormats.rgb : colorFormats.hsl)"
          />
          <button
            @click="copyColorCode"
            class="absolute right-2 top-1/2 -translate-y-1/2 p-2 rounded-md hover:bg-gray-200"
            title="Copy to clipboard"
          >
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
              <path d="M8 3a1 1 0 011-1h2a1 1 0 110 2H9a1 1 0 01-1-1z" />
              <path d="M6 3a2 2 0 00-2 2v11a2 2 0 002 2h8a2 2 0 002-2V5a2 2 0 00-2-2 3 3 0 01-3 3H9a3 3 0 01-3-3z" />
            </svg>
          </button>
        </div>
      </div>
      
      <div>
        <h3 class="text-lg font-medium text-gray-800 mb-2">All formats:</h3>
        <ul class="space-y-2 text-gray-700">
          <li><span class="font-medium">HEX:</span> {{ colorFormats.hex }}</li>
          <li><span class="font-medium">RGB:</span> {{ colorFormats.rgb }}</li>
          <li><span class="font-medium">HSL:</span> {{ colorFormats.hsl }}</li>
        </ul>
      </div>
    </div>
    
    <template #instructions>
      <ol class="list-decimal list-inside space-y-2 text-gray-600">
        <li>Click on the color picker to select a color</li>
        <li>View the selected color in the preview panel</li>
        <li>Choose your preferred format (HEX, RGB, or HSL)</li>
        <li>Click the copy button to copy the color value to your clipboard</li>
        <li>All color formats are displayed at the bottom for reference</li>
      </ol>
    </template>
  </ToolContainer>
</template>
