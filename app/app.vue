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
      rel: "preload",
      as: "video",
      href: "/video/loading.mp4",
      type: "video/mp4",
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
html,
body,
#__nuxt {
  height: 100%;
  margin: 0;
  padding: 0;
  overflow: hidden;
  background-color: #fffbf0;
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
