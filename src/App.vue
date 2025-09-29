<template>
  <div class="fullscreen">
    <ImageCarousel :images="images" />
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import ImageCarousel from './components/ImageCarousel.vue'

const images = ref(
  Array.from({ length: 30 }, (_, i) => ({
    url: `https://picsum.photos/seed/${i+1}/600/400`
  }))
)
</script>

<style>
.fullscreen {
  width: 100vw;
  height: 100vh;
  min-height: 100vh;
  min-width: 100vw;
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  overflow: hidden;
  background: #f5f8fa;
  display: flex;
  align-items: center;
  justify-content: center;
}
body, html {
  margin: 0;
  padding: 0;
  width: 100vw;
  height: 100vh;
  box-sizing: border-box;
  overflow: hidden;
}
</style>