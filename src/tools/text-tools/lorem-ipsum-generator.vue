<script setup>
import { ref, computed } from 'vue';
import ToolContainer from '../../components/ToolContainer.vue';

const outputType = ref('paragraphs'); // 'paragraphs', 'words', 'sentences', 'lists'
const count = ref(3);
const includeHTML = ref(false);
const result = ref('');
const copied = ref(false);

// Lorem ipsum dictionary
const words = [
  'lorem', 'ipsum', 'dolor', 'sit', 'amet', 'consectetur', 'adipiscing', 'elit',
  'a', 'ac', 'accumsan', 'ad', 'aenean', 'aliquam', 'aliquet', 'ante',
  'aptent', 'arcu', 'at', 'auctor', 'augue', 'bibendum', 'blandit', 'class',
  'commodo', 'condimentum', 'congue', 'consequat', 'conubia', 'convallis', 'cras', 'cubilia',
  'curabitur', 'curae', 'cursus', 'dapibus', 'diam', 'dictum', 'dictumst', 'dignissim',
  'dis', 'donec', 'dui', 'duis', 'efficitur', 'egestas', 'eget', 'eleifend',
  'elementum', 'enim', 'erat', 'eros', 'est', 'et', 'etiam', 'eu',
  'euismod', 'ex', 'facilisi', 'facilisis', 'fames', 'faucibus', 'felis', 'fermentum',
  'feugiat', 'finibus', 'fringilla', 'fusce', 'gravida', 'habitant', 'habitasse', 'hac',
  'hendrerit', 'himenaeos', 'iaculis', 'id', 'imperdiet', 'in', 'inceptos', 'integer',
  'interdum', 'justo', 'lacinia', 'lacus', 'laoreet', 'lectus', 'leo', 'libero',
  'ligula', 'litora', 'lobortis', 'luctus', 'maecenas', 'magna', 'magnis', 'malesuada',
  'massa', 'mattis', 'mauris', 'maximus', 'metus', 'mi', 'molestie', 'mollis',
  'montes', 'morbi', 'mus', 'nam', 'nascetur', 'natoque', 'nec', 'neque',
  'netus', 'nibh', 'nisi', 'nisl', 'non', 'nostra', 'nulla', 'nullam',
  'nunc', 'odio', 'orci', 'ornare', 'parturient', 'pellentesque', 'penatibus', 'per',
  'pharetra', 'phasellus', 'placerat', 'platea', 'porta', 'porttitor', 'posuere', 'potenti',
  'praesent', 'pretium', 'primis', 'proin', 'pulvinar', 'purus', 'quam', 'quis',
  'quisque', 'rhoncus', 'ridiculus', 'risus', 'rutrum', 'sagittis', 'sapien', 'scelerisque',
  'sed', 'sem', 'semper', 'senectus', 'sociosqu', 'sodales', 'sollicitudin', 'suscipit',
  'suspendisse', 'taciti', 'tellus', 'tempor', 'tempus', 'tincidunt', 'torquent', 'tortor',
  'tristique', 'turpis', 'ullamcorper', 'ultrices', 'ultricies', 'urna', 'ut', 'varius',
  'vehicula', 'vel', 'velit', 'venenatis', 'vestibulum', 'vitae', 'vivamus', 'viverra',
  'volutpat', 'vulputate'
];

// Helper functions
function getRandomWord() {
  return words[Math.floor(Math.random() * words.length)];
}

function capitalizeFirst(text) {
  return text.charAt(0).toUpperCase() + text.slice(1);
}

function generateSentence(minWords = 5, maxWords = 15) {
  const numWords = Math.floor(Math.random() * (maxWords - minWords + 1)) + minWords;
  let sentence = '';
  
  for (let i = 0; i < numWords; i++) {
    sentence += getRandomWord();
    if (i < numWords - 1) {
      sentence += ' ';
    }
  }
  
  return capitalizeFirst(sentence) + '.';
}

function generateParagraph(minSentences = 3, maxSentences = 8) {
  const numSentences = Math.floor(Math.random() * (maxSentences - minSentences + 1)) + minSentences;
  let paragraph = '';
  
  for (let i = 0; i < numSentences; i++) {
    paragraph += generateSentence();
    if (i < numSentences - 1) {
      paragraph += ' ';
    }
  }
  
  return paragraph;
}

