<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import gsap from 'gsap'

// Refs
const containerRef = ref<HTMLElement | null>(null)
const coverRef = ref<HTMLElement | null>(null) 
const innerRef = ref<HTMLElement | null>(null) 
const contentElementsRef = ref<HTMLElement | null>(null)
const openButtonRef = ref<HTMLElement | null>(null)
const closeButtonRef = ref<HTMLElement | null>(null)

// State
const isOpened = ref(false)

let ctx: gsap.Context
let tl: gsap.core.Timeline

const paperTexture = "https://www.transparenttextures.com/patterns/cream-paper.png"

const openInvitation = () => {
    isOpened.value = true;
    if (tl) tl.play();
}

const closeInvitation = () => {
    if (tl) tl.reverse();
}

onMounted(() => {
  ctx = gsap.context(() => {
    
    tl = gsap.timeline({ 
        paused: true,
        onReverseComplete: () => {
            isOpened.value = false;
        }
    });

    // --- ANIMATION ---
    
    // 1. Bìa trượt lên trên (Nút mở nằm trong bìa nên sẽ trượt theo)
    tl.to(coverRef.value, {
        y: '-100%', 
        duration: 1.5,
        ease: 'power2.inOut'
    })

    // 2. Nội dung bên dưới hiện dần ra
    .fromTo(innerRef.value, 
        { scale: 0.92, filter: 'brightness(0.5)' },
        { scale: 1, filter: 'brightness(1)', duration: 1.5, ease: 'power2.inOut' },
        "<" 
    )

    // 3. Các dòng chữ bên trong bay lên
    .from(contentElementsRef.value!.children, {
        y: 30,
        opacity: 0,
        duration: 1,
        stagger: 0.1, 
        ease: 'back.out(1.2)'
    }, "-=0.5") 

    // 4. Hiện nút Đóng (X)
    .to(closeButtonRef.value, {
        opacity: 1, 
        scale: 1,
        pointerEvents: 'auto', 
        duration: 0.5 
    })

  }, containerRef.value!)
})

onUnmounted(() => ctx?.revert())
</script>

