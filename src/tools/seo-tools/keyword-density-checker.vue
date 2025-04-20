<script setup>
import { ref, computed } from 'vue';
import ToolContainer from '../../components/ToolContainer.vue';

const content = ref('');
const excludeCommonWords = ref(true);
const minWordLength = ref(3);
const showCount = ref(10);
const analyzedResult = ref(null);
const isLoading = ref(false);

// Common words to exclude (stop words)
const commonWords = [
  'the', 'be', 'to', 'of', 'and', 'a', 'in', 'that', 'have', 'i',
  'it', 'for', 'not', 'on', 'with', 'he', 'as', 'you', 'do', 'at',
  'this', 'but', 'his', 'by', 'from', 'they', 'we', 'say', 'her', 'she',
  'or', 'an', 'will', 'my', 'one', 'all', 'would', 'there', 'their', 'what',
  'so', 'up', 'out', 'if', 'about', 'who', 'get', 'which', 'go', 'me',
  'when', 'make', 'can', 'like', 'time', 'no', 'just', 'him', 'know', 'take',
  'people', 'into', 'year', 'your', 'good', 'some', 'could', 'them', 'see', 'other',
  'than', 'then', 'now', 'look', 'only', 'come', 'its', 'over', 'think', 'also'
];

const wordCount = computed(() => {
  if (!content.value) return 0;
  return content.value.trim().split(/\s+/).length;
});

