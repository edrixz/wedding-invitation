<script setup lang="ts">
import { ref, computed } from "vue";
import WeddingCartoon from "./WeddingCartoon.vue";
import WeddingFormal from "./WeddingFormal.vue";

const props = defineProps<{
  side: "bride" | "groom";
}>();

const currentStyle = ref("formal");
const isSwitching = ref(false);
const isCartoon = computed(() => currentStyle.value === "cartoon");
const activeComponentRef = ref<any>(null);

const toggleStyle = async () => {
  if (isSwitching.value) return;
  isSwitching.value = true;

  // 1. Nếu đang mở -> Ra lệnh đóng và chờ
  if (activeComponentRef.value && activeComponentRef.value.isOpened) {
    activeComponentRef.value.closeInvitation();
    await new Promise((resolve) => setTimeout(resolve, 1800)); // Chờ animation đóng xong (1.8s)
  }

  // 2. Chuyển đổi Style
  const nextStyle = isCartoon.value ? "formal" : "cartoon";
  try {
    const img = new Image();
    img.src =
      nextStyle === "formal" ? "/images/cover.jpg" : "/images/cover-anime.jpg";
    await img.decode();
  } catch (e) {
    console.warn("Preload failed", e);
  }

  currentStyle.value = nextStyle;

  // 3. Kết thúc chuyển đổi
  setTimeout(() => {
    isSwitching.value = false;
  }, 600);
};
</script>

<template>
  <div class="relative w-full h-dvh overflow-hidden bg-black">
    <div class="fixed top-5 left-5 z-9999">
      <button
        @click="toggleStyle"
        :disabled="isSwitching"
        class="group relative flex items-center cursor-pointer outline-none select-none w-14 h-8 rounded-full transition-all duration-500 ease-in-out shadow-lg hover:scale-105"
        :class="[
          !isCartoon
            ? 'bg-red-950/90 border border-yellow-700/60'
            : 'bg-[#FFD54F] border-2 border-[#3E2723]',
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
              ? 'left-1 w-6 h-6 bg-linear-to-br from-yellow-100 to-yellow-600 border border-white/20'
              : 'left-[calc(100%-1.6rem)] w-5 h-5 bg-white border-2 border-[#3E2723]',
          ]"
        >
          <div
            class="transform transition-transform duration-500"
            :class="isCartoon ? 'rotate-12 scale-90' : 'scale-75'"
          >
            <svg
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

    <Transition name="switch-style">
      <component
        :is="isCartoon ? WeddingCartoon : WeddingFormal"
        :side="props.side"
        :key="currentStyle"
        ref="activeComponentRef"
        class="absolute inset-0 w-full h-full"
      />
    </Transition>
  </div>
</template>

<style scoped>
.cubic-bezier {
  transition-timing-function: cubic-bezier(0.68, -0.55, 0.265, 1.55);
}
.switch-style-enter-active,
.switch-style-leave-active {
  transition: all 0.6s cubic-bezier(0.4, 0, 0.2, 1);
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
}
.switch-style-enter-from {
  opacity: 0;
  transform: scale(1.05) filter(blur(4px));
  z-index: 2;
}
.switch-style-leave-to {
  opacity: 0;
  transform: scale(0.95) filter(blur(2px));
  z-index: 1;
}
.switch-style-enter-to,
.switch-style-leave-from {
  opacity: 1;
  transform: scale(1) filter(blur(0));
}
</style>