function generateContent() {
  let content = '';
  
  switch (outputType.value) {
    case 'paragraphs':
      for (let i = 0; i < count.value; i++) {
        content += includeHTML.value ? `<p>${generateParagraph()}</p>` : generateParagraph();
        if (i < count.value - 1) {
          content += includeHTML.value ? '\n\n' : '\n\n';
        }
      }
      break;
      
    case 'sentences':
      for (let i = 0; i < count.value; i++) {
        content += generateSentence();
        if (i < count.value - 1) {
          content += ' ';
        }
      }
      break;
      
    case 'words':
      let wordList = [];
      for (let i = 0; i < count.value; i++) {
        wordList.push(getRandomWord());
      }
      content = wordList.join(' ');
      break;
      
    case 'lists':
      if (includeHTML.value) {
        content += '<ul>\n';
        for (let i = 0; i < count.value; i++) {
          content += `  <li>${generateSentence(3, 8)}</li>\n`;
        }
        content += '</ul>';
      } else {
        for (let i = 0; i < count.value; i++) {
          content += `• ${generateSentence(3, 8)}\n`;
        }
      }
      break;
  }
  
  result.value = content;
  copied.value = false;
}

function copyToClipboard() {
  navigator.clipboard.writeText(result.value);
  copied.value = true;
  
  setTimeout(() => {
    copied.value = false;
  }, 2000);
}
</script>

<template>
  <ToolContainer 
    title="Lorem Ipsum Generator" 
    description="Generate placeholder text for your designs and layouts."
  >
    <div class="space-y-6">
      <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div class="space-y-4">
          <div>
            <label for="output-type" class="block text-sm font-medium text-gray-700 mb-2">
              Output Type:
            </label>
            <select
              id="output-type"
              v-model="outputType"
              class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
            >
              <option value="paragraphs">Paragraphs</option>
              <option value="sentences">Sentences</option>
              <option value="words">Words</option>
              <option value="lists">List Items</option>
            </select>
          </div>
          
          <div>
            <label for="count" class="block text-sm font-medium text-gray-700 mb-2">
              Count ({{ outputType }}):
            </label>
            <input
              id="count"
              v-model.number="count"
              type="number"
              min="1"
              max="100"
              class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
            />
          </div>
          
          <div>
            <label class="inline-flex items-center">
              <input type="checkbox" v-model="includeHTML" class="form-checkbox text-indigo-600">
              <span class="ml-2 text-sm text-gray-700">Include HTML tags</span>
            </label>
          </div>
          
          <div>
            <button
              @click="generateContent"
              class="w-full px-4 py-2 bg-indigo-600 text-white rounded-md hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2"
            >
              Generate Lorem Ipsum
            </button>
          </div>
        </div>
        
        <div>
          <div class="flex justify-between items-center mb-2">
            <label for="result" class="block text-sm font-medium text-gray-700">
              Generated Text:
            </label>
            
            <button
              @click="copyToClipboard"
              class="px-3 py-1 text-xs bg-gray-200 hover:bg-gray-300 rounded-md flex items-center"
              :class="{ 'bg-green-100 text-green-800': copied }"
            >
              <svg v-if="copied" xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd" />
              </svg>
              <svg v-else xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1" viewBox="0 0 20 20" fill="currentColor">
                <path d="M8 3a1 1 0 011-1h2a1 1 0 110 2H9a1 1 0 01-1-1z" />
                <path d="M6 3a2 2 0 00-2 2v11a2 2 0 002 2h8a2 2 0 002-2V5a2 2 0 00-2-2 3 3 0 01-3 3H9a3 3 0 01-3-3z" />
              </svg>
              {{ copied ? 'Copied!' : 'Copy' }}
            </button>
          </div>
          
          <textarea
            id="result"
            v-model="result"
            readonly
            rows="12"
            class="block w-full font-mono rounded-md border-gray-300 bg-gray-50 shadow-sm"
            placeholder="Your generated text will appear here..."
          ></textarea>
        </div>
      </div>
    </div>
    
    <template #instructions>
      <ol class="list-decimal list-inside space-y-2 text-gray-600">
        <li>Select the type of content you want to generate (paragraphs, sentences, words, or list items)</li>
        <li>Enter the number of items you want to generate</li>
        <li>Check "Include HTML tags" if you need HTML formatted text</li>
        <li>Click "Generate Lorem Ipsum" to create your placeholder text</li>
        <li>Copy the generated text to your clipboard by clicking the "Copy" button</li>
      </ol>
    </template>
  </ToolContainer>
</template>
