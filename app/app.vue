<script setup lang="ts">
import { ref, onMounted } from 'vue'
import WeddingMain from './components/WeddingMain.vue'
import GlobalLoading from './components/GlobalLoading.vue'

// CẤU HÌNH: Thời gian chờ tối thiểu (Miliseconds)
// 2000 = 2 giây. Bạn sửa số này nếu muốn lâu hơn.
const MIN_LOADING_TIME = 2000 

// State
const isLoading = ref(true)
const loadProgress = ref(0)

// Danh sách các file cần tải trước
const assetsToPreload = [
  '/images/cover-anime.jpg',
  '/images/cover-real.jpg',
  '/images/title.png',
  '/video/loading.mp4', // Preload luôn cả video loading nếu cần
  'https://www.transparenttextures.com/patterns/cream-paper.png'
]

const preloadImage = (src: string) => {
  return new Promise((resolve) => {
    // Nếu là video thì bỏ qua hoặc xử lý riêng, ở đây ta giả định preload ảnh
    if (src.endsWith('.mp4')) {
        resolve(true)
        return
    }
    const img = new Image()
    img.src = src
    img.onload = resolve
    img.onerror = resolve 
  })
}

onMounted(async () => {
  const totalAssets = assetsToPreload.length
  let loadedCount = 0
  
  // 1. Tạo Promise đếm ngược thời gian tối thiểu
  const timerPromise = new Promise(resolve => setTimeout(resolve, MIN_LOADING_TIME))

  // 2. Tạo Promise tải tài nguyên
  const assetPromises = assetsToPreload.map(async (src) => {
    await preloadImage(src)
    loadedCount++
    // Chỉ hiển thị tiến trình tối đa 90% khi đang tải
    // Để dành 10% cuối cho hiệu ứng mượt mà khi kết thúc
    loadProgress.value = Math.floor((loadedCount / totalAssets) * 90)
  })

  // 3. Đợi CẢ HAI việc trên cùng xong
  // (Dù ảnh tải xong trong 0.1s, nó vẫn phải đợi timerPromise đủ 2s)
  await Promise.all([Promise.all(assetPromises), timerPromise])

  // 4. Khi đã xong tất cả -> Đẩy lên 100%
  loadProgress.value = 100
  
  // Delay nhẹ 0.5s ở trạng thái 100% cho người dùng nhìn thấy "100%"
  setTimeout(() => {
    isLoading.value = false
  }, 500)
})
</script>

<template>
  <div>
    <Transition name="fade" mode="out-in">
      <GlobalLoading v-if="isLoading" :progress="loadProgress" />
    </Transition>

    <div v-show="!isLoading">
      <WeddingMain />
    </div>
  </div>
</template>

<style>
/* CSS Fade mượt mà */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.8s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>