<script setup>
import { ref } from 'vue';
import ToolContainer from '../../components/ToolContainer.vue';

const htmlInput = ref('');
const formattedHtml = ref('');
const indentSize = ref(2);
const error = ref('');

function formatHtml() {
  try {
    error.value = '';
    if (!htmlInput.value.trim()) {
      formattedHtml.value = '';
      return;
    }
    
    // Simple HTML formatting algorithm
    let formatted = '';
    let indent = 0;
    const lines = htmlInput.value
      .replace(/>\s*</g, '>\n<') // Add new line between tags
      .split('\n');
    
    for (let line of lines) {
      line = line.trim();
      if (!line) continue;
      
      // Check if this line is a closing tag
      if (line.match(/^<\/[^>]+>$/)) {
        indent -= indentSize.value;
      }
      
      // Add the line with proper indentation
      formatted += ' '.repeat(Math.max(0, indent)) + line + '\n';
      
      // Check if this line is an opening tag (but not self-closing)
      if (line.match(/^<[^\/][^>]*>$/) && !line.match(/\/\s*>$/)) {
        indent += indentSize.value;
      }
    }
    
    formattedHtml.value = formatted;
  } catch (err) {
    error.value = 'Error formatting HTML: ' + err.message;
    console.error(err);
  }
}

function copyToClipboard() {
  navigator.clipboard.writeText(formattedHtml.value);
  alert('Copied to clipboard!');
}
</script>

<template>
  <ToolContainer 
    title="HTML Formatter" 
    description="Format and beautify your HTML code for better readability."
  >
    <div class="space-y-6">
      <div>
        <label for="indent-size" class="block text-sm font-medium text-gray-700 mb-2">
          Indent Size:
        </label>
        <select
          id="indent-size"
          v-model="indentSize"
          class="rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
        >
          <option :value="2">2 spaces</option>
          <option :value="4">4 spaces</option>
          <option :value="8">8 spaces</option>
        </select>
      </div>
      
      <div>
        <label for="html-input" class="block text-sm font-medium text-gray-700 mb-2">
          Input HTML:
        </label>
        <textarea
          id="html-input"
          v-model="htmlInput"
          rows="8"
          class="block w-full font-mono rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
          placeholder="Paste your HTML code here..."
        ></textarea>
      </div>
      
      <div class="flex justify-center">
        <button
          @click="formatHtml"
          class="px-4 py-2 bg-indigo-600 text-white rounded-md hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2"
        >
          Format HTML
        </button>
      </div>
      
      <div v-if="error" class="bg-red-50 border-l-4 border-red-400 p-4 text-red-700">
        {{ error }}
      </div>
      
      <div v-if="formattedHtml">
        <label for="formatted-html" class="block text-sm font-medium text-gray-700 mb-2">
          Formatted HTML:
        </label>
        <div class="relative">
          <textarea
            id="formatted-html"
            readonly
            rows="12"
            class="block w-full font-mono rounded-md border-gray-300 bg-gray-50 shadow-sm"
            :value="formattedHtml"
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
        <li>Select your preferred indentation size</li>
        <li>Paste your HTML code into the input field</li>
        <li>Click the "Format HTML" button</li>
        <li>The formatted HTML will appear in the result field</li>
        <li>Click the copy icon to copy the formatted code to your clipboard</li>
      </ol>
    </template>
  </ToolContainer>
</template>
