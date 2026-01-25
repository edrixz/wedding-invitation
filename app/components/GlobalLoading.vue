<template>
  <div class="fixed inset-0 z-99999 flex flex-col items-center justify-center bg-white text-[#3E2723]">
    
    <div class="relative mb-4 w-40 h-40 md:w-56 md:h-56 flex items-center justify-center">
        <video 
            src="/video/loading.mp4" 
            autoplay 
            loop 
            muted 
            playsinline
            class="w-full h-full object-contain mix-blend-multiply"
        ></video>
    </div>

    <h2 class="font-serif text-lg md:text-xl font-bold tracking-widest mb-2 text-[#5d4037] animate-pulse">
      ĐANG TẢI...
    </h2>

    <div class="w-64 h-1.5 bg-gray-100 rounded-full overflow-hidden border border-gray-200">
      <div class="h-full bg-[#B71C1C] transition-all duration-300 ease-out" :style="{ width: progress + '%' }"></div>
    </div>
    
    <p class="mt-2 text-xs font-bold text-[#880E4F]">{{ progress }}%</p>

  </div>
</template>

<script setup lang="ts">
defineProps<{
  progress: number
}>()
</script>

<style scoped>
/* Không cần CSS animation tim đập nữa vì đã có video */
</style>