<script setup lang="ts">
// Trang điều hướng đơn giản
</script>

<template>
  <div
    class="h-dvh w-full flex flex-col items-center justify-between bg-[#FDFBF7] text-[#3E2723] font-serif p-6 py-20"
  >
    <div class="mb-8 text-center">
      <p
        class="text-xs uppercase tracking-[3px] text-gray-500 mb-2 font-wedding-1"
      >
        Trân trọng kính mời
      </p>
    </div>

    <div class="mb-8 text-center">
      <h1 class="text-5xl md:text-5xl text-[#B8860B] mb-2 font-great-vibes">
        Phương Huyền
      </h1>
      <h1 class="text-3xl md:text-5xl text-[#B8860B] mb-2 font-great-vibes">
        &
      </h1>
      <h1 class="text-5xl md:text-5xl text-[#B8860B] mb-2 font-great-vibes">
        Văn Hiếu
      </h1>
      <p class="text-sm italic text-gray-400 font-wedding-1">22 . 02 . 2026</p>
    </div>

    <div class="flex flex-row gap-4 w-full max-w-xs px-10">
      <NuxtLink
        to="/bride"
        class="block w-full text-center py-3 border border-[#B8860B] text-[#B8860B] rounded hover:bg-[#B8860B] hover:text-white transition-colors uppercase text-xs tracking-widest font-bold"
      >
        Nhà Gái
      </NuxtLink>

      <NuxtLink
        to="/groom"
        class="block w-full text-center py-3 border border-[#B8860B] text-[#B8860B] rounded hover:bg-[#B8860B] hover:text-white transition-colors uppercase text-xs tracking-widest font-bold"
      >
        Nhà Trai
      </NuxtLink>
    </div>
  </div>
</template>

<style scoped></style>
