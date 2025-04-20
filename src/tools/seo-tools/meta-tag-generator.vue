<script setup>
import { ref, computed } from 'vue';
import ToolContainer from '../../components/ToolContainer.vue';

const metaInfo = ref({
  title: '',
  description: '',
  keywords: '',
  author: '',
  viewport: 'width=device-width, initial-scale=1.0',
  robots: 'index, follow',
  ogTitle: '',
  ogDescription: '',
  ogImage: '',
  ogUrl: '',
  twitterCard: 'summary_large_image',
  twitterTitle: '',
  twitterDescription: '',
  twitterImage: '',
  favicon: ''
});

const useOpenGraph = ref(true);
const useTwitterCards = ref(true);
const useFavicon = ref(true);

const generateTimeout = ref(null);

const generatedMetaTags = computed(() => {
  let tags = [];
  
  // Basic tags
  if (metaInfo.value.title) {
    tags.push(`<title>${metaInfo.value.title}</title>`);
  }
  
  if (metaInfo.value.description) {
    tags.push(`<meta name="description" content="${metaInfo.value.description}">`);
  }
  
  if (metaInfo.value.keywords) {
    tags.push(`<meta name="keywords" content="${metaInfo.value.keywords}">`);
  }
  
  if (metaInfo.value.author) {
    tags.push(`<meta name="author" content="${metaInfo.value.author}">`);
  }
  
  if (metaInfo.value.viewport) {
    tags.push(`<meta name="viewport" content="${metaInfo.value.viewport}">`);
  }
  
  if (metaInfo.value.robots) {
    tags.push(`<meta name="robots" content="${metaInfo.value.robots}">`);
  }
  
  // OpenGraph tags
  if (useOpenGraph.value) {
    tags.push('<meta property="og:type" content="website">');
    
    if (metaInfo.value.ogTitle || metaInfo.value.title) {
      tags.push(`<meta property="og:title" content="${metaInfo.value.ogTitle || metaInfo.value.title}">`);
    }
    
    if (metaInfo.value.ogDescription || metaInfo.value.description) {
      tags.push(`<meta property="og:description" content="${metaInfo.value.ogDescription || metaInfo.value.description}">`);
    }
    
    if (metaInfo.value.ogImage) {
      tags.push(`<meta property="og:image" content="${metaInfo.value.ogImage}">`);
    }
    
    if (metaInfo.value.ogUrl) {
      tags.push(`<meta property="og:url" content="${metaInfo.value.ogUrl}">`);
    }
  }
  
  // Twitter Card tags
  if (useTwitterCards.value) {
    if (metaInfo.value.twitterCard) {
      tags.push(`<meta name="twitter:card" content="${metaInfo.value.twitterCard}">`);
    }
    
    if (metaInfo.value.twitterTitle || metaInfo.value.title) {
      tags.push(`<meta name="twitter:title" content="${metaInfo.value.twitterTitle || metaInfo.value.title}">`);
    }
    
    if (metaInfo.value.twitterDescription || metaInfo.value.description) {
      tags.push(`<meta name="twitter:description" content="${metaInfo.value.twitterDescription || metaInfo.value.description}">`);
    }
    
    if (metaInfo.value.twitterImage || metaInfo.value.ogImage) {
      tags.push(`<meta name="twitter:image" content="${metaInfo.value.twitterImage || metaInfo.value.ogImage}">`);
    }
  }
  
  // Favicon
  if (useFavicon.value && metaInfo.value.favicon) {
    tags.push(`<link rel="icon" href="${metaInfo.value.favicon}">`);
  }
  
  return tags.join('\n');
});

function copyToClipboard() {
  navigator.clipboard.writeText(generatedMetaTags.value);
  alert('Meta tags copied to clipboard!');
}

function fillDemoData() {
  metaInfo.value = {
    title: 'FreeNetly - Free Online Tools',
    description: 'FreeNetly offers free online tools for developers, designers, and digital marketers. Boost your productivity today!',
    keywords: 'online tools, web tools, developer tools, free tools, productivity',
    author: 'FreeNetly',
    viewport: 'width=device-width, initial-scale=1.0',
    robots: 'index, follow',
    ogTitle: 'FreeNetly - Free Online Tools for Everyone',
    ogDescription: 'Discover our collection of free online tools to help with your daily tasks',
    ogImage: 'https://freenetly.com/images/og-image.jpg',
    ogUrl: 'https://freenetly.com',
    twitterCard: 'summary_large_image',
    twitterTitle: '',
    twitterDescription: '',
    twitterImage: '',
    favicon: '/favicon.ico'
  };
}
</script>

