<template>
  <div class="bg-white p-6 rounded-lg shadow-md">
    <h2 class="text-2xl font-bold mb-4">XML Sitemap Generator</h2>
    
    <!-- Base URL input section -->
    <div class="mb-6">
      <label class="block text-gray-700 font-medium mb-2">Website URL</label>
      <div class="flex">
        <input
          v-model="baseUrl"
          type="text"
          placeholder="https://example.com"
          class="flex-grow px-4 py-2 border rounded-l focus:outline-none focus:ring-2 focus:ring-blue-500"
          :class="{'border-red-500': urlError}"
        />
        <button 
          @click="fetchUrls" 
          class="bg-blue-600 text-white px-4 py-2 rounded-r hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500"
          :disabled="isLoading || !isValidUrl(baseUrl)"
        >
          <span v-if="isLoading">Loading...</span>
          <span v-else>Fetch URLs</span>
        </button>
      </div>
      <p v-if="urlError" class="text-sm text-red-500 mt-1">{{ urlError }}</p>
      <p class="text-sm text-gray-500 mt-1">Enter the base URL of your website to create a sitemap.</p>
    </div>

    <!-- URL exclusion patterns -->
    <div class="mb-6">
      <div class="flex justify-between items-center mb-2">
        <label class="block text-gray-700 font-medium">Exclude URL Patterns</label>
        <button 
          @click="addExclusionPattern" 
          class="text-sm text-blue-600 hover:text-blue-800 focus:outline-none"
        >
          + Add Pattern
        </button>
      </div>
      
      <div v-if="exclusionPatterns.length === 0" class="text-gray-500 text-sm mb-2">
        No exclusion patterns added. URLs matching these patterns will be ignored during fetch.
      </div>
      
      <div v-for="(pattern, index) in exclusionPatterns" :key="'pattern-'+index" class="mb-2">
        <div class="flex">
          <input
            v-model="exclusionPatterns[index]"
            type="text"
            placeholder="*/admin/* or *.pdf"
            class="flex-grow px-3 py-1 border rounded-l focus:outline-none focus:ring-2 focus:ring-blue-500"
          />
          <button 
            @click="removeExclusionPattern(index)" 
            class="bg-gray-200 px-3 py-1 rounded-r hover:bg-gray-300"
          >
            ✕
          </button>
        </div>
      </div>
    </div>

    <!-- Manual URL entries section -->
    <div class="mb-6">
      <div class="flex justify-between items-center mb-2">
        <label class="block text-gray-700 font-medium">URLs to include</label>
        <button 
          @click="addNewUrl" 
          class="text-sm text-blue-600 hover:text-blue-800 focus:outline-none"
        >
          + Add URL
        </button>
      </div>
      
      <div v-if="urls.length === 0" class="text-gray-500 text-center py-4 border rounded bg-gray-50">
        No URLs added yet. Fetch URLs automatically or add them manually.
      </div>
      
      <div v-for="(url, index) in urls" :key="index" class="mb-2">
        <div class="flex items-start">
          <div class="flex-grow">
            <div class="border rounded overflow-hidden">
              <!-- URL input with accordion for settings -->
              <div class="flex items-center px-3 py-2 bg-gray-50 border-b">
                <input
                  v-model="url.loc"
                  type="text"
                  class="flex-grow bg-transparent focus:outline-none"
                  placeholder="https://example.com/page"
                  :class="{'border-red-500': url.error}"
                />
                <button 
                  @click="url.showSettings = !url.showSettings" 
                  class="ml-2 text-gray-500 hover:text-gray-700"
                >
                  <span v-if="!url.showSettings">⚙️</span>
                  <span v-else>▲</span>
                </button>
                <button 
                  @click="removeUrl(index)" 
                  class="ml-2 text-red-500 hover:text-red-700"
                >
                  ✕
                </button>
              </div>
              <p v-if="url.error" class="text-xs text-red-500 px-3 py-1 bg-red-50">{{ url.error }}</p>
              
              <!-- URL settings (expanded when showSettings is true) -->
              <div v-if="url.showSettings" class="p-3 bg-gray-50">
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                  <!-- Priority setting -->
                  <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1">Priority</label>
                    <select v-model="url.priority" class="w-full border rounded px-2 py-1">
                      <option value="1.0">1.0 (Highest)</option>
                      <option value="0.9">0.9</option>
                      <option value="0.8">0.8 (High)</option>
                      <option value="0.7">0.7</option>
                      <option value="0.6">0.6</option>
                      <option value="0.5">0.5 (Medium)</option>
                      <option value="0.4">0.4</option>
                      <option value="0.3">0.3</option>
                      <option value="0.2">0.2 (Low)</option>
                      <option value="0.1">0.1</option>
                    </select>
                  </div>
                  
                  <!-- Change frequency setting -->
                  <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1">Change Frequency</label>
                    <select v-model="url.changefreq" class="w-full border rounded px-2 py-1">
                      <option value="always">Always</option>
                      <option value="hourly">Hourly</option>
                      <option value="daily">Daily</option>
                      <option value="weekly">Weekly</option>
                      <option value="monthly">Monthly</option>
                      <option value="yearly">Yearly</option>
                      <option value="never">Never</option>
                    </select>
                  </div>
                  
                  <!-- Last modified date -->
                  <div>
                    <label class="block text-sm font-medium text-gray-700 mb-1">Last Modified</label>
                    <input 
                      type="date" 
                      v-model="url.lastmod" 
                      class="w-full border rounded px-2 py-1"
                    />
                  </div>

                  <!-- Image sitemap options -->
                  <div>
                    <div class="flex items-center">
                      <input 
                        type="checkbox" 
                        :id="`hasImage-${index}`" 
                        v-model="url.hasImage"
                        class="h-4 w-4 text-blue-600 border-gray-300 rounded"
                      />
                      <label :for="`hasImage-${index}`" class="ml-2 text-sm text-gray-700">
                        Has image
                      </label>
                    </div>
                  </div>
                  
                  <!-- Image URL and caption (if hasImage is checked) -->
                  <div v-if="url.hasImage" class="md:col-span-2">
                    <div class="space-y-2">
                      <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Image URL</label>
                        <input 
                          type="text" 
                          v-model="url.image.url" 
                          placeholder="https://example.com/image.jpg"
                          class="w-full border rounded px-2 py-1"
                        />
                      </div>
                      <div>
                        <label class="block text-sm font-medium text-gray-700 mb-1">Image Caption (optional)</label>
                        <input 
                          type="text" 
                          v-model="url.image.caption" 
                          placeholder="Image description"
                          class="w-full border rounded px-2 py-1"
                        />
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Sitemap options section -->
    <div class="mb-6">
      <h3 class="text-lg font-medium mb-3">Sitemap Options</h3>
      <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">Sitemap Format</label>
          <select v-model="sitemapFormat" class="w-full border rounded px-2 py-1">
            <option value="xml">XML Sitemap</option>
            <option value="rss">RSS Feed</option>
            <option value="txt">Text List</option>
            <option value="html">HTML Sitemap</option>
          </select>
        </div>
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">Default Priority</label>
          <select v-model="defaultSettings.priority" class="w-full border rounded px-2 py-1">
            <option value="0.8">0.8 (High)</option>
            <option value="0.5">0.5 (Medium)</option>
            <option value="0.3">0.3 (Low)</option>
          </select>
        </div>
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">Default Change Frequency</label>
          <select v-model="defaultSettings.changefreq" class="w-full border rounded px-2 py-1">
            <option value="daily">Daily</option>
            <option value="weekly">Weekly</option>
            <option value="monthly">Monthly</option>
          </select>
        </div>
        <div class="flex items-center">
          <input 
            type="checkbox" 
            id="includeLastMod" 
            v-model="includeLastMod"
            class="h-4 w-4 text-blue-600 border-gray-300 rounded"
          />
          <label for="includeLastMod" class="ml-2 text-sm text-gray-700">
            Include last modified date
          </label>
        </div>
      </div>
    </div>

    <!-- Advanced Settings (collapsible) -->
    <div class="mb-6">
      <div 
        @click="showAdvancedSettings = !showAdvancedSettings" 
        class="flex justify-between items-center cursor-pointer bg-gray-50 p-3 rounded border"
      >
        <h3 class="text-lg font-medium">Advanced Settings</h3>
        <span>{{ showAdvancedSettings ? '▲' : '▼' }}</span>
      </div>
      
      <div v-if="showAdvancedSettings" class="mt-3 p-4 border rounded">
        <div class="space-y-4">
          <!-- Robots.txt generator -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Generate robots.txt</label>
            <div class="flex items-center mb-2">
              <input 
                type="checkbox" 
                id="generateRobots" 
                v-model="generateRobots"
                class="h-4 w-4 text-blue-600 border-gray-300 rounded"
              />
              <label for="generateRobots" class="ml-2 text-sm text-gray-700">
                Create robots.txt file with sitemap reference
              </label>
            </div>
            
            <div v-if="generateRobots" class="mt-2">
              <textarea 
                v-model="robotsTxt"
                class="w-full h-32 border rounded p-2 font-mono text-sm"
                placeholder="User-agent: *&#10;Disallow: /admin/&#10;Sitemap: https://example.com/sitemap.xml"
              ></textarea>
            </div>
          </div>
          
          <!-- Sitemap index option -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Sitemap Index</label>
            <div class="flex items-center mb-2">
              <input 
                type="checkbox" 
                id="createSitemapIndex" 
                v-model="createSitemapIndex"
                class="h-4 w-4 text-blue-600 border-gray-300 rounded"
              />
              <label for="createSitemapIndex" class="ml-2 text-sm text-gray-700">
                Create a sitemap index file (for large sites)
              </label>
            </div>
            
            <div v-if="createSitemapIndex" class="mt-2">
              <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                  <label class="block text-xs font-medium text-gray-700 mb-1">URLs per sitemap</label>
                  <input 
                    type="number" 
                    v-model="urlsPerSitemap"
                    min="1" 
                    max="50000"
                    class="w-full border rounded px-2 py-1"
                  />
                  <p class="text-xs text-gray-500 mt-1">Maximum: 50,000 URLs per sitemap file</p>
                </div>
                
                <div>
                  <label class="block text-xs font-medium text-gray-700 mb-1">Sitemap naming pattern</label>
                  <input 
                    type="text" 
                    v-model="sitemapNamePattern"
                    placeholder="sitemap_{num}.xml"
                    class="w-full border rounded px-2 py-1"
                  />
                </div>
              </div>
            </div>
          </div>
          
          <!-- Google verification code -->
          <div>
            <label class="block text-sm font-medium text-gray-700 mb-1">Google Site Verification</label>
            <input 
              type="text" 
              v-model="googleVerification"
              placeholder="Google verification meta tag content"
              class="w-full border rounded px-2 py-1"
            />
            <p class="text-xs text-gray-500 mt-1">Add Google Search Console verification code (optional)</p>
          </div>
        </div>
      </div>
    </div>

    <!-- Generate sitemap button -->
    <div class="mb-6">
      <button 
        @click="generateSitemap" 
        class="w-full bg-green-600 text-white py-3 rounded-md hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-green-500"
        :disabled="urls.length === 0"
      >
        Generate Sitemap
      </button>
    </div>

    <!-- Status messages -->
    <div v-if="statusMessage" class="mb-6">
      <div :class="[
        'p-3 rounded flex items-center',
        statusType === 'success' ? 'bg-green-100 text-green-800' : 'bg-red-100 text-red-800'
      ]">
        <span v-if="statusType === 'success'" class="mr-2">✓</span>
        <span v-else class="mr-2">⚠️</span>
        {{ statusMessage }}
      </div>
    </div>

    <!-- Sitemap file size estimation -->
    <div v-if="estimatedSize" class="mb-6">
      <div class="p-3 bg-blue-50 text-blue-800 rounded">
        <p>Estimated sitemap size: {{ estimatedSize }}</p>
        <p v-if="parseFloat(estimatedSize) > 10" class="text-sm mt-1">
          Large sitemaps (>10MB) may need to be split into smaller files.
        </p>
      </div>
    </div>

    <!-- Sitemap preview section -->
    <div v-if="generatedSitemap" class="mb-6">
      <div class="flex justify-between items-center mb-2">
        <h3 class="text-lg font-medium">Sitemap Preview</h3>
        <div>
          <button 
            @click="downloadSitemap" 
            class="bg-blue-600 text-white px-4 py-1 rounded hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            Download
          </button>
          <button 
            @click="copyToClipboard" 
            class="ml-2 bg-gray-200 text-gray-800 px-4 py-1 rounded hover:bg-gray-300 focus:outline-none focus:ring-2 focus:ring-gray-400"
          >
            Copy
          </button>
        </div>
      </div>
      <div class="border rounded bg-gray-50 p-4 overflow-auto max-h-96">
        <pre><code>{{ generatedSitemap }}</code></pre>
      </div>
    </div>

    <!-- Help and tips section -->
    <div class="bg-blue-50 p-4 rounded-md border border-blue-100">
      <h3 class="text-lg font-medium text-blue-800 mb-2">Tips for Effective Sitemaps</h3>
      <ul class="list-disc list-inside text-sm text-blue-700 space-y-1">
        <li>Include all important pages but exclude duplicate or non-essential pages</li>
        <li>Set higher priority (0.8-1.0) for more important pages</li>
        <li>Update your sitemap when you add or change content</li>
        <li>Submit your sitemap to Google Search Console and Bing Webmaster Tools</li>
        <li>For large sites, consider creating multiple sitemaps</li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: 'XmlSitemapGenerator',
  data() {
    return {
      baseUrl: '',
      urls: [],
      exclusionPatterns: [],
      isLoading: false,
      sitemapFormat: 'xml',
      defaultSettings: {
        priority: '0.5',
        changefreq: 'weekly',
      },
      includeLastMod: true,
      generatedSitemap: '',
      showAdvancedSettings: false,
      generateRobots: false,
      robotsTxt: 'User-agent: *\nAllow: /\nSitemap: {sitemapUrl}',
      createSitemapIndex: false,
      urlsPerSitemap: 10000,
      sitemapNamePattern: 'sitemap_{num}.xml',
      googleVerification: '',
      statusMessage: '',
      statusType: 'success',
      estimatedSize: '',
      urlError: '',
    };
  },
  methods: {
    // Add a new URL entry
    addNewUrl() {
      const today = new Date().toISOString().split('T')[0];
      this.urls.push({
        loc: this.baseUrl ? `${this.baseUrl.replace(/\/+$/, '')}/` : 'https://',
        priority: this.defaultSettings.priority,
        changefreq: this.defaultSettings.changefreq,
        lastmod: today,
        showSettings: false,
        hasImage: false,
        image: {
          url: '',
          caption: ''
        },
        error: ''
      });
    },
    
    // Remove URL at index
    removeUrl(index) {
      this.urls.splice(index, 1);
    },

    // Add exclusion pattern
    addExclusionPattern() {
      this.exclusionPatterns.push('*/example/*');
    },

    // Remove exclusion pattern
    removeExclusionPattern(index) {
      this.exclusionPatterns.splice(index, 1);
    },
    
    // Validate URL
    isValidUrl(url) {
      if (!url) return false;
      try {
        new URL(url);
        return true;
      } catch (e) {
        return false;
      }
    },

    // Validate all URLs and return true if all are valid
    validateUrls() {
      let allValid = true;
      this.urlError = '';
      
      // Validate base URL
      if (this.baseUrl && !this.isValidUrl(this.baseUrl)) {
        this.urlError = 'Please enter a valid base URL (e.g., https://example.com)';
        allValid = false;
      }
      
      // Validate individual URLs
      this.urls.forEach(url => {
        url.error = '';
        if (!this.isValidUrl(url.loc)) {
          url.error = 'Invalid URL format';
          allValid = false;
        }
        
        // Validate image URL if present
        if (url.hasImage && url.image.url && !this.isValidUrl(url.image.url)) {
          url.error = 'Invalid image URL format';
          allValid = false;
        }
      });
      
      return allValid;
    },

    // Fetch URLs from the provided website
    async fetchUrls() {
      if (!this.baseUrl) return;
      if (!this.isValidUrl(this.baseUrl)) {
        this.urlError = 'Please enter a valid URL';
        return;
      }
      
      this.urlError = '';
      this.isLoading = true;
      
      try {
        // In a real implementation, we would make an API call to a backend that can crawl the website
        // For this demo, we'll simulate adding some common pages
        const baseUrl = this.baseUrl.replace(/\/+$/, '');
        
        setTimeout(() => {
          const today = new Date().toISOString().split('T')[0];
          const commonPages = ['', '/about', '/contact', '/blog', '/products', '/services'];
          
          let fetchedUrls = commonPages.map(page => ({
            loc: `${baseUrl}${page}`,
            priority: page === '' ? '1.0' : '0.8',
            changefreq: page === '' ? 'daily' : 'weekly',
            lastmod: today,
            showSettings: false,
            hasImage: false,
            image: {
              url: '',
              caption: ''
            },
            error: ''
          }));
          
          // Filter out URLs that match exclusion patterns
          if (this.exclusionPatterns.length > 0) {
            fetchedUrls = fetchedUrls.filter(url => {
              return !this.exclusionPatterns.some(pattern => {
                // Convert glob pattern to regex
                const regexPattern = pattern
                  .replace(/\*/g, '.*')
                  .replace(/\?/g, '.');
                const regex = new RegExp(regexPattern);
                return regex.test(url.loc);
              });
            });
          }
          
          this.urls = fetchedUrls;
          this.isLoading = false;
          
          this.showStatusMessage('URLs fetched successfully!', 'success');
        }, 1500); // Simulate a delay for the API call
      } catch (error) {
        console.error("Error fetching URLs:", error);
        this.isLoading = false;
        this.showStatusMessage('Error fetching URLs. Please try again.', 'error');
      }
    },

    // Display status message
    showStatusMessage(message, type = 'success') {
      this.statusMessage = message;
      this.statusType = type;
      
      // Clear status after 5 seconds
      setTimeout(() => {
        this.statusMessage = '';
      }, 5000);
    },

    // Calculate estimated file size
    calculateEstimatedSize() {
      // Rough estimation based on XML format
      const avgBytesPerUrl = 200; // Average bytes per URL in XML format
      const imgBytesPerUrl = 150; // Additional bytes for image data
      
      let totalBytes = 200; // XML header and schema declarations
      
      this.urls.forEach(url => {
        let urlBytes = avgBytesPerUrl;
        if (url.hasImage && url.image.url) {
          urlBytes += imgBytesPerUrl;
          if (url.image.caption) {
            urlBytes += url.image.caption.length;
          }
        }
        totalBytes += urlBytes;
      });
      
      // Convert to KB or MB
      if (totalBytes < 1024 * 1024) {
        this.estimatedSize = (totalBytes / 1024).toFixed(2) + ' KB';
      } else {
        this.estimatedSize = (totalBytes / (1024 * 1024)).toFixed(2) + ' MB';
      }
    },

    // Generate sitemap based on input URLs and format selection
    generateSitemap() {
      if (this.urls.length === 0) return;
      
      // Validate URLs first
      if (!this.validateUrls()) {
        this.showStatusMessage('Please fix the invalid URLs before generating.', 'error');
        return;
      }
      
      switch(this.sitemapFormat) {
        case 'xml':
          this.generateXmlSitemap();
          break;
        case 'rss':
          this.generateRssFeed();
          break;
        case 'txt':
          this.generateTextSitemap();
          break;
        case 'html':
          this.generateHtmlSitemap();
          break;
        default:
          this.generateXmlSitemap();
      }
      
      // Calculate estimated file size
      this.calculateEstimatedSize();
      
      // Show success message
      this.showStatusMessage('Sitemap generated successfully!', 'success');
      
      // Generate robots.txt if selected
      if (this.generateRobots) {
        this.generateRobotsTxt();
      }
    },

    // Generate robots.txt file
    generateRobotsTxt() {
      const sitemapUrl = this.baseUrl ? 
        `${this.baseUrl.replace(/\/+$/, '')}/sitemap.xml` : 
        'https://example.com/sitemap.xml';
      
      // Replace placeholder with actual sitemap URL
      let robotsContent = this.robotsTxt.replace('{sitemapUrl}', sitemapUrl);
      
      // We could offer this for download or display separately
      console.log('Generated robots.txt:', robotsContent);
    },
    
    // Generate XML sitemap format
    generateXmlSitemap() {
      if (this.createSitemapIndex && this.urls.length > this.urlsPerSitemap) {
        this.generateSitemapIndex();
        return;
      }
      
      let content = `<?xml version="1.0" encoding="UTF-8"?>\n`;
      
      // Add Google verification if provided
      if (this.googleVerification) {
        content += `<!-- Google Verification: ${this.escapeXml(this.googleVerification)} -->\n`;
      }
      
      // Check if we need to include image namespace
      const hasImages = this.urls.some(url => url.hasImage && url.image.url);
      
      if (hasImages) {
        content += `<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9" 
  xmlns:image="http://www.google.com/schemas/sitemap-image/1.1">\n`;
      } else {
        content += `<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">\n`;
      }
      
      this.urls.forEach(url => {
        content += `  <url>\n`;
        content += `    <loc>${this.escapeXml(url.loc)}</loc>\n`;
        
        if (this.includeLastMod && url.lastmod) {
          content += `    <lastmod>${url.lastmod}</lastmod>\n`;
        }
        
        content += `    <changefreq>${url.changefreq}</changefreq>\n`;
        content += `    <priority>${url.priority}</priority>\n`;
        
        // Add image data if available
        if (url.hasImage && url.image.url) {
          content += `    <image:image>\n`;
          content += `      <image:loc>${this.escapeXml(url.image.url)}</image:loc>\n`;
          if (url.image.caption) {
            content += `      <image:caption>${this.escapeXml(url.image.caption)}</image:caption>\n`;
          }
          content += `    </image:image>\n`;
        }
        
        content += `  </url>\n`;
      });
      
      content += `</urlset>`;
      
      this.generatedSitemap = content;
    },
    
    // Generate sitemap index for large sites
    generateSitemapIndex() {
      const baseUrl = this.baseUrl ? this.baseUrl.replace(/\/+$/, '') : 'https://example.com';
      const totalSitemaps = Math.ceil(this.urls.length / this.urlsPerSitemap);
      
      let content = `<?xml version="1.0" encoding="UTF-8"?>\n`;
      content += `<sitemapindex xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">\n`;
      
      for (let i = 1; i <= totalSitemaps; i++) {
        const sitemapUrl = `${baseUrl}/${this.sitemapNamePattern.replace('{num}', i)}`;
        content += `  <sitemap>\n`;
        content += `    <loc>${this.escapeXml(sitemapUrl)}</loc>\n`;
        content += `    <lastmod>${new Date().toISOString().split('T')[0]}</lastmod>\n`;
        content += `  </sitemap>\n`;
      }
      
      content += `</sitemapindex>`;
      
      this.generatedSitemap = content;
      
      // Show info about splitting sitemaps
      this.showStatusMessage(`Site has ${this.urls.length} URLs. Generated sitemap index with ${totalSitemaps} sitemap files.`, 'success');
    },
    
    // Generate RSS feed format
    generateRssFeed() {
      const now = new Date().toUTCString();
      let content = `<?xml version="1.0" encoding="UTF-8"?>\n`;
      content += `<rss version="2.0">\n`;
      content += `<channel>\n`;
      content += `  <title>Site Map - ${this.baseUrl}</title>\n`;
      content += `  <link>${this.baseUrl}</link>\n`;
      content += `  <description>Site map for ${this.baseUrl}</description>\n`;
      content += `  <pubDate>${now}</pubDate>\n`;
      
      this.urls.forEach(url => {
        content += `  <item>\n`;
        content += `    <title>${this.escapeXml(url.loc)}</title>\n`;
        content += `    <link>${this.escapeXml(url.loc)}</link>\n`;
        content += `    <description>Priority: ${url.priority}</description>\n`;
        
        if (this.includeLastMod && url.lastmod) {
          content += `    <pubDate>${new Date(url.lastmod).toUTCString()}</pubDate>\n`;
        }
        
        content += `  </item>\n`;
      });
      
      content += `</channel>\n`;
      content += `</rss>`;
      
      this.generatedSitemap = content;
    },
    
    // Generate text sitemap format
    generateTextSitemap() {
      let content = '';
      
      this.urls.forEach(url => {
        content += `${url.loc}\n`;
      });
      
      this.generatedSitemap = content;
    },
    
    // Generate HTML sitemap format
    generateHtmlSitemap() {
      let content = `<!DOCTYPE html>\n`;
      content += `<html lang="en">\n`;
      content += `<head>\n`;
      content += `  <meta charset="UTF-8">\n`;
      content += `  <title>Sitemap for ${this.baseUrl}</title>\n`;
      
      // Add Google verification if provided
      if (this.googleVerification) {
        content += `  <meta name="google-site-verification" content="${this.escapeXml(this.googleVerification)}" />\n`;
      }
      
      content += `  <style>\n`;
      content += `    body { font-family: Arial, sans-serif; margin: 0; padding: 20px; }\n`;
      content += `    h1 { color: #333; }\n`;
      content += `    ul { list-style-type: none; padding: 0; }\n`;
      content += `    li { margin: 10px 0; }\n`;
      content += `    a { color: #0066cc; text-decoration: none; }\n`;
      content += `    a:hover { text-decoration: underline; }\n`;
      content += `    .priority { color: #666; font-size: 0.8em; margin-left: 10px; }\n`;
      content += `    .image-info { color: #0aa; font-size: 0.8em; margin-left: 10px; }\n`;
      content += `  </style>\n`;
      content += `</head>\n`;
      content += `<body>\n`;
      content += `  <h1>Sitemap for ${this.baseUrl}</h1>\n`;
      content += `  <ul>\n`;
      
      this.urls.forEach(url => {
        content += `    <li>\n`;
        content += `      <a href="${this.escapeXml(url.loc)}">${this.escapeXml(url.loc)}</a>\n`;
        content += `      <span class="priority">Priority: ${url.priority}</span>\n`;
        
        if (url.hasImage && url.image.url) {
          content += `      <span class="image-info">Has image</span>\n`;
        }
        
        content += `    </li>\n`;
      });
      
      content += `  </ul>\n`;
      content += `</body>\n`;
      content += `</html>`;
      
      this.generatedSitemap = content;
    },
    
    // Download the generated sitemap
    downloadSitemap() {
      if (!this.generatedSitemap) return;
      
      const fileExtension = {
        'xml': 'xml',
        'rss': 'xml',
        'txt': 'txt',
        'html': 'html'
      }[this.sitemapFormat];
      
      const fileName = this.createSitemapIndex ? 'sitemap_index.xml' : `sitemap.${fileExtension}`;
      const blob = new Blob([this.generatedSitemap], { type: 'text/plain' });
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      
      a.href = url;
      a.download = fileName;
      document.body.appendChild(a);
      a.click();
      document.body.removeChild(a);
      URL.revokeObjectURL(url);
      
      this.showStatusMessage(`${fileName} downloaded successfully!`, 'success');
    },
    
    // Copy sitemap to clipboard
    copyToClipboard() {
      if (!this.generatedSitemap) return;
      
      navigator.clipboard.writeText(this.generatedSitemap)
        .then(() => {
          this.showStatusMessage('Sitemap copied to clipboard!', 'success');
        })
        .catch(err => {
          console.error('Error copying to clipboard:', err);
          this.showStatusMessage('Failed to copy to clipboard', 'error');
        });
    },
    
    // Helper to escape XML special characters
    escapeXml(unsafe) {
      if (!unsafe) return '';
      return unsafe
        .replace(/&/g, '&amp;')
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt;')
        .replace(/"/g, '&quot;')
        .replace(/'/g, '&apos;');
    }
  }
};
</script>

<style scoped>
pre {
  white-space: pre-wrap;
  word-wrap: break-word;
  font-family: monospace;
  font-size: 0.9em;
}
</style>
