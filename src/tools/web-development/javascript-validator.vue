<script setup>
import { ref } from 'vue';
import ToolContainer from '../../components/ToolContainer.vue';

const jsCode = ref('');
const validationResult = ref(null);
const isLoading = ref(false);

function validateJS() {
  if (!jsCode.value.trim()) {
    validationResult.value = null;
    return;
  }
  
  isLoading.value = true;
  try {
    // Try to parse the JavaScript code using Function constructor
    // This will catch syntax errors
    new Function(jsCode.value);
    
    // If we get here, the code is syntactically valid
    validationResult.value = {
      valid: true,
      message: 'JavaScript is valid! No syntax errors detected.'
    };
    
    // Basic checks for common issues
    const warnings = [];
    
    // Check for console.log statements
    if (jsCode.value.includes('console.log')) {
      warnings.push('Console statements found (e.g., console.log). Consider removing them for production.');
    }
    
    // Check for alert statements
    if (jsCode.value.includes('alert(')) {
      warnings.push('Alert statements found. Consider using a more user-friendly notification system.');
    }
    
    // Check for potential undefined variables (very basic check)
    if (jsCode.value.match(/\b(undefined|null)\b/)) {
      warnings.push('Code contains explicit undefined or null values. Make sure these are handled properly.');
    }
    
    // Check for eval usage
    if (jsCode.value.includes('eval(')) {
      warnings.push('Eval statements found. Eval is potentially dangerous and should be avoided.');
    }
    
    if (warnings.length > 0) {
      validationResult.value.warnings = warnings;
    }
    
  } catch (error) {
    // If an error occurs, the code is invalid
    validationResult.value = {
      valid: false,
      message: 'JavaScript has syntax errors:',
      error: error.message
    };
  } finally {
    isLoading.value = false;
  }
}
</script>

<template>
  <ToolContainer 
    title="JavaScript Validator" 
    description="Check your JavaScript code for syntax errors and basic issues."
  >
    <div class="space-y-6">
      <div>
        <label for="js-code" class="block text-sm font-medium text-gray-700 mb-2">
          Enter your JavaScript code:
        </label>
        <textarea
          id="js-code"
          v-model="jsCode"
          rows="12"
          class="block w-full font-mono rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500"
          placeholder="// Your JavaScript code here
function example() {
  const greeting = 'Hello, world!';
  console.log(greeting);
  return greeting;
}"
        ></textarea>
      </div>
      
      <div class="flex justify-center">
        <button
          @click="validateJS"
          class="px-4 py-2 bg-indigo-600 text-white rounded-md hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2"
          :disabled="isLoading"
        >
          <template v-if="isLoading">
            <svg class="animate-spin -ml-1 mr-2 h-4 w-4 text-white inline-block" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
              <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
            </svg>
            Validating...
          </template>
          <template v-else>
            Validate JavaScript
          </template>
        </button>
      </div>
      
      <div v-if="validationResult" :class="[
        'p-4 rounded-md',
        validationResult.valid ? 'bg-green-50 border border-green-200' : 'bg-red-50 border border-red-200'
      ]">
        <div class="flex">
          <div class="flex-shrink-0">
            <template v-if="validationResult.valid">
              <svg class="h-5 w-5 text-green-400" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clip-rule="evenodd" />
              </svg>
            </template>
            <template v-else>
              <svg class="h-5 w-5 text-red-400" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20" fill="currentColor">
                <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd" />
              </svg>
            </template>
          </div>
          <div class="ml-3">
            <h3 class="text-sm font-medium" :class="validationResult.valid ? 'text-green-800' : 'text-red-800'">
              {{ validationResult.message }}
            </h3>
            <div v-if="validationResult.error" class="mt-2 text-sm text-red-700 font-mono whitespace-pre-wrap">
              {{ validationResult.error }}
            </div>
            <div v-if="validationResult.warnings && validationResult.warnings.length > 0" class="mt-4">
              <p class="text-sm text-amber-700 font-medium mb-2">Warnings:</p>
              <ul class="list-disc pl-5 space-y-1">
                <li v-for="(warning, index) in validationResult.warnings" :key="index" class="text-sm text-amber-700">
                  {{ warning }}
                </li>
              </ul>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <template #instructions>
      <ol class="list-decimal list-inside space-y-2 text-gray-600">
        <li>Enter or paste your JavaScript code into the input field</li>
        <li>Click the "Validate JavaScript" button</li>
        <li>Check the results for syntax errors</li>
        <li>Review any warnings about potential issues in your code</li>
        <li>Note: This validator checks for syntax errors only; it doesn't check for logical errors or run the code</li>
      </ol>
    </template>
  </ToolContainer>
</template>
