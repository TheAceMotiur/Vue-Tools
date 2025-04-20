<script setup>
import { ref, computed, watch } from 'vue';
import ToolContainer from '../../components/ToolContainer.vue';

const websiteURL = ref('');
const sitemapEntries = ref([
  { url: '', priority: '0.8', changefreq: 'monthly', lastmod: '' }
]);
const includeLastmod = ref(true);
const includeChangefreq = ref(true);
const includePriority = ref(true);
const copied = ref(false);
const urlError = ref('');

const changefreqOptions = [
  { value: 'always', label: 'Always' },
  { value: 'hourly', label: 'Hourly' },
  { value: 'daily', label: 'Daily' },
  { value: 'weekly', label: 'Weekly' },
  { value: 'monthly', label: 'Monthly' },
  { value: 'yearly', label: 'Yearly' },
  { value: 'never', label: 'Never' }
];

const priorityOptions = [
  { value: '1.0', label: '1.0 (Highest)' },
  { value: '0.9', label: '0.9' },
  { value: '0.8', label: '0.8 (Default)' },
  { value: '0.7', label: '0.7' },
  { value: '0.6', label: '0.6' },
  { value: '0.5', label: '0.5 (Medium)' },
  { value: '0.4', label: '0.4' },
  { value: '0.3', label: '0.3' },
  { value: '0.2', label: '0.2' },
  { value: '0.1', label: '0.1 (Lowest)' }
];

