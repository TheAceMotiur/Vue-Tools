<script setup>
import { ref, computed } from 'vue';
import ToolContainer from '../../components/ToolContainer.vue';

const inputText = ref('');
const selectedCase = ref('uppercase');

const convertedText = computed(() => {
  if (!inputText.value) return '';
  
  switch (selectedCase.value) {
    case 'uppercase':
      return inputText.value.toUpperCase();
    case 'lowercase':
      return inputText.value.toLowerCase();
    case 'capitalize':
      return inputText.value
        .split(' ')
        .map(word => word.charAt(0).toUpperCase() + word.slice(1).toLowerCase())
        .join(' ');
    case 'sentence':
      return inputText.value.charAt(0).toUpperCase() + inputText.value.slice(1).toLowerCase();
    case 'alternating':
      return inputText.value
        .split('')
        .map((char, index) => index % 2 === 0 ? char.toUpperCase() : char.toLowerCase())
        .join('');
    default:
      return inputText.value;
  }
});

function copyToClipboard() {
  navigator.clipboard.writeText(convertedText.value);
  alert('Copied to clipboard!');
}
</script>

<template>
  <ToolContainer 
    title="Text Case Converter" 
    description="Convert your text to different case formats with this simple tool."
  >
    <div class="space-y-6">
      <div>
        <label for="input-text" class="block text-sm font-medium text-gray-700 mb-2">
          Enter your text:
        </label>
        <textarea
          id="input-text"
          v-model="inputText"
          rows="5"
          class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
          placeholder="Type or paste your text here..."
        ></textarea>
      </div>
      
      <div>
        <label class="block text-sm font-medium text-gray-700 mb-2">
          Select case transformation:
        </label>
        <div class="flex flex-wrap gap-3">
          <label class="inline-flex items-center">
            <input type="radio" v-model="selectedCase" value="uppercase" class="form-radio text-indigo-600">
            <span class="ml-2">UPPERCASE</span>
          </label>
          <label class="inline-flex items-center">
            <input type="radio" v-model="selectedCase" value="lowercase" class="form-radio text-indigo-600">
            <span class="ml-2">lowercase</span>
          </label>
          <label class="inline-flex items-center">
            <input type="radio" v-model="selectedCase" value="capitalize" class="form-radio text-indigo-600">
            <span class="ml-2">Title Case</span>
          </label>
          <label class="inline-flex items-center">
            <input type="radio" v-model="selectedCase" value="sentence" class="form-radio text-indigo-600">
            <span class="ml-2">Sentence case</span>
          </label>
          <label class="inline-flex items-center">
            <input type="radio" v-model="selectedCase" value="alternating" class="form-radio text-indigo-600">
            <span class="ml-2">AlTeRnAtInG cAsE</span>
          </label>
        </div>
      </div>
      
      <div>
        <label for="output-text" class="block text-sm font-medium text-gray-700 mb-2">
          Result:
        </label>
        <div class="relative">
          <textarea
            id="output-text"
            readonly
            rows="5"
            class="block w-full rounded-md border-gray-300 bg-gray-50 shadow-sm"
            :value="convertedText"
          ></textarea>
          <button
            v-if="convertedText"
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
        <li>Enter or paste your text in the input field</li>
        <li>Select your desired case transformation</li>
        <li>The converted text will appear in the result field</li>
        <li>Click the copy icon to copy the result to your clipboard</li>
      </ol>
    </template>
  </ToolContainer>
</template>
