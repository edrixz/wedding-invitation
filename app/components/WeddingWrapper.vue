<script setup lang="ts">
import { ref, computed } from "vue";
import WeddingCartoon from "./WeddingCartoon.vue";
import WeddingFormal from "./WeddingFormal.vue";

// 1. Nhận thông tin phe nào (bride/groom)
const props = defineProps<{
  side: "bride" | "groom";
}>();

// 2. Quản lý Style (Mặc định Formal)
const currentStyle = ref("formal");
const isSwitching = ref(false);
const isCartoon = computed(() => currentStyle.value === "cartoon");

// 3. Hàm chuyển đổi Style (Có preload ảnh)
const toggleStyle = async () => {
  if (isSwitching.value) return;

  const nextStyle = isCartoon.value ? "formal" : "cartoon";
  isSwitching.value = true;

  try {
    const img = new Image();
    img.src =
      nextStyle === "formal" ? "/images/cover.jpg" : "/images/cover-anime.jpg";
    await img.decode(); // Đợi giải mã ảnh xong để tránh giật
  } catch (e) {
    console.warn("Preload failed", e);
  }

  currentStyle.value = nextStyle;
  isSwitching.value = false;
};
</script>

<template>
  <div class="relative w-full h-full">
    <div class="fixed top-4 left-4 z-9999">
      <button
        @click="toggleStyle"
        :disabled="isSwitching"
        class="group relative flex items-center cursor-pointer outline-none select-none w-14 h-8 rounded-full transition-all duration-500 ease-in-out"
        :class="[
          !isCartoon
            ? 'bg-red-950/80 border border-yellow-700/40 backdrop-blur-sm shadow-sm'
            : 'bg-[#FFD54F] border-2 border-[#3E2723] shadow-[2px_2px_0px_#3E2723]',
        ]"
      >
        <div
          v-if="isSwitching"
          class="absolute inset-0 flex items-center justify-center z-20"
        >
          <div
            class="w-3 h-3 border-2 rounded-full animate-spin border-white/30 border-t-white"
            :class="isCartoon ? 'border-amber-900/30 border-t-amber-900' : ''"
          ></div>
        </div>

        <div
          class="absolute top-1/2 -translate-y-1/2 rounded-full transition-all duration-500 cubic-bezier flex items-center justify-center z-10 shadow-sm"
          :class="[
            !isCartoon
              ? 'left-1 w-6 h-6 bg-linear-to-br from-yellow-50 to-yellow-600 border border-white/20'
              : 'left-[calc(100%-1.6rem)] w-5 h-5 bg-white border-2 border-[#3E2723]',
          ]"
        >
          <div
            class="transform transition-transform duration-500"
            :class="isCartoon ? 'rotate-12 scale-90' : 'scale-75'"
          >
            <span v-if="isCartoon" class="text-[10px] leading-none">🐣</span>
            <svg
              v-else
              xmlns="http://www.w3.org/2000/svg"
              fill="currentColor"
              viewBox="0 0 24 24"
              class="w-3 h-3 text-red-900"
            >
              <path
                d="M11.645 20.91l-.007-.003-.022-.012a15.247 15.247 0 01-.383-.218 25.18 25.18 0 01-4.244-3.17C4.688 15.36 2.25 12.174 2.25 8.25 2.25 5.322 4.714 3 7.688 3A5.5 5.5 0 0112 5.052 5.5 5.5 0 0116.313 3c2.973 0 5.437 2.322 5.437 5.25 0 3.925-2.438 7.111-4.739 9.256a25.175 25.175 0 01-4.244 3.17 15.247 15.247 0 01-.383.219l-.022.012-.007.004-.003.001a.752.752 0 01-.704 0l-.003-.001z"
              />
            </svg>
          </div>
        </div>
      </button>
    </div>

    <Transition name="fade-slow" mode="out-in">
      <component
        :is="isCartoon ? WeddingCartoon : WeddingFormal"
        :side="props.side"
      />
    </Transition>
  </div>
</template>

<style scoped>
.fade-slow-enter-active,
.fade-slow-leave-active {
  transition: opacity 0.6s ease;
}
.fade-slow-enter-from,
.fade-slow-leave-to {
  opacity: 0;
}
.cubic-bezier {
  transition-timing-function: cubic-bezier(0.68, -0.55, 0.265, 1.55);
}
</style>