function analyzeContent() {
  if (!content.value.trim()) {
    return;
  }
  
  isLoading.value = true;
  
  setTimeout(() => {
    const text = content.value.toLowerCase();
    
    // Remove HTML tags
    const cleanText = text.replace(/<[^>]*>/g, ' ');
    
    // Extract words
    const words = cleanText.split(/[\s.,\/#!$%\^&\*;:{}=\-_`~()]+/);
    
    // Count total valid words
    const totalWords = words.filter(word => word.trim().length > 0).length;
    
    // Count word frequencies
    const wordFreq = {};
    
    words.forEach(word => {
      word = word.trim();
      
      // Skip empty words
      if (!word) return;
      
      // Skip short words if needed
      if (word.length < minWordLength.value) return;
      
      // Skip common words if option is selected
      if (excludeCommonWords.value && commonWords.includes(word)) return;
      
      wordFreq[word] = (wordFreq[word] || 0) + 1;
    });
    
    // Convert to array and sort by frequency
    const sortedWords = Object.entries(wordFreq)
      .sort((a, b) => b[1] - a[1])
      .map(([word, count]) => ({
        word,
        count,
        density: (count / totalWords * 100).toFixed(2)
      }))
      .slice(0, showCount.value);
    
    // Find keyword phrases (2 words)
    const phrases = {};
    for (let i = 0; i < words.length - 1; i++) {
      const word1 = words[i].trim();
      const word2 = words[i + 1].trim();
      
      if (!word1 || !word2) continue;
      if (word1.length < minWordLength.value || word2.length < minWordLength.value) continue;
      if (excludeCommonWords.value && (commonWords.includes(word1) || commonWords.includes(word2))) continue;
      
      const phrase = `${word1} ${word2}`;
      phrases[phrase] = (phrases[phrase] || 0) + 1;
    }
    
    const sortedPhrases = Object.entries(phrases)
      .sort((a, b) => b[1] - a[1])
      .map(([phrase, count]) => ({
        phrase,
        count,
        density: (count / (totalWords - 1) * 100).toFixed(2)
      }))
      .slice(0, showCount.value);
    
    analyzedResult.value = {
      totalWords,
      words: sortedWords,
      phrases: sortedPhrases
    };
    
    isLoading.value = false;
  }, 500); // Simulate processing time
}

function getDensityColor(density) {
  const densityNum = parseFloat(density);
  if (densityNum > 5) return 'text-red-600'; // Over-optimized
  if (densityNum > 2.5) return 'text-green-600'; // Good
  return 'text-gray-600'; // Normal
}

function clearResults() {
  content.value = '';
  analyzedResult.value = null;
}
</script>

<template>
  <ToolContainer 
    title="Keyword Density Checker" 
    description="Analyze your content to find keyword density and optimize for SEO."
  >
    <div class="space-y-6">
      <div>
        <label for="content" class="block text-sm font-medium text-gray-700 mb-2">
          Paste your content to analyze:
        </label>
        <textarea
          id="content"
          v-model="content"
          rows="10"
          class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
          placeholder="Paste your article, blog post, or website content here..."
        ></textarea>
        <p class="mt-1 text-sm text-gray-500">Word count: {{ wordCount }}</p>
      </div>
      
      <div class="flex flex-col sm:flex-row space-y-2 sm:space-y-0 sm:space-x-4">
        <label class="inline-flex items-center">
          <input type="checkbox" v-model="excludeCommonWords" class="form-checkbox text-indigo-600">
          <span class="ml-2 text-sm text-gray-700">Exclude common words</span>
        </label>
        
        <div class="flex items-center space-x-2">
          <label for="min-word-length" class="text-sm text-gray-700">Min word length:</label>
          <input
            id="min-word-length"
            v-model.number="minWordLength"
            type="number"
            min="1"
            max="10"
            class="w-16 rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
          />
        </div>
        
        <div class="flex items-center space-x-2">
          <label for="show-count" class="text-sm text-gray-700">Show top:</label>
          <input
            id="show-count"
            v-model.number="showCount"
            type="number"
            min="5"
            max="50"
            class="w-16 rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
          />
        </div>
      </div>
      
      <div class="flex space-x-4">
        <button
          @click="analyzeContent"
          class="px-4 py-2 bg-indigo-600 text-white rounded-md hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2"
          :disabled="isLoading || !content.trim()"
        >
          <template v-if="isLoading">
            <svg class="animate-spin -ml-1 mr-2 h-4 w-4 text-white inline-block" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
              <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
            </svg>
            Analyzing...
          </template>
          <template v-else>
            Analyze Content
          </template>
        </button>
        
        <button
          @click="clearResults"
          class="px-4 py-2 bg-gray-200 text-gray-700 rounded-md hover:bg-gray-300 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-offset-2"
          :disabled="isLoading || (!content && !analyzedResult)"
        >
          Clear
        </button>
      </div>
      
      <div v-if="analyzedResult" class="space-y-6">
        <div>
          <h3 class="text-lg font-medium text-gray-900 mb-2">Keyword Analysis Results</h3>
          <p class="text-sm text-gray-500">Total words analyzed: {{ analyzedResult.totalWords }}</p>
        </div>
        
        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
          <div>
            <h4 class="font-medium text-gray-700 mb-2">Top Keywords</h4>
            <div class="bg-gray-50 rounded-md border border-gray-200 overflow-hidden">
              <table class="min-w-full divide-y divide-gray-200">
                <thead class="bg-gray-100">
                  <tr>
                    <th scope="col" class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Keyword</th>
                    <th scope="col" class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Count</th>
                    <th scope="col" class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Density</th>
                  </tr>
                </thead>
                <tbody class="divide-y divide-gray-200">
                  <tr v-for="(item, index) in analyzedResult.words" :key="`word-${index}`">
                    <td class="px-4 py-2 whitespace-nowrap text-sm font-medium text-gray-900">{{ item.word }}</td>
                    <td class="px-4 py-2 whitespace-nowrap text-sm text-gray-500">{{ item.count }}</td>
                    <td class="px-4 py-2 whitespace-nowrap text-sm" :class="getDensityColor(item.density)">
                      {{ item.density }}%
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
          
          <div>
            <h4 class="font-medium text-gray-700 mb-2">Top Keyword Phrases</h4>
            <div class="bg-gray-50 rounded-md border border-gray-200 overflow-hidden">
              <table class="min-w-full divide-y divide-gray-200">
                <thead class="bg-gray-100">
                  <tr>
                    <th scope="col" class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Phrase</th>
                    <th scope="col" class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Count</th>
                    <th scope="col" class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Density</th>
                  </tr>
                </thead>
                <tbody class="divide-y divide-gray-200">
                  <tr v-for="(item, index) in analyzedResult.phrases" :key="`phrase-${index}`">
                    <td class="px-4 py-2 whitespace-nowrap text-sm font-medium text-gray-900">{{ item.phrase }}</td>
                    <td class="px-4 py-2 whitespace-nowrap text-sm text-gray-500">{{ item.count }}</td>
                    <td class="px-4 py-2 whitespace-nowrap text-sm" :class="getDensityColor(item.density)">
                      {{ item.density }}%
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
        
        <div class="bg-blue-50 border-l-4 border-blue-400 p-4">
          <div class="flex">
            <div class="flex-shrink-0">
              <svg class="h-5 w-5 text-blue-400" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7-4a1 1 0 11-2 0 1 1 0 012 0zM9 9a1 1 0 000 2v3a1 1 0 001 1h1a1 1 0 100-2h-1V9a1 1 0 00-1-1H9z" clip-rule="evenodd" />
              </svg>
            </div>
            <div class="ml-3">
              <h3 class="text-sm font-medium text-blue-800">SEO Tip</h3>
              <div class="mt-2 text-sm text-blue-700">
                <p>
                  Ideal keyword density is typically between 1-3%. Higher than 5% might be considered keyword stuffing by search engines.
                </p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <template #instructions>
      <ol class="list-decimal list-inside space-y-2 text-gray-600">
        <li>Paste your content into the text area</li>
        <li>Configure analysis options (exclude common words, minimum word length)</li>
        <li>Click "Analyze Content" to check keyword density</li>
        <li>Review individual keywords and phrases</li>
        <li>Look for keywords with density between 1-3% for optimal SEO</li>
        <li>Use this data to optimize your content for search engines</li>
      </ol>
    </template>
  </ToolContainer>
</template>
