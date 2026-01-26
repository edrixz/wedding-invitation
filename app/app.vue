<script setup lang="ts">
import { ref, onMounted, computed } from "vue";
import { useHead } from "#imports";
import WeddingMain from "./components/WeddingMain.vue";
import GlobalLoading from "./components/GlobalLoading.vue";

// 1. Preload Video
useHead({
  link: [
    {
      rel: "preload",
      as: "video",
      href: "/video/loading.mp4",
      type: "video/mp4",
    },
  ],
});

// CẤU HÌNH THỜI GIAN
const MIN_LOADING_TIME = 2500;

// State
const isLoading = ref(true);
const rawProgress = ref(0); // Giá trị thực (có thể là số lẻ)

// Computed để làm tròn số khi truyền vào component (tránh hiển thị 12.345%)
const loadProgress = computed(() => Math.floor(rawProgress.value));

const assetsToPreload = [
  "/images/loading-thumb.jpg",
  "/images/cover-anime.jpg",
  "/images/cover-real.jpg",
  "/images/title.png",
  "https://www.transparenttextures.com/patterns/cream-paper.png",
];

const preloadImage = (src: string) => {
  return new Promise((resolve) => {
    const img = new Image();
    img.src = src;
    img.onload = resolve;
    img.onerror = resolve;
  });
};

onMounted(async () => {
  // --- A. LOGIC TĂNG TIẾN TRÌNH TỰ ĐỘNG (FAKE PROGRESS) ---
  // Mục tiêu: Tăng từ 0 -> 90% trong khoảng MIN_LOADING_TIME
  const intervalTime = 30; // Cập nhật mỗi 30ms cho mượt
  const totalSteps = MIN_LOADING_TIME / intervalTime;
  const stepIncrement = 90 / totalSteps; // Mỗi bước tăng bao nhiêu %

  const progressInterval = setInterval(() => {
    // Tăng dần
    const nextValue = rawProgress.value + stepIncrement;

    // Chặn lại ở 90% (hoặc 95%) nếu chưa tải xong thật
    if (nextValue >= 90) {
      rawProgress.value = 90;
    } else {
      rawProgress.value = nextValue;
    }
  }, intervalTime);

  // --- B. LOGIC TẢI TÀI NGUYÊN THẬT (REAL LOADING) ---
  const timerPromise = new Promise((resolve) =>
    setTimeout(resolve, MIN_LOADING_TIME),
  );

  const assetPromises = assetsToPreload.map((src) => preloadImage(src));

  // Đợi cả Timer và việc Tải ảnh xong
  await Promise.all([Promise.all(assetPromises), timerPromise]);

  // --- C. KẾT THÚC ---
  // Dọn dẹp bộ đếm
  clearInterval(progressInterval);

  // Nhảy vọt lên 100% (Xong hẳn)
  rawProgress.value = 100;

  setTimeout(() => {
    isLoading.value = false;
  }, 500);
});
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
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.8s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