<template>
  <div ref="containerRef" class="w-full h-[100dvh] bg-black overflow-hidden relative">
    
    <div ref="closeButtonRef" 
         @click="closeInvitation" 
         class="absolute top-8 right-8 z-50 cursor-pointer opacity-0 scale-50 pointer-events-none p-2 rounded-full bg-black/10 hover:bg-black/20 backdrop-blur-sm active:scale-90 transition-transform">
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="2.5" stroke="#8B4513" class="w-6 h-6 md:w-8 md:h-8">
            <path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
        </svg>
    </div>

    <div ref="innerRef" class="absolute inset-0 z-0 flex flex-col items-center justify-center bg-[#FFFBF0]"
         :style="{ backgroundImage: `url('${paperTexture}')` }">
         
         <div class="absolute top-6 left-6 w-12 h-12 md:w-16 md:h-16 border-t-2 border-l-2 border-[#b8860b]/50"></div>
         <div class="absolute top-6 right-6 w-12 h-12 md:w-16 md:h-16 border-t-2 border-r-2 border-[#b8860b]/50"></div>
         <div class="absolute bottom-6 left-6 w-12 h-12 md:w-16 md:h-16 border-b-2 border-l-2 border-[#b8860b]/50"></div>
         <div class="absolute bottom-6 right-6 w-12 h-12 md:w-16 md:h-16 border-b-2 border-r-2 border-[#b8860b]/50"></div>

         <div ref="contentElementsRef" class="relative z-10 flex flex-col items-center text-center space-y-5 px-6 md:px-0 w-full max-w-md">
              <div>
                  <p class="text-xs uppercase tracking-[4px] text-[#8B4513] mb-2 font-bold">Trân trọng kính mời</p>
                  <h2 class="text-base italic text-[#5d4037] font-dancing">Tới dự lễ Vu Quy của hai con chúng tôi</h2>
              </div>
              <div class="font-dancing text-5xl md:text-7xl text-[#b8860b] leading-tight py-2 drop-shadow-sm gold-gradient-text">
                  Phương Huyền <br/> <span class="text-xl text-[#5d4037] font-serif">&</span> <br/> Văn Hiếu
              </div>
              <div class="w-full border-y border-[#b8860b]/30 py-4 bg-[#b8860b]/5">
                  <p class="text-[10px] font-bold uppercase text-[#5d4037] mb-2">Vào hồi 11 giờ 30</p>
                  <div class="text-4xl font-bold text-[#7f1d1d] mb-2 tracking-widest">22.02.26</div>
                  <p class="text-[10px] uppercase tracking-widest text-[#5d4037]">Chủ Nhật (06/01 Âm Lịch)</p>
              </div>
              <div>
                  <p class="font-bold uppercase text-[#3e2723] text-sm mb-1">Tại Tư Gia Nhà Gái</p>
                  <p class="text-sm text-[#5d4037]">TDP Tân Tiến - Xã Kiến Xương - Hưng Yên</p>
              </div>
              <button class="px-8 py-3 border border-[#b8860b] text-[#b8860b] text-xs uppercase font-bold hover:bg-[#b8860b] hover:text-white transition-colors tracking-widest shadow-sm rounded-sm">
                  Xem bản đồ
              </button>
              <div class="w-full grid grid-cols-2 gap-4 text-[10px] uppercase tracking-wide border-t border-[#b8860b]/30 pt-4 opacity-80">
                  <div class="text-left"><span class="font-bold block text-[#7f1d1d] mb-1">Nhà Gái</span><p>Mẹ: Phạm Thị Báu</p></div>
                  <div class="text-right"><span class="font-bold block text-[#7f1d1d] mb-1">Nhà Trai</span><p>Bố: Lều Văn Toàn</p><p>Mẹ: Vũ Thị Thuỷ</p></div>
              </div>
         </div>
    </div>

    <div ref="coverRef" class="absolute inset-0 z-20 w-full h-full shadow-[0_10px_50px_rgba(0,0,0,0.8)] will-change-transform">
        <img src="/images/DSC01773.jpg" class="w-full h-full object-cover" alt="Cover" />
        <div class="absolute inset-0 bg-gradient-to-t from-black/90 via-black/30 to-black/10"></div>

        <div class="absolute inset-0 flex flex-col items-center justify-end pb-32 px-6 text-center z-10">
            <p class="text-[#FFD700] text-xs uppercase tracking-[6px] mb-4 font-bold text-shadow-gold">Thành Hôn</p>
            <h1 class="font-dancing text-6xl md:text-8xl text-white drop-shadow-2xl gold-gradient-text mb-6">Huyền & Hiếu</h1>
            <div class="w-24 h-[2px] bg-[#FFD700] mb-6 opacity-80"></div>
            <p class="text-white/90 text-sm uppercase tracking-[4px] font-bold mb-2">22 . 02 . 2026</p>
        </div>

        <div ref="openButtonRef"
             @click="openInvitation"
             class="absolute bottom-12 left-1/2 -translate-x-1/2 z-50 flex flex-col items-center cursor-pointer animate-bounce">
            <p class="text-[#ffd700] text-[10px] uppercase tracking-[4px] mb-2 font-bold shadow-black drop-shadow-md">Bấm để mở thiệp</p>
            <div class="w-12 h-12 rounded-full border border-[#FFD700]/50 bg-black/40 backdrop-blur-sm flex items-center justify-center hover:bg-[#FFD700] hover:text-black transition-colors duration-300">
                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6 text-[#FFD700] hover:text-black">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M4.5 15.75l7.5-7.5 7.5 7.5" />
                </svg>
            </div>
        </div>

        <div class="absolute -bottom-20 left-0 right-0 h-20 bg-black/80 blur-xl"></div>
    </div>

  </div>
</template>

<style scoped>
.font-dancing { font-family: 'Dancing Script', cursive; }
.gold-gradient-text {
    background: linear-gradient(to bottom, #fffeba, #FFD700, #daa520);
    -webkit-background-clip: text;
    background-clip: text;
    color: transparent;
    filter: drop-shadow(0 4px 6px rgba(0,0,0,0.8));
}
.text-shadow-gold { text-shadow: 0 2px 4px rgba(0,0,0,0.9); }
</style>