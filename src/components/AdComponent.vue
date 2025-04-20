<template>
  <div class="ad-container my-6 text-center">
    <!-- Responsive Ad -->
    <ins class="adsbygoogle"
         style="display:block"
         data-ad-client="ca-pub-9354746037074515"
         data-ad-slot="7755725804"
         data-ad-format="auto"
         data-full-width-responsive="true"></ins>
  </div>
</template>

<script setup>
import { onMounted } from 'vue';

onMounted(() => {
  try {
    // Only push ads if adsense is loaded
    if (window.adsbygoogle) {
      (window.adsbygoogle = window.adsbygoogle || []).push({});
    }
  } catch (error) {
    console.error('AdSense error:', error);
  }
});
</script>

<style scoped>
.ad-container {
  min-height: 100px; /* Prevent layout shift */
  width: 100%;
  overflow: hidden;
  background-color: #f9fafb; /* Light gray background to indicate ad area */
  border-radius: 0.375rem;
}
</style>
