<script setup lang="ts">
import { ref, computed } from 'vue'
// Import 2 component giao diện
import WeddingCartoon from './WeddingCartoon.vue'
import WeddingFormal from './WeddingFormal.vue'

// State để lưu style hiện tại ('cartoon' hoặc 'formal')
const currentStyle = ref<'cartoon' | 'formal'>('cartoon') // Mặc định là cartoon

// Computed property để xác định component nào sẽ được render
const currentComponent = computed(() => {
    return currentStyle.value === 'cartoon' ? WeddingCartoon : WeddingFormal
})

// Hàm xử lý khi chọn dropdown
const handleStyleChange = (event: Event) => {
    const target = event.target as HTMLSelectElement
    currentStyle.value = target.value as 'cartoon' | 'formal'
}
</script>

<template>
  <div class="relative w-full h-full">
    
    <div class="fixed top-4 left-4 z-9999">
        <div class="relative inline-block text-left">
            <select 
                :value="currentStyle"
                @change="handleStyleChange"
                class="appearance-none bg-black/50 backdrop-blur-md text-white/90 border border-white/20 rounded-full px-4 py-2 pr-8 text-sm font-medium hover:bg-black/70 focus:outline-none focus:ring-2 focus:ring-[#FFD700]/50 cursor-pointer transition-all"
            >
                <option value="cartoon" class="text-black">Phong cách Hoạt hình</option>
                <option value="formal" class="text-black">Phong cách Trang trọng</option>
            </select>
            
            <div class="pointer-events-none absolute inset-y-0 right-0 flex items-center px-2 text-white/70">
                <svg class="fill-current h-4 w-4" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20"><path d="M9.293 12.95l.707.707L15.657 8l-1.414-1.414L10 10.828 5.757 6.586 4.343 8z"/></svg>
            </div>
        </div>
    </div>

    <Transition name="fade" mode="out-in">
        <component :is="currentComponent" :key="currentStyle" />
    </Transition>

  </div>
</template>

<style scoped>
/* Hiệu ứng fade nhẹ khi chuyển component */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>