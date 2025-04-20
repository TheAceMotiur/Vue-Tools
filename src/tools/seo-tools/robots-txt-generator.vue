<script setup>
import { ref } from 'vue';
import ToolContainer from '../../components/ToolContainer.vue';

const siteURL = ref('');
const robotsEntries = ref([
  {
    userAgent: '*',
    rules: [],
  },
]);
const sitemapURLs = ref([]);
const additionalDirectives = ref('');
const generatedRobots = ref('');
const copied = ref(false);

const commonUserAgents = [
  { name: 'All Robots (*)', value: '*' },
  { name: 'Googlebot', value: 'Googlebot' },
  { name: 'Bingbot', value: 'Bingbot' },
  { name: 'Yahoo! Slurp', value: 'Slurp' },
  { name: 'DuckDuckBot', value: 'DuckDuckBot' },
];

const ruleTypes = [
  { name: 'Allow', value: 'Allow' },
  { name: 'Disallow', value: 'Disallow' },
];

function loadTemplate(template) {
  switch (template) {
    case 'allow-all':
      robotsEntries.value = [{ userAgent: '*', rules: [] }];
      break;
    case 'block-all':
      robotsEntries.value = [{ userAgent: '*', rules: [{ type: 'Disallow', path: '/' }] }];
      break;
    case 'wordpress':
      robotsEntries.value = [
        { userAgent: '*', rules: [{ type: 'Disallow', path: '/wp-admin/' }, { type: 'Allow', path: '/wp-admin/admin-ajax.php' }] },
      ];
      break;
    case 'e-commerce':
      robotsEntries.value = [
        { userAgent: '*', rules: [{ type: 'Disallow', path: '/checkout/' }, { type: 'Disallow', path: '/cart/' }] },
      ];
      break;
  }
  updateGeneratedRobots();
}

function addUserAgent() {
  robotsEntries.value.push({ userAgent: '*', rules: [] });
}

function removeUserAgent(index) {
  robotsEntries.value.splice(index, 1);
  updateGeneratedRobots();
}

function addRule(agentIndex) {
  robotsEntries.value[agentIndex].rules.push({ type: 'Disallow', path: '' });
}

function removeRule(agentIndex, ruleIndex) {
  robotsEntries.value[agentIndex].rules.splice(ruleIndex, 1);
  updateGeneratedRobots();
}

function addSitemap() {
  sitemapURLs.value.push({ url: siteURL.value ? `${siteURL.value.replace(/\/$/, '')}/sitemap.xml` : '' });
}

function removeSitemap(index) {
  sitemapURLs.value.splice(index, 1);
  updateGeneratedRobots();
}

function updateGeneratedRobots() {
  let robotsTxt = '';
  robotsEntries.value.forEach((entry) => {
    robotsTxt += `User-agent: ${entry.userAgent}\n`;
    entry.rules.forEach((rule) => {
      robotsTxt += `${rule.type}: ${rule.path}\n`;
    });
    robotsTxt += '\n';
  });
  sitemapURLs.value.forEach((sitemap) => {
    robotsTxt += `Sitemap: ${sitemap.url}\n`;
  });
  robotsTxt += additionalDirectives.value.trim() ? `\n${additionalDirectives.value.trim()}\n` : '';
  generatedRobots.value = robotsTxt.trim();
}

function copyToClipboard() {
  navigator.clipboard.writeText(generatedRobots.value).then(() => {
    copied.value = true;
    setTimeout(() => (copied.value = false), 2000);
  });
}

