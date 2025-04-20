<script setup>
import { ref, computed, watch } from 'vue';
import ToolContainer from '../../components/ToolContainer.vue';

const text = ref('');
const showSpaces = ref(false);
const showLineNumbers = ref(false);

const stats = computed(() => {
  const input = text.value;
  
  // Count characters
  const charCount = input.length;
  
  // Count characters excluding spaces
  const charNoSpacesCount = input.replace(/\s/g, '').length;
  
  // Count words
  const words = input.trim() ? input.trim().split(/\s+/) : [];
  const wordCount = words.length;
  
  // Count lines
  const lines = input.split(/\r\n|\r|\n/);
  const lineCount = input ? lines.length : 0;
  
  // Count sentences (basic approximation)
  const sentenceCount = input.split(/[.!?]+/).filter(sentence => sentence.trim().length > 0).length;
  
  // Count paragraphs (groups of text separated by at least one empty line)
  const paragraphCount = input.split(/\n{2,}/).filter(para => para.trim().length > 0).length || 0;
  
  // Calculate reading time (average 225 words per minute)
  const readingTimeMinutes = wordCount / 225;
  const readingTimeFormatted = readingTimeMinutes < 1 
    ? `${Math.ceil(readingTimeMinutes * 60)} seconds` 
    : `${Math.floor(readingTimeMinutes)} minute${Math.floor(readingTimeMinutes) !== 1 ? 's' : ''}`;
    
  return {
    charCount,
    charNoSpacesCount,
    wordCount,
    lineCount,
    sentenceCount,
    paragraphCount,
    readingTimeFormatted
  };
});

const formattedText = computed(() => {
  if (!text.value) return '';
  
  let formatted = text.value;
  
  if (showSpaces.value) {
    // Replace spaces with visible character (·)
    formatted = formatted.replace(/ /g, '·');
    // Replace tabs with visible character (→)
    formatted = formatted.replace(/\t/g, '→');
  }
  
  if (showLineNumbers.value) {
    const lines = formatted.split('\n');
    formatted = lines.map((line, index) => `${index + 1}. ${line}`).join('\n');
  }
  
  return formatted;
});

function copyStats() {
  const statsText = `Word count: ${stats.value.wordCount}
Character count: ${stats.value.charCount}
Character count (no spaces): ${stats.value.charNoSpacesCount}
Line count: ${stats.value.lineCount}
Paragraph count: ${stats.value.paragraphCount}
Sentence count: ${stats.value.sentenceCount}
Estimated reading time: ${stats.value.readingTimeFormatted}`;

  navigator.clipboard.writeText(statsText);
  alert('Statistics copied to clipboard!');
}
</script>

<template>
  <ToolContainer 
    title="Word Counter" 
    description="Count words, characters, lines, and more in your text."
  >
    <div class="space-y-6">
      <div>
        <label for="text-input" class="block text-sm font-medium text-gray-700 mb-2">
          Enter or paste your text:
        </label>
        <textarea
          id="text-input"
          v-model="text"
          rows="12"
          class="block w-full font-mono rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
          placeholder="Type or paste your text here..."
        ></textarea>
      </div>
      
      <div class="bg-indigo-50 p-4 rounded-md shadow-sm">
        <div class="flex justify-between items-center mb-4">
          <h3 class="text-lg font-medium text-indigo-900">Text Statistics</h3>
          <button
            @click="copyStats"
            class="text-xs px-2 py-1 bg-indigo-100 hover:bg-indigo-200 text-indigo-700 rounded"
            title="Copy statistics"
          >
            Copy Stats
          </button>
        </div>
        
        <div class="grid grid-cols-2 sm:grid-cols-3 gap-4">
          <div>
            <p class="text-xs text-indigo-700 mb-1">Words</p>
            <p class="text-2xl font-bold text-indigo-900">{{ stats.wordCount }}</p>
          </div>
          
          <div>
            <p class="text-xs text-indigo-700 mb-1">Characters</p>
            <p class="text-2xl font-bold text-indigo-900">{{ stats.charCount }}</p>
          </div>
          
          <div>
            <p class="text-xs text-indigo-700 mb-1">Characters (no spaces)</p>
            <p class="text-2xl font-bold text-indigo-900">{{ stats.charNoSpacesCount }}</p>
          </div>
          
          <div>
            <p class="text-xs text-indigo-700 mb-1">Lines</p>
            <p class="text-2xl font-bold text-indigo-900">{{ stats.lineCount }}</p>
          </div>
          
          <div>
            <p class="text-xs text-indigo-700 mb-1">Paragraphs</p>
            <p class="text-2xl font-bold text-indigo-900">{{ stats.paragraphCount }}</p>
          </div>
          
          <div>
            <p class="text-xs text-indigo-700 mb-1">Sentences</p>
            <p class="text-2xl font-bold text-indigo-900">{{ stats.sentenceCount }}</p>
          </div>
        </div>
        
        <div class="mt-3 pt-3 border-t border-indigo-200">
          <p class="text-sm text-indigo-800">
            <span class="font-medium">Estimated reading time:</span> {{ stats.readingTimeFormatted }}
          </p>
        </div>
      </div>
      
      <div class="flex space-x-4">
        <label class="inline-flex items-center">
          <input type="checkbox" v-model="showSpaces" class="form-checkbox text-indigo-600">
          <span class="ml-2 text-sm text-gray-700">Show spaces and tabs</span>
        </label>
        
        <label class="inline-flex items-center">
          <input type="checkbox" v-model="showLineNumbers" class="form-checkbox text-indigo-600">
          <span class="ml-2 text-sm text-gray-700">Show line numbers</span>
        </label>
      </div>
      
      <div v-if="formattedText && (showSpaces || showLineNumbers)" class="border rounded p-4 bg-gray-50 font-mono text-sm whitespace-pre-wrap">
        {{ formattedText }}
      </div>
    </div>
    
    <template #instructions>
      <ol class="list-decimal list-inside space-y-2 text-gray-600">
        <li>Enter or paste your text into the input field</li>
        <li>View the statistics about your text (words, characters, etc.)</li>
        <li>Toggle "Show spaces and tabs" to visualize whitespace</li>
        <li>Toggle "Show line numbers" to display line numbers</li>
        <li>Click "Copy Stats" to copy the statistics to your clipboard</li>
      </ol>
    </template>
  </ToolContainer>
</template>
