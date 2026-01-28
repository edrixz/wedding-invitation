<script setup lang="ts">
import type { NuxtError } from "#app";

// Nhận object error từ Nuxt
defineProps({
  error: Object as () => NuxtError,
});

// Hàm xóa lỗi và quay về trang chủ
const handleError = () => clearError({ redirect: "/" });
</script>

<template>
  <div
    class="antialiased w-full h-dvh bg-zinc-900 flex justify-center items-center overflow-hidden font-serif text-[#3E2723]"
  >
    <div
      class="relative w-full max-w-97.5 h-full bg-[#fafaef] shadow-2xl overflow-hidden flex flex-col items-center justify-center p-6 text-center"
    >
      <div
        class="pointer-events-none absolute inset-0 z-0 opacity-40 mix-blend-multiply"
        style="
          background-image: url(&quot;https://www.transparenttextures.com/patterns/cream-paper.png&quot;);
        "
      ></div>

      <div class="relative space-y-6">
        <img
          src="/images/404.png"
          class="object-cover max-h-30 mx-auto mb-4 mix-blend-multiply brightness-110 contrast-110"
          alt="404 Illustration"
          decoding="sync"
        />

        <h2 class="font-great-vibes text-4xl md:text-5xl text-red-900">
          Lạc đường rồi...
        </h2>

        <p class="font-serif italic text-sm md:text-base px-8 text-gray-600">
          Có vẻ bạn đã đi lạc khỏi nơi tổ chức hôn lễ.<br />Đừng lo, hãy quay
          lại sảnh chính nhé.
        </p>

        <button
          @click="handleError"
          class="mt-8 px-8 py-3 bg-[#880E4F] text-white font-cinzel font-bold text-xs tracking-[2px] rounded-full shadow-lg hover:bg-[#B71C1C] transition-all transform hover:scale-105 active:scale-95 cursor-pointer uppercase border border-red-900/20"
        >
          Quay về trang chủ
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Các font này đã được load global trong app.vue, ở đây chỉ khai báo class */
.font-cinzel {
  font-family: "Cinzel", serif;
}
.font-great-vibes {
  font-family: "Great Vibes", cursive;
}
.font-serif {
  font-family: "Cormorant Garamond", serif;
}
</style>
