<script setup lang="ts">
import { ref, onMounted } from 'vue'
import WeddingMain from './components/WeddingMain.vue'
import GlobalLoading from './components/GlobalLoading.vue'

// State
const isLoading = ref(true)
const loadProgress = ref(0)

// Danh sách các ảnh cần preload (đảm bảo đường dẫn chính xác)
const assetsToPreload = [
  '/images/cover-anime.jpg',
  '/images/cover.jpg',
  '/images/title.png',
  'https://www.transparenttextures.com/patterns/cream-paper.png' // Texture giấy
]

// Hàm preload một ảnh
const preloadImage = (src: string) => {
  return new Promise((resolve, reject) => {
    const img = new Image()
    img.src = src
    img.onload = resolve
    img.onerror = resolve // Vẫn resolve dù lỗi để không treo app
  })
}

onMounted(async () => {
  const totalAssets = assetsToPreload.length
  let loadedCount = 0
  
  // Thời gian tối thiểu loading (để người dùng kịp nhìn thấy hiệu ứng đẹp, tránh nháy màn hình)
  const minLoadTime = new Promise(resolve => setTimeout(resolve, 2000))

  // Bắt đầu tải tất cả ảnh song song
  const imagePromises = assetsToPreload.map(async (src) => {
    await preloadImage(src)
    loadedCount++
    // Cập nhật tiến trình (nhân 90% để dành 10% cuối cho animation mượt)
    loadProgress.value = Math.floor((loadedCount / totalAssets) * 90)
  })

  // Đợi tải xong ảnh VÀ đợi đủ thời gian tối thiểu
  await Promise.all([Promise.all(imagePromises), minLoadTime])

  // Đẩy lên 100% và tắt loading
  loadProgress.value = 100
  setTimeout(() => {
    isLoading.value = false
  }, 500) // Delay nhẹ 0.5s ở mức 100% rồi mới ẩn
})
</script>

<template>
  <div>
    <Transition name="fade">
      <GlobalLoading v-if="isLoading" :progress="loadProgress" />
    </Transition>

    <div v-show="!isLoading">
      <WeddingMain />
    </div>
  </div>
</template>

<style>
/* CSS Global cho Transition Fade */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.8s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>