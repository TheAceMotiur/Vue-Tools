<script setup>
import { ref } from 'vue';
import ToolContainer from '../../components/ToolContainer.vue';

const cssInput = ref('');
const minifiedCss = ref('');
const originalSize = ref(0);
const minifiedSize = ref(0);
const savedPercentage = ref(0);
const error = ref('');

function minifyCss() {
  try {
    error.value = '';
    
    if (!cssInput.value.trim()) {
      minifiedCss.value = '';
      originalSize.value = 0;
      minifiedSize.value = 0;
      savedPercentage.value = 0;
      return;
    }
    
    // Store original size
    originalSize.value = new Blob([cssInput.value]).size;
    
    // Simple CSS minification rules
    let result = cssInput.value
      // Remove comments
      .replace(/\/\*[\s\S]*?\*\//g, '')
      // Remove whitespace before and after brackets
      .replace(/\s*{\s*/g, '{')
      .replace(/\s*}\s*/g, '}')
      // Remove whitespace before and after colons and semicolons
      .replace(/\s*:\s*/g, ':')
      .replace(/\s*;\s*/g, ';')
      // Remove last semicolons before closing brackets
      .replace(/;\}/g, '}')
      // Remove multiple whitespaces
      .replace(/\s+/g, ' ')
      // Trim leading and trailing whitespace
      .trim();
      
    minifiedCss.value = result;
    
    // Calculate size reduction
    minifiedSize.value = new Blob([result]).size;
    const reduction = originalSize.value - minifiedSize.value;
    savedPercentage.value = originalSize.value > 0 
      ? Math.round((reduction / originalSize.value) * 100) 
      : 0;
      
  } catch (err) {
    error.value = 'Error minifying CSS: ' + err.message;
    console.error(err);
  }
}

function copyToClipboard() {
  navigator.clipboard.writeText(minifiedCss.value);
  alert('Minified CSS copied to clipboard!');
}
</script>

<template>
  <ToolContainer 
    title="CSS Minifier" 
    description="Minify your CSS code to reduce file size and improve load times."
  >
    <div class="space-y-6">
      <div>
        <label for="css-input" class="block text-sm font-medium text-gray-700 mb-2">
          Paste your CSS code here:
        </label>
        <textarea
          id="css-input"
          v-model="cssInput"
          rows="12"
          class="block w-full font-mono rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
          placeholder="/* Your CSS code here */
body {
    font-family: Arial, sans-serif;
    font-size: 16px;
    line-height: 1.5;
    color: #333333;
    background-color: #ffffff;
}"
        ></textarea>
      </div>
      
      <div class="flex justify-center">
        <button
          @click="minifyCss"
          class="px-4 py-2 bg-indigo-600 text-white rounded-md hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2"
        >
          Minify CSS
        </button>
      </div>
      
      <div v-if="error" class="bg-red-50 border-l-4 border-red-400 p-4 text-red-700">
        {{ error }}
      </div>
      
      <div v-if="minifiedCss">
        <div class="flex justify-between items-center mb-2">
          <label for="minified-css" class="block text-sm font-medium text-gray-700">
            Minified CSS:
          </label>
          
          <div class="flex space-x-2 text-xs text-gray-500">
            <span>Original: {{ originalSize }} bytes</span>
            <span>Minified: {{ minifiedSize }} bytes</span>
            <span class="font-medium text-green-600">Saved: {{ savedPercentage }}%</span>
          </div>
        </div>
        
        <div class="relative">
          <textarea
            id="minified-css"
            readonly
            rows="6"
            class="block w-full font-mono rounded-md border-gray-300 bg-gray-50 shadow-sm text-sm"
            :value="minifiedCss"
          ></textarea>
          <button
            @click="copyToClipboard"
            class="absolute right-2 top-2 p-2 rounded-md hover:bg-gray-200"
            title="Copy to clipboard"
          >
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
              <path d="M8 3a1 1 0 011-1h2a1 1 0 110 2H9a1 1 0 01-1-1z" />
              <path d="M6 3a2 2 0 00-2 2v11a2 2 0 002 2h8a2 2 0 002-2V5a2 2 0 00-2-2 3 3 0 01-3 3H9a3 3 0 01-3-3z" />
            </svg>
          </button>
        </div>
      </div>
    </div>
    
    <template #instructions>
      <ol class="list-decimal list-inside space-y-2 text-gray-600">
        <li>Paste your CSS code into the input field</li>
        <li>Click the "Minify CSS" button</li>
        <li>The minified CSS will appear in the result field</li>
        <li>Click the copy button to copy the minified CSS to your clipboard</li>
        <li>Check the size statistics to see how much space you've saved</li>
      </ol>
    </template>
  </ToolContainer>
</template>
