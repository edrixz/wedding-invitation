<script setup lang="ts">
import { ref, onMounted, computed } from "vue";
import { useHead } from "#imports";
import GlobalLoading from "./components/GlobalLoading.vue"; // Giữ lại Loading

// 1. Preload Video & Fonts
useHead({
  title: "Phương Huyền & Văn Hiếu - Wedding Invitation",
  meta: [
    {
      name: "description",
      content: "Trân trọng kính mời tới dự lễ cưới của chúng tôi.",
    },
    {
      name: "viewport",
      content:
        "width=device-width, initial-scale=1, maximum-scale=1, user-scalable=0",
    }, // Chặn zoom trên mobile
  ],
  link: [
    {
      rel: "preload",
      as: "video",
      href: "/video/loading.mp4",
      type: "video/mp4",
    },
    // Preload Font quan trọng để tránh layout shift
    {
      rel: "stylesheet",
      href: "https://fonts.googleapis.com/css2?family=Cinzel:wght@400;700&family=Great+Vibes&family=Cormorant+Garamond:ital,wght@0,400;0,600;1,400;1,600&display=swap",
    },
  ],
});

// CẤU HÌNH THỜI GIAN
const MIN_LOADING_TIME = 2500;

// State
const isLoading = ref(true);
const rawProgress = ref(0);

const loadProgress = computed(() => Math.floor(rawProgress.value));

// CẬP NHẬT LẠI DANH SÁCH ẢNH CHO ĐÚNG VỚI COMPONENT MỚI
const assetsToPreload = [
  "/images/loading-thumb.jpg",
  "/images/cover-anime.jpg",
  "/images/cover.jpg",
  "/images/title.png",
  "https://www.transparenttextures.com/patterns/cream-paper.png",
];

const preloadImage = (src: string) => {
  return new Promise((resolve) => {
    const img = new Image();
    img.src = src;
    img.onload = resolve;
    img.onerror = resolve; // Lỗi vẫn cho qua để không kẹt loading
  });
};

onMounted(async () => {
  // --- A. LOGIC TĂNG TIẾN TRÌNH GIẢ LẬP ---
  const intervalTime = 30;
  const totalSteps = MIN_LOADING_TIME / intervalTime;
  const stepIncrement = 90 / totalSteps;

  const progressInterval = setInterval(() => {
    const nextValue = rawProgress.value + stepIncrement;
    if (nextValue >= 90) {
      rawProgress.value = 90;
    } else {
      rawProgress.value = nextValue;
    }
  }, intervalTime);

  // --- B. LOGIC TẢI TÀI NGUYÊN THẬT ---
  const timerPromise = new Promise((resolve) =>
    setTimeout(resolve, MIN_LOADING_TIME),
  );
  const assetPromises = assetsToPreload.map((src) => preloadImage(src));

  // Đợi cả 2
  await Promise.all([Promise.all(assetPromises), timerPromise]);

  // --- C. KẾT THÚC ---
  clearInterval(progressInterval);
  rawProgress.value = 100;

  setTimeout(() => {
    isLoading.value = false;
  }, 500);
});
</script>

<template>
  <div class="antialiased">
    <Transition name="fade" mode="out-in">
      <GlobalLoading v-if="isLoading" :progress="loadProgress" />
    </Transition>

    <div v-show="!isLoading">
      <NuxtPage />
    </div>
  </div>
</template>

<style>
/* CSS Reset cơ bản cho full height */
html,
body,
#__nuxt {
  height: 100%;
  margin: 0;
  padding: 0;
  overflow: hidden; /* Tránh cuộn thừa khi có hiệu ứng bay nhảy */
  background-color: #fffbf0; /* Màu nền trùng với thiệp để tránh nháy trắng */
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.8s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
