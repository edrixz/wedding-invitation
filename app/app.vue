<script setup lang="ts">
import { ref, onMounted, computed } from "vue";
import { useHead } from "#imports";
import GlobalLoading from "./components/GlobalLoading.vue";

useHead({
  title: "Wedding Invitation",
  meta: [
    {
      name: "description",
      content: "Trân trọng kính mời tới dự lễ cưới của chúng tôi.",
    },
    {
      name: "viewport",
      content:
        "width=device-width, initial-scale=1, maximum-scale=1, user-scalable=0",
    },
  ],
  link: [
    {
      rel: "icon",
      type: "image/svg+xml",
      href: "data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 100 100%22><text y=%22.9em%22 font-size=%2290%22>💍</text></svg>",
    },
    {
      rel: "preload",
      as: "video",
      href: "/video/loading.mp4",
      type: "video/mp4",
    },
    {
      rel: "stylesheet",
      href: "https://fonts.googleapis.com/css2?family=Cinzel:wght@400;700&family=Great+Vibes&family=Cormorant+Garamond:ital,wght@0,400;0,600;1,400;1,600&display=swap",
    },
  ],
});

const MIN_LOADING_TIME = 2500;
const isLoading = ref(true);
const rawProgress = ref(0);
const loadProgress = computed(() => Math.floor(rawProgress.value));

const assetsToPreload = [
  "/images/loading-thumb.jpg",
  "/images/cover-anime.jpg",
  "/images/cover.jpg",
  "/images/title.png",
  "/images/404.png",
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
  const intervalTime = 30;
  const totalSteps = MIN_LOADING_TIME / intervalTime;
  const stepIncrement = 90 / totalSteps;

  const progressInterval = setInterval(() => {
    const nextValue = rawProgress.value + stepIncrement;
    if (nextValue >= 90) rawProgress.value = 90;
    else rawProgress.value = nextValue;
  }, intervalTime);

  const timerPromise = new Promise((resolve) =>
    setTimeout(resolve, MIN_LOADING_TIME),
  );
  const assetPromises = assetsToPreload.map((src) => preloadImage(src));

  await Promise.all([Promise.all(assetPromises), timerPromise]);

  clearInterval(progressInterval);
  rawProgress.value = 100;
  setTimeout(() => {
    isLoading.value = false;
  }, 500);
});
</script>

<template>
  <div
    class="antialiased w-full h-dvh bg-zinc-900 flex justify-center items-center overflow-hidden"
  >
    <div
      class="relative w-full max-w-97.5 h-full bg-[#FFFBF0] shadow-2xl overflow-hidden"
    >
      <Transition name="fade" mode="out-in">
        <GlobalLoading v-if="isLoading" :progress="loadProgress" />
      </Transition>

      <div v-show="!isLoading" class="h-full w-full">
        <NuxtPage />
      </div>
    </div>
  </div>
</template>

<style>
/* Reset css */
html,
body,
#__nuxt {
  height: 100%;
  margin: 0;
  padding: 0;
  overflow: hidden;
  background-color: #18181b; /* Màu nền zinc-900 */
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