// Validate website URL
watch(websiteURL, (newValue) => {
  if (!newValue) {
    urlError.value = '';
    return;
  }
  
  try {
    const url = new URL(newValue);
    if (url.protocol !== 'http:' && url.protocol !== 'https:') {
      urlError.value = 'URL must start with http:// or https://';
    } else {
      urlError.value = '';
      
      // Auto-update URLs with website domain if they're relative paths
      sitemapEntries.value = sitemapEntries.value.map(entry => {
        if (entry.url && !entry.url.match(/^https?:\/\//)) {
          return { 
            ...entry, 
            url: `${websiteURL.value.replace(/\/$/, '')}/${entry.url.replace(/^\//, '')}` 
          };
        }
        return entry;
      });
    }
  } catch (e) {
    urlError.value = 'Please enter a valid URL';
  }
});

function addEntry() {
  sitemapEntries.value.push({ 
    url: '', 
    priority: '0.8', 
    changefreq: 'monthly', 
    lastmod: ''  
  });
}

function removeEntry(index) {
  sitemapEntries.value.splice(index, 1);
}

function formatDate(dateString) {
  if (!dateString) return '';
  
  try {
    const date = new Date(dateString);
    return date.toISOString().split('T')[0];
  } catch (e) {
    return '';
  }
}

function setTodayDate(index) {
  const today = new Date().toISOString().split('T')[0];
  sitemapEntries.value[index].lastmod = today;
}

function addHomePage() {
  if (!websiteURL.value) {
    urlError.value = 'Please enter your website URL first';
    return;
  }
  
  sitemapEntries.value.unshift({
    url: websiteURL.value,
    priority: '1.0',
    changefreq: 'weekly',
    lastmod: new Date().toISOString().split('T')[0]
  });
}

function addCommonPages() {
  if (!websiteURL.value) {
    urlError.value = 'Please enter your website URL first';
    return;
  }
  
  const baseUrl = websiteURL.value.replace(/\/$/, '');
  const commonPages = [
    { path: '/about', priority: '0.8', changefreq: 'monthly' },
    { path: '/contact', priority: '0.8', changefreq: 'monthly' },
    { path: '/services', priority: '0.9', changefreq: 'weekly' },
    { path: '/products', priority: '0.9', changefreq: 'weekly' },
    { path: '/blog', priority: '0.8', changefreq: 'weekly' },
    { path: '/faq', priority: '0.7', changefreq: 'monthly' },
    { path: '/terms', priority: '0.3', changefreq: 'yearly' },
    { path: '/privacy', priority: '0.3', changefreq: 'yearly' }
  ];
  
  // Only add pages that don't already exist
  const existingUrls = sitemapEntries.value.map(entry => entry.url);
  
  for (const page of commonPages) {
    const fullUrl = `${baseUrl}${page.path}`;
    if (!existingUrls.includes(fullUrl)) {
      sitemapEntries.value.push({
        url: fullUrl,
        priority: page.priority,
        changefreq: page.changefreq,
        lastmod: includeLastmod.value ? new Date().toISOString().split('T')[0] : ''
      });
    }
  }
}

// Generate sitemap XML content
const generatedSitemap = computed(() => {
  // XML header and opening tags
  let xml = '<?xml version="1.0" encoding="UTF-8"?>\n';
  xml += '<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">\n';
  
  // Add entries
  for (const entry of sitemapEntries.value) {
    if (entry.url) {
      xml += '  <url>\n';
      xml += `    <loc>${entry.url.trim()}</loc>\n`;
      
      if (includeLastmod.value && entry.lastmod) {
        xml += `    <lastmod>${formatDate(entry.lastmod)}</lastmod>\n`;
      }
      
      if (includeChangefreq.value && entry.changefreq) {
        xml += `    <changefreq>${entry.changefreq}</changefreq>\n`;
      }
      
      if (includePriority.value && entry.priority) {
        xml += `    <priority>${entry.priority}</priority>\n`;
      }
      
      xml += '  </url>\n';
    }
  }
  
  // Close urlset tag
  xml += '</urlset>';
  
  return xml;
});

function copyToClipboard() {
  navigator.clipboard.writeText(generatedSitemap.value);
  copied.value = true;
  
  setTimeout(() => {
    copied.value = false;
  }, 2000);
}

function downloadSitemap() {
  const blob = new Blob([generatedSitemap.value], { type: 'application/xml' });
  const url = URL.createObjectURL(blob);
  const link = document.createElement('a');
  link.href = url;
  link.download = 'sitemap.xml';
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
}

function validateUrl(url) {
  if (!url) return false;
  
  try {
    new URL(url);
    return true;
  } catch (e) {
    return false;
  }
}
</script>

<template>
  <ToolContainer 
    title="XML Sitemap Generator" 
    description="Create an XML sitemap to help search engines discover and index your website pages."
  >
    <div class="space-y-6">
      <div>
        <label for="website-url" class="block text-sm font-medium text-gray-700 mb-2">
          Your Website URL:
        </label>
        <div class="mt-1">
          <input
            id="website-url"
            v-model="websiteURL"
            type="url"
            class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
            :class="{ 'border-red-300': urlError }"
            placeholder="https://example.com"
          />
          <p v-if="urlError" class="mt-1 text-sm text-red-600">{{ urlError }}</p>
        </div>
      </div>
      
      <div class="bg-gray-50 p-4 rounded-md border border-gray-200 flex flex-wrap gap-4">
        <div class="flex items-center">
          <label class="inline-flex items-center">
            <input type="checkbox" v-model="includeLastmod" class="form-checkbox h-4 w-4 text-indigo-600">
            <span class="ml-2 text-sm text-gray-700">Include Last Modified</span>
          </label>
        </div>
        
        <div class="flex items-center">
          <label class="inline-flex items-center">
            <input type="checkbox" v-model="includeChangefreq" class="form-checkbox h-4 w-4 text-indigo-600">
            <span class="ml-2 text-sm text-gray-700">Include Change Frequency</span>
          </label>
        </div>
        
        <div class="flex items-center">
          <label class="inline-flex items-center">
            <input type="checkbox" v-model="includePriority" class="form-checkbox h-4 w-4 text-indigo-600">
            <span class="ml-2 text-sm text-gray-700">Include Priority</span>
          </label>
        </div>
        
        <div class="flex space-x-2 ml-auto">
          <button
            @click="addHomePage"
            class="px-3 py-1 bg-gray-200 hover:bg-gray-300 text-gray-700 rounded text-sm"
            :disabled="!websiteURL"
          >
            Add Homepage
          </button>
          
          <button
            @click="addCommonPages"
            class="px-3 py-1 bg-gray-200 hover:bg-gray-300 text-gray-700 rounded text-sm"
            :disabled="!websiteURL"
          >
            Add Common Pages
          </button>
        </div>
      </div>
      
      <div class="border rounded-md overflow-hidden">
        <table class="min-w-full divide-y divide-gray-200">
          <thead class="bg-gray-50">
            <tr>
              <th scope="col" class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                URL
              </th>
              <th v-if="includePriority" scope="col" class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Priority
              </th>
              <th v-if="includeChangefreq" scope="col" class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Change Frequency
              </th>
              <th v-if="includeLastmod" scope="col" class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Last Modified
              </th>
              <th scope="col" class="relative px-4 py-3">
                <span class="sr-only">Actions</span>
              </th>
            </tr>
          </thead>
          <tbody class="bg-white divide-y divide-gray-200">
            <tr v-for="(entry, index) in sitemapEntries" :key="`entry-${index}`" :class="!validateUrl(entry.url) && entry.url ? 'bg-red-50' : ''">
              <td class="px-4 py-2">
                <input
                  v-model="entry.url"
                  type="text"
                  class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 text-sm"
                  :class="!validateUrl(entry.url) && entry.url ? 'border-red-300' : ''"
                  :placeholder="websiteURL ? `${websiteURL}/page` : 'https://example.com/page'"
                />
              </td>
              <td v-if="includePriority" class="px-4 py-2">
                <select
                  v-model="entry.priority"
                  class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 text-sm"
                >
                  <option v-for="option in priorityOptions" :key="option.value" :value="option.value">
                    {{ option.label }}
                  </option>
                </select>
              </td>
              <td v-if="includeChangefreq" class="px-4 py-2">
                <select
                  v-model="entry.changefreq"
                  class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 text-sm"
                >
                  <option v-for="option in changefreqOptions" :key="option.value" :value="option.value">
                    {{ option.label }}
                  </option>
                </select>
              </td>
              <td v-if="includeLastmod" class="px-4 py-2">
                <div class="flex space-x-2">
                  <input
                    v-model="entry.lastmod"
                    type="date"
                    class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 text-sm"
                  />
                  <button
                    @click="setTodayDate(index)"
                    class="px-2 py-1 bg-gray-100 hover:bg-gray-200 text-xs rounded"
                    title="Set to today's date"
                  >
                    Today
                  </button>
                </div>
              </td>
              <td class="px-4 py-2 whitespace-nowrap text-right text-sm font-medium">
                <button
                  @click="removeEntry(index)"
                  class="text-red-500 hover:text-red-700"
                >
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                  </svg>
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
      
      <div class="flex justify-center">
        <button
          @click="addEntry"
          class="px-4 py-2 bg-indigo-100 text-indigo-700 rounded-md hover:bg-indigo-200 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2"
        >
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 inline-block mr-1" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd" d="M10 3a1 1 0 011 1v5h5a1 1 0 110 2h-5v5a1 1 0 11-2 0v-5H4a1 1 0 110-2h5V4a1 1 0 011-1z" clip-rule="evenodd" />
          </svg>
          Add URL
        </button>
      </div>
      
      <div class="bg-gray-50 rounded-md p-4 border">
        <div class="flex justify-between items-center mb-2">
          <h4 class="text-sm font-medium text-gray-700">Generated XML Sitemap:</h4>
          <div class="space-x-2">
            <button
              @click="copyToClipboard"
              class="px-3 py-1 bg-indigo-100 hover:bg-indigo-200 text-indigo-700 rounded text-sm flex items-center inline-flex"
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
            
            <button
              @click="downloadSitemap"
              class="px-3 py-1 bg-green-100 hover:bg-green-200 text-green-700 rounded text-sm flex items-center inline-flex"
            >
              <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M3 17a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm3.293-7.707a1 1 0 011.414 0L9 10.586V3a1 1 0 112 0v7.586l1.293-1.293a1 1 0 111.414 1.414l-3 3a1 1 0 01-1.414 0l-3-3a1 1 0 010-1.414z" clip-rule="evenodd" />
              </svg>
              Download
            </button>
          </div>
        </div>
        
        <pre class="bg-white p-4 rounded-md border border-gray-200 text-sm font-mono overflow-auto max-h-60">{{ generatedSitemap }}</pre>
      </div>
      
      <div class="bg-blue-50 border-l-4 border-blue-400 p-4">
        <div class="flex">
          <div class="flex-shrink-0">
            <svg class="h-5 w-5 text-blue-400" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
              <path fill-rule="evenodd" d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-7-4a1 1 0 11-2 0 1 1 0 012 0zM9 9a1 1 0 000 2v3a1 1 0 001 1h1a1 1 0 100-2h-1V9a1 1 0 00-1-1H9z" clip-rule="evenodd" />
            </svg>
          </div>
          <div class="ml-3">
            <h3 class="text-sm font-medium text-blue-800">Next Steps</h3>
            <div class="mt-2 text-sm text-blue-700">
              <p>
                After creating your sitemap.xml file:
              </p>
              <ol class="list-decimal list-inside mt-1">
                <li>Upload it to your website's root directory</li>
                <li>Add it to your robots.txt file with: Sitemap: {{ websiteURL || 'https://example.com' }}/sitemap.xml</li>
                <li>Submit it to Google Search Console, Bing Webmaster Tools, and other search engines</li>
              </ol>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <template #instructions>
      <ol class="list-decimal list-inside space-y-2 text-gray-600">
        <li>Enter your website URL</li>
        <li>Choose which elements to include in your sitemap (last modified date, change frequency, priority)</li>
        <li>Add URLs manually or use the quick buttons to add a homepage and common pages</li>
        <li>For each URL, set the appropriate priority and change frequency</li>
        <li>Add last modified dates for each URL (use the "Today" button to set current date)</li>
        <li>Copy or download your XML sitemap when finished</li>
        <li>Upload the sitemap.xml file to your website's root directory</li>
      </ol>
    </template>
  </ToolContainer>
</template>