<template>
  <ToolContainer 
    title="Meta Tag Generator" 
    description="Generate optimal meta tags for your website to improve SEO"
  >
    <div class="space-y-6">
      <div class="flex justify-end">
        <button
          @click="fillDemoData"
          class="px-3 py-1.5 bg-gray-100 text-gray-700 rounded border border-gray-300 hover:bg-gray-200 text-sm"
        >
          Fill with demo data
        </button>
      </div>
      
      <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
        <!-- Basic Meta Tags -->
        <div>
          <h3 class="text-lg font-medium text-gray-800 mb-4">Basic Meta Tags</h3>
          
          <div class="space-y-4">
            <div>
              <label for="title" class="block text-sm font-medium text-gray-700 mb-1">
                Page Title
              </label>
              <input
                id="title"
                v-model="metaInfo.title"
                type="text"
                class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                placeholder="Your page title"
              />
            </div>
            
            <div>
              <label for="description" class="block text-sm font-medium text-gray-700 mb-1">
                Description
              </label>
              <textarea
                id="description"
                v-model="metaInfo.description"
                rows="3"
                class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                placeholder="Brief description of your page"
              ></textarea>
            </div>
            
            <div>
              <label for="keywords" class="block text-sm font-medium text-gray-700 mb-1">
                Keywords
              </label>
              <input
                id="keywords"
                v-model="metaInfo.keywords"
                type="text"
                class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                placeholder="keyword1, keyword2, keyword3"
              />
            </div>
            
            <div>
              <label for="author" class="block text-sm font-medium text-gray-700 mb-1">
                Author
              </label>
              <input
                id="author"
                v-model="metaInfo.author"
                type="text"
                class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                placeholder="Page author"
              />
            </div>
          </div>
        </div>
        
        <!-- OpenGraph Tags -->
        <div>
          <div class="flex items-center justify-between mb-4">
            <h3 class="text-lg font-medium text-gray-800">Open Graph Tags</h3>
            <label class="inline-flex items-center">
              <input type="checkbox" v-model="useOpenGraph" class="rounded text-indigo-600 shadow-sm focus:border-indigo-300 focus:ring focus:ring-indigo-200 focus:ring-opacity-50">
              <span class="ml-2 text-sm text-gray-600">Include Open Graph</span>
            </label>
          </div>
          
          <div class="space-y-4" :class="{ 'opacity-50': !useOpenGraph }">
            <div>
              <label for="og-title" class="block text-sm font-medium text-gray-700 mb-1">
                OG Title (optional, uses page title if empty)
              </label>
              <input
                id="og-title"
                v-model="metaInfo.ogTitle"
                type="text"
                class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                placeholder="Social media title"
                :disabled="!useOpenGraph"
              />
            </div>
            
            <div>
              <label for="og-description" class="block text-sm font-medium text-gray-700 mb-1">
                OG Description (optional, uses meta description if empty)
              </label>
              <textarea
                id="og-description"
                v-model="metaInfo.ogDescription"
                rows="3"
                class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                placeholder="Social media description"
                :disabled="!useOpenGraph"
              ></textarea>
            </div>
            
            <div>
              <label for="og-image" class="block text-sm font-medium text-gray-700 mb-1">
                OG Image URL
              </label>
              <input
                id="og-image"
                v-model="metaInfo.ogImage"
                type="url"
                class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                placeholder="https://example.com/image.jpg"
                :disabled="!useOpenGraph"
              />
            </div>
            
            <div>
              <label for="og-url" class="block text-sm font-medium text-gray-700 mb-1">
                OG URL
              </label>
              <input
                id="og-url"
                v-model="metaInfo.ogUrl"
                type="url"
                class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
                placeholder="https://example.com/page"
                :disabled="!useOpenGraph"
              />
            </div>
          </div>
        </div>
      </div>
      
      <div class="border-t border-gray-200 pt-6">
        <div class="flex justify-between items-center mb-4">
          <h3 class="text-lg font-medium text-gray-800">Generated Meta Tags</h3>
          <button
            @click="copyToClipboard"
            class="inline-flex items-center px-3 py-1.5 bg-indigo-100 text-indigo-700 rounded hover:bg-indigo-200"
          >
            <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1" viewBox="0 0 20 20" fill="currentColor">
              <path d="M8 3a1 1 0 011-1h2a1 1 0 110 2H9a1 1 0 01-1-1z" />
              <path d="M6 3a2 2 0 00-2 2v11a2 2 0 002 2h8a2 2 0 002-2V5a2 2 0 00-2-2 3 3 0 01-3 3H9a3 3 0 01-3-3z" />
            </svg>
            Copy
          </button>
        </div>
        
        <div class="bg-gray-50 p-4 border border-gray-200 rounded-md">
          <pre class="font-mono text-sm overflow-x-auto whitespace-pre-wrap">{{ generatedMetaTags }}</pre>
        </div>
      </div>
    </div>
    
    <template #instructions>
      <ol class="list-decimal list-inside space-y-2 text-gray-600">
        <li>Fill in the basic meta tag information (title, description, etc.)</li>
        <li>Toggle Open Graph and Twitter Card options as needed</li>
        <li>Fill in social media specific information</li>
        <li>Copy the generated meta tags and paste them in your HTML <code>&lt;head&gt;</code> section</li>
        <li>You can use the "Fill with demo data" button to see an example</li>
      </ol>
    </template>
  </ToolContainer>
</template>