function downloadRobotsFile() {
  const blob = new Blob([generatedRobots.value], { type: 'text/plain' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = 'robots.txt';
  a.click();
  URL.revokeObjectURL(url);
}
</script>

<template>
  <ToolContainer 
    title="Robots.txt Generator" 
    description="Create a robots.txt file to control how search engines crawl your site."
  >
    <div class="space-y-6">
      <div>
        <label for="site-url" class="block text-sm font-medium text-gray-700 mb-2">
          Your Site URL (optional):
        </label>
        <div class="mt-1">
          <input
            id="site-url"
            v-model="siteURL"
            type="url"
            class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
            placeholder="https://example.com"
          />
        </div>
      </div>
      
      <div class="flex space-x-4">
        <span class="text-sm font-medium text-gray-700">Load Template:</span>
        <button
          @click="loadTemplate('allow-all')"
          class="px-2 py-1 bg-gray-100 hover:bg-gray-200 text-xs rounded"
        >
          Allow All
        </button>
        <button
          @click="loadTemplate('block-all')"
          class="px-2 py-1 bg-gray-100 hover:bg-gray-200 text-xs rounded"
        >
          Block All
        </button>
        <button
          @click="loadTemplate('wordpress')"
          class="px-2 py-1 bg-gray-100 hover:bg-gray-200 text-xs rounded"
        >
          WordPress
        </button>
        <button
          @click="loadTemplate('e-commerce')"
          class="px-2 py-1 bg-gray-100 hover:bg-gray-200 text-xs rounded"
        >
          E-commerce
        </button>
      </div>
      
      <div class="border rounded-md p-4 bg-gray-50 space-y-6">
        <div v-for="(entry, agentIndex) in robotsEntries" :key="`agent-${agentIndex}`" class="p-4 border rounded-md bg-white space-y-4">
          <div class="flex justify-between items-center">
            <div class="w-full sm:w-auto sm:flex-1">
              <label :for="`user-agent-${agentIndex}`" class="block text-sm font-medium text-gray-700 mb-1">
                User Agent:
              </label>
              <div class="flex space-x-2">
                <select
                  :id="`user-agent-${agentIndex}`"
                  v-model="entry.userAgent"
                  class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 text-sm"
                >
                  <option v-for="agent in commonUserAgents" :key="agent.value" :value="agent.value">
                    {{ agent.name }}
                  </option>
                </select>
                <button
                  v-if="robotsEntries.length > 1"
                  @click="removeUserAgent(agentIndex)"
                  class="text-red-500 hover:text-red-700"
                >
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                  </svg>
                </button>
              </div>
            </div>
          </div>
          
          <div class="space-y-3">
            <div class="flex justify-between items-center">
              <h4 class="text-sm font-medium text-gray-700">Rules:</h4>
              <button
                @click="addRule(agentIndex)"
                class="text-sm text-indigo-600 hover:text-indigo-800 flex items-center"
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
                </svg>
                Add Rule
              </button>
            </div>
            
            <div v-if="entry.rules.length === 0" class="text-sm text-gray-500 italic">
              No rules specified. All pages will be allowed.
            </div>
            
            <div v-for="(rule, ruleIndex) in entry.rules" :key="`rule-${agentIndex}-${ruleIndex}`" class="flex items-center space-x-2">
              <select
                v-model="rule.type"
                class="rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 text-sm"
              >
                <option v-for="type in ruleTypes" :key="type.value" :value="type.value">
                  {{ type.name }}
                </option>
              </select>
              
              <input
                v-model="rule.path"
                type="text"
                class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 text-sm"
                placeholder="/path/to/disallow-or-allow"
              />
              
              <button
                @click="removeRule(agentIndex, ruleIndex)"
                class="text-red-500 hover:text-red-700"
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
                </svg>
              </button>
            </div>
          </div>
        </div>
        
        <button
          @click="addUserAgent"
          class="px-3 py-2 bg-gray-100 hover:bg-gray-200 text-sm rounded flex items-center"
        >
          <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
          </svg>
          Add Another User Agent
        </button>
      </div>
      
      <div class="border rounded-md p-4 bg-gray-50 space-y-4">
        <div class="flex justify-between items-center">
          <h4 class="text-sm font-medium text-gray-700">Sitemaps:</h4>
          <button
            @click="addSitemap"
            class="text-sm text-indigo-600 hover:text-indigo-800 flex items-center"
          >
            <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
            </svg>
            Add Sitemap
          </button>
        </div>
        
        <div v-for="(sitemap, index) in sitemapURLs" :key="`sitemap-${index}`" class="flex items-center space-x-2">
          <div class="flex-grow">
            <input
              v-model="sitemap.url"
              type="text"
              class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 text-sm"
              :placeholder="siteURL ? `${siteURL.replace(/\/$/, '')}/sitemap.xml` : 'https://example.com/sitemap.xml'"
            />
          </div>
          
          <button
            v-if="sitemapURLs.length > 1"
            @click="removeSitemap(index)"
            class="text-red-500 hover:text-red-700"
          >
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
            </svg>
          </button>
        </div>
      </div>
      
      <div>
        <label for="additional-directives" class="block text-sm font-medium text-gray-700 mb-2">
          Additional Custom Directives:
        </label>
        <textarea
          id="additional-directives"
          v-model="additionalDirectives"
          rows="4"
          class="block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 text-sm font-mono"
          placeholder="# Add any additional directives here
Host: example.com"
        ></textarea>
      </div>
      
      <div class="bg-gray-50 rounded-md p-4 border">
        <div class="flex justify-between items-center mb-2">
          <h4 class="text-sm font-medium text-gray-700">Generated robots.txt:</h4>
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
              @click="downloadRobotsFile"
              class="px-3 py-1 bg-green-100 hover:bg-green-200 text-green-700 rounded text-sm flex items-center inline-flex"
            >
              <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M3 17a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm3.293-7.707a1 1 0 011.414 0L9 10.586V3a1 1 0 112 0v7.586l1.293-1.293a1 1 0 111.414 1.414l-3 3a1 1 0 01-1.414 0l-3-3a1 1 0 010-1.414z" clip-rule="evenodd" />
              </svg>
              Download
            </button>
          </div>
        </div>
        
        <pre class="bg-white p-4 rounded-md border border-gray-200 text-sm font-mono overflow-auto max-h-60">{{ generatedRobots }}</pre>
      </div>
    </div>
    
    <template #instructions>
      <ol class="list-decimal list-inside space-y-2 text-gray-600">
        <li>Enter your website URL (optional, used for sitemap paths)</li>
        <li>Choose a template to get started or create your rules from scratch</li>
        <li>Add user agents to specify different rules for different bots</li>
        <li>Add Allow/Disallow rules to control which pages robots can access</li>
        <li>Add sitemap URLs to help search engines find your content</li>
        <li>The preview will update automatically as you make changes</li>
        <li>Click "Copy" or "Download" to get your robots.txt file</li>
      </ol>
    </template>
  </ToolContainer>
</template>