<script setup lang="ts">
import { ref, onMounted } from "vue";

defineProps<{
  progress: number;
}>();

const isVideoPlaying = ref(false);
const showThumb = ref(false);
const videoRef = ref<HTMLVideoElement | null>(null);

const onVideoCanPlay = () => {
  isVideoPlaying.value = true;
};

onMounted(() => {
  if (videoRef.value && videoRef.value.readyState >= 3) {
    isVideoPlaying.value = true;
  } else {
    setTimeout(() => {
      if (!isVideoPlaying.value) {
        showThumb.value = true;
      }
    }, 200);
  }
});
</script>

<template>
  <div
    class="absolute inset-0 z-99999 flex flex-col items-center justify-center bg-white text-[#3E2723]"
  >
    <div
      class="relative mb-4 w-40 h-40 md:w-56 md:h-56 flex items-center justify-center"
    >
      <div
        v-if="!isVideoPlaying && showThumb"
        class="absolute inset-0 flex items-center justify-center z-20"
      >
        <img
          src="/images/loading-thumb.jpg"
          class="w-16 h-16 md:w-20 md:h-20 object-contain animate-pulse"
          alt="Loading..."
        />
      </div>

      <video
        ref="videoRef"
        src="/video/loading.mp4"
        autoplay
        loop
        muted
        playsinline
        @canplay="onVideoCanPlay"
        class="w-full h-full object-contain mix-blend-multiply relative z-10 transition-opacity duration-500"
        :class="{ 'opacity-0': !isVideoPlaying, 'opacity-100': isVideoPlaying }"
      ></video>
    </div>

    <h2
      class="font-serif text-lg md:text-xl font-bold tracking-widest mb-2 text-[#5d4037] animate-pulse"
    >
      ĐANG TẢI...
    </h2>

    <div
      class="w-64 h-1.5 bg-gray-100 rounded-full overflow-hidden border border-gray-200"
    >
      <div
        class="h-full bg-[#B71C1C] transition-all duration-300 ease-out"
        :style="{ width: progress + '%' }"
      ></div>
    </div>

    <p class="mt-2 text-xs font-bold text-[#880E4F]">{{ progress }}%</p>
  </div>
</template>
