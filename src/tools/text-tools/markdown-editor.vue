<script setup>
import { ref, computed, watch, onMounted } from 'vue';
import ToolContainer from '../../components/ToolContainer.vue';

const markdownInput = ref('');
const previewMode = ref('split'); // 'split', 'edit', 'preview'
const isDarkMode = ref(false);
const copied = ref(false);

// Sample markdown to help users get started
const sampleMarkdown = `# Markdown Editor

## Basic formatting

**Bold text** and *italic text* are supported.

## Lists

### Unordered List
- Item 1
- Item 2
- Item 3

### Ordered List
1. First item
2. Second item
3. Third item

## Links and Images

[Visit Google](https://google.com)

![Sample Image](https://via.placeholder.com/150)

## Code

Inline \`code\` example.

\`\`\`javascript
// Code block example
function greet(name) {
  return \`Hello, \${name}!\`;
}
\`\`\`

## Blockquotes

> This is a blockquote
> It can span multiple lines

## Tables

| Header 1 | Header 2 |
|----------|----------|
| Cell 1   | Cell 2   |
| Cell 3   | Cell 4   |

## Horizontal Rule

---

`;

// Simple Markdown renderer function
function renderMarkdown(markdown) {
  if (!markdown) return '';
  
  let html = markdown
    // Headers
    .replace(/^### (.*$)/gim, '<h3>$1</h3>')
    .replace(/^## (.*$)/gim, '<h2>$1</h2>')
    .replace(/^# (.*$)/gim, '<h1>$1</h1>')
    
    // Bold & Italic
    .replace(/\*\*(.*?)\*\*/gim, '<strong>$1</strong>')
    .replace(/\*(.*?)\*/gim, '<em>$1</em>')
    
    // Code blocks
    .replace(/```(.*?)\n([\s\S]*?)```/gim, function(match, lang, code) {
      return `<pre class="language-${lang || 'plaintext'}"><code>${code.replace(/</g, '&lt;').replace(/>/g, '&gt;')}</code></pre>`;
    })
    
    // Inline code
    .replace(/`([^`]+)`/gim, '<code>$1</code>')
    
    // Lists
    .replace(/^\s*\n\* (.*)/gim, '<ul>\n<li>$1</li>')
    .replace(/^\* (.*)/gim, '<li>$1</li>')
    .replace(/^\s*\n- (.*)/gim, '<ul>\n<li>$1</li>')
    .replace(/^- (.*)/gim, '<li>$1</li>')
    .replace(/^\s*\n\d+\. (.*)/gim, '<ol>\n<li>$1</li>')
    .replace(/^\d+\. (.*)/gim, '<li>$1</li>')
    
    // Fix lists
    .replace(/<\/ul>\s*<ul>/g, '')
    .replace(/<\/ol>\s*<ol>/g, '')
    
    // Add closing tags for lists
    .replace(/<\/li>\s*(?!<li|<\/ul|<\/ol)/g, '</li></ul>')
    .replace(/<ol>[\s\S]*?<\/li>(?!<li)/g, match => match + '</ol>')
    
    // Blockquotes
    .replace(/^\> (.*$)/gim, '<blockquote>$1</blockquote>')
    .replace(/<\/blockquote><blockquote>/g, '<br>')
    
    // Links
    .replace(/\[([^\]]+)\]\(([^)]+)\)/gim, '<a href="$2" target="_blank">$1</a>')
    
    // Images
    .replace(/!\[([^\]]+)\]\(([^)]+)\)/gim, '<img src="$2" alt="$1" class="max-w-full h-auto">')
    
    // Horizontal rules
    .replace(/^\s*-{3,}\s*$/gim, '<hr>')
    
    // Tables (basic support)
    .replace(/^\|(.+)\|$/gim, function(match, content) {
      const cells = content.split('|').map(cell => cell.trim());
      const isHeader = cells.some(cell => cell.includes('--'));
      
      if (isHeader) {
        return '';
      }
      
      const cellTag = isHeader ? 'th' : 'td';
      const rowContent = cells.map(cell => `<${cellTag}>${cell}</${cellTag}>`).join('');
      return `<tr>${rowContent}</tr>`;
    })
    
    // Paragraphs
    .replace(/\n\n/g, '<br><br>')
    
    // Fix stray tags
    .replace(/<\/blockquote>(?!<blockquote)/g, '</blockquote><br>')
    .replace(/<\/h[1-3]>/g, match => match + '<br>');
    
  // Table wrapper
  if (html.includes('<tr>')) {
    html = html.replace(/(<tr>.+?<\/tr>)+/g, '<table class="border-collapse border border-gray-300">$&</table>');
  }
  
  return html;
}

const renderedMarkdown = computed(() => {
  return renderMarkdown(markdownInput.value);
});

function loadSample() {
  markdownInput.value = sampleMarkdown;
}

function copyHtml() {
  navigator.clipboard.writeText(renderedMarkdown.value);
  copied.value = true;
  
  setTimeout(() => {
    copied.value = false;
  }, 2000);
}

function handleTab(e) {
  if (e.key === 'Tab') {
    e.preventDefault();
    
    const start = e.target.selectionStart;
    const end = e.target.selectionEnd;
    
    markdownInput.value = markdownInput.value.substring(0, start) + '  ' + markdownInput.value.substring(end);
    
    // Move cursor position
    e.target.selectionStart = e.target.selectionEnd = start + 2;
  }
}

// Save markdown content to localStorage
watch(markdownInput, (newValue) => {
  if (newValue) {
    localStorage.setItem('markdown-editor-content', newValue);
  }
});

onMounted(() => {
  // Load saved content if it exists
  const savedContent = localStorage.getItem('markdown-editor-content');
  if (savedContent) {
    markdownInput.value = savedContent;
  }
});
</script>

<template>
  <ToolContainer 
    title="Markdown Editor" 
    description="Write and preview Markdown with real-time rendering."
  >
    <div class="space-y-4">
      <div class="flex justify-between items-center">
        <div class="flex space-x-2">
          <button
            @click="previewMode = 'edit'"
            class="px-3 py-1 text-sm rounded-md"
            :class="previewMode === 'edit' 
              ? 'bg-indigo-100 text-indigo-700' 
              : 'bg-gray-100 text-gray-700 hover:bg-gray-200'"
          >
            Edit
          </button>
          <button
            @click="previewMode = 'split'"
            class="px-3 py-1 text-sm rounded-md"
            :class="previewMode === 'split' 
              ? 'bg-indigo-100 text-indigo-700' 
              : 'bg-gray-100 text-gray-700 hover:bg-gray-200'"
          >
            Split
          </button>
          <button
            @click="previewMode = 'preview'"
            class="px-3 py-1 text-sm rounded-md"
            :class="previewMode === 'preview' 
              ? 'bg-indigo-100 text-indigo-700' 
              : 'bg-gray-100 text-gray-700 hover:bg-gray-200'"
          >
            Preview
          </button>
        </div>
        
        <div class="flex space-x-2">
          <button
            @click="loadSample"
            class="px-3 py-1 text-sm bg-gray-100 text-gray-700 rounded-md hover:bg-gray-200"
          >
            Load Sample
          </button>
          
          <button
            @click="copyHtml"
            class="px-3 py-1 text-sm bg-gray-100 hover:bg-gray-200 rounded-md flex items-center"
            :class="{ 'bg-green-100 text-green-800': copied }"
          >
            <svg v-if="copied" xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1" viewBox="0 0 20 20" fill="currentColor">
              <path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd" />
            </svg>
            <svg v-else xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1" viewBox="0 0 20 20" fill="currentColor">
              <path d="M8 3a1 1 0 011-1h2a1 1 0 110 2H9a1 1 0 01-1-1z" />
              <path d="M6 3a2 2 0 00-2 2v11a2 2 0 002 2h8a2 2 0 002-2V5a2 2 0 00-2-2 3 3 0 01-3 3H9a3 3 0 01-3-3z" />
            </svg>
            {{ copied ? 'Copied HTML!' : 'Copy HTML' }}
          </button>
          
          <label class="inline-flex items-center px-3 py-1 bg-gray-100 text-gray-700 rounded-md hover:bg-gray-200 cursor-pointer">
            <input type="checkbox" v-model="isDarkMode" class="form-checkbox text-indigo-600 mr-1 h-4 w-4">
            <span class="text-sm">Dark Mode</span>
          </label>
        </div>
      </div>
      
      <div class="flex flex-col md:flex-row gap-4">
        <div v-if="previewMode !== 'preview'" 
          class="w-full md:w-1/2 border rounded-md shadow-sm"
          :class="{ 'bg-gray-900': isDarkMode }"
        >
          <div class="border-b px-3 py-2 text-sm font-medium"
               :class="{ 'bg-gray-800 text-white border-gray-700': isDarkMode, 'bg-gray-50': !isDarkMode }">
            Markdown
          </div>
          <textarea
            v-model="markdownInput"
            @keydown="handleTab"
            class="w-full px-3 py-2 font-mono text-sm h-[600px] focus:outline-none"
            :class="{ 'bg-gray-800 text-gray-200': isDarkMode, 'bg-white': !isDarkMode }"
            placeholder="# Start writing your Markdown here..."
          ></textarea>
        </div>
        
        <div v-if="previewMode !== 'edit'" 
          class="w-full md:w-1/2 border rounded-md shadow-sm"
          :class="{ 'bg-gray-900 border-gray-700': isDarkMode }"
        >
          <div class="border-b px-3 py-2 text-sm font-medium"
               :class="{ 'bg-gray-800 text-white border-gray-700': isDarkMode, 'bg-gray-50': !isDarkMode }">
            Preview
          </div>
          <div 
            class="px-4 py-3 overflow-auto h-[600px] prose max-w-none"
            :class="{ 'prose-invert': isDarkMode }"
            v-html="renderedMarkdown">
          </div>
        </div>
      </div>
    </div>
    
    <template #instructions>
      <ol class="list-decimal list-inside space-y-2 text-gray-600">
        <li>Write your Markdown in the editor on the left</li>
        <li>See the rendered preview on the right in real-time</li>
        <li>Toggle between Edit, Split, and Preview modes using the buttons above</li>
        <li>Use the "Load Sample" button to see Markdown examples</li>
        <li>Click "Copy HTML" to copy the rendered HTML to your clipboard</li>
        <li>Switch to Dark Mode for low-light environments</li>
      </ol>
    </template>
  </ToolContainer>
</template>

<style scoped>
/* Custom styling for the markdown preview */
:deep(.prose) {
  font-size: 0.9rem;
}

:deep(.prose h1) {
  font-size: 1.8rem;
  margin-top: 0;
  margin-bottom: 1rem;
}

:deep(.prose h2) {
  font-size: 1.5rem;
  margin-top: 1.5rem;
  margin-bottom: 0.75rem;
}

:deep(.prose h3) {
  font-size: 1.25rem;
  margin-top: 1.25rem;
  margin-bottom: 0.5rem;
}

:deep(.prose code) {
  background-color: #f3f4f6;
  padding: 0.2em 0.4em;
  border-radius: 0.25rem;
  font-size: 0.85em;
}

:deep(.prose-invert code) {
  background-color: #374151;
}

:deep(.prose pre) {
  background-color: #f3f4f6;
  padding: 1rem;
  border-radius: 0.375rem;
  overflow-x: auto;
}

:deep(.prose-invert pre) {
  background-color: #374151;
}

:deep(.prose blockquote) {
  border-left: 4px solid #d1d5db;
  padding-left: 1rem;
  font-style: italic;
}

:deep(.prose-invert blockquote) {
  border-left-color: #4b5563;
}

:deep(.prose table) {
  border-collapse: collapse;
  width: 100%;
}

:deep(.prose th),
:deep(.prose td) {
  border: 1px solid #d1d5db;
  padding: 0.5rem;
}

:deep(.prose-invert th),
:deep(.prose-invert td) {
  border-color: #4b5563;
}
</style>
