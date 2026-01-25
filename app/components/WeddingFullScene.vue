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

    // 1. Ẩn nút Mở và logo trên bìa
    tl.to([openButtonRef.value, '.cover-content'], { opacity: 0, scale: 0.9, duration: 0.3, pointerEvents: 'none' })

    // 2. Bìa trượt lên trên
    .to(coverRef.value, {
        y: '-100%', 
        duration: 1.5,
        ease: 'power2.inOut'
    })

    // 3. Nội dung bên dưới hiện dần ra
    .fromTo(innerRef.value, 
        { scale: 0.92, filter: 'brightness(0.5)' },
        { scale: 1, filter: 'brightness(1)', duration: 1.5, ease: 'power2.inOut' },
        "<" 
    )

    // 4. Các dòng chữ bên trong bay lên
    .from(contentElementsRef.value!.children, {
        y: 30,
        opacity: 0,
        duration: 1,
        stagger: 0.1, 
        ease: 'back.out(1.2)'
    }, "-=0.5") 

    // 5. Hiện nút Đóng (X)
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
  <div ref="containerRef" class="w-full h-[100dvh] bg-black overflow-hidden relative font-pangolin">
    
    <div ref="closeButtonRef" 
         @click="closeInvitation" 
         class="absolute top-6 right-6 z-50 cursor-pointer opacity-0 scale-50 pointer-events-none p-2 rounded-full bg-[#3E2723]/80 border-2 border-[#FFF8DC] hover:bg-[#5D4037] active:scale-90 transition-transform shadow-cartoon">
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="3" stroke="#FFF8DC" class="w-6 h-6 md:w-8 md:h-8">
            <path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
        </svg>
    </div>

    <div ref="innerRef" class="absolute inset-0 z-0 flex flex-col items-center justify-center bg-[#FFFBF0]"
         :style="{ backgroundImage: `url('${paperTexture}')` }">
         
         <div class="absolute top-6 left-6 w-12 h-12 border-t-4 border-l-4 border-[#8B4513]/40 rounded-tl-lg"></div>
         <div class="absolute top-6 right-6 w-12 h-12 border-t-4 border-r-4 border-[#8B4513]/40 rounded-tr-lg"></div>
         <div class="absolute bottom-6 left-6 w-12 h-12 border-b-4 border-l-4 border-[#8B4513]/40 rounded-bl-lg"></div>
         <div class="absolute bottom-6 right-6 w-12 h-12 border-b-4 border-r-4 border-[#8B4513]/40 rounded-br-lg"></div>

         <div ref="contentElementsRef" class="relative z-10 flex flex-col items-center text-center space-y-4 md:space-y-6 px-6 md:px-0 w-full max-w-md text-[#5d4037]">
              <div>
                  <p class="text-sm uppercase tracking-[2px] mb-1 font-bold">Trân trọng kính mời</p>
                  <h2 class="text-base md:text-lg italic">Tới dự lễ Vu Quy của hai con chúng tôi</h2>
              </div>
              
              <div class="text-5xl md:text-7xl text-[#b8860b] leading-10 py-2 font-patrick font-bold">
                  Phương Huyền <br/> <span class="text-3xl md:text-4xl text-[#5d4037]">&</span> <br/> Văn Hiếu
              </div>

              <div class="w-full border-y-2 border-[#b8860b]/30 py-3 md:py-4 bg-[#b8860b]/10 rounded-lg">
                  <p class="text-xs font-bold uppercase mb-1 md:mb-2">Vào hồi 11 giờ 30</p>
                  <div class="text-4xl md:text-5xl font-bold text-[#7f1d1d] mb-1 md:mb-2 tracking-widest font-patrick">22.02.2026</div>
                  <p class="text-xs uppercase tracking-widest">Tức ngày 6 tháng 1 năm Bính Ngọ</p>
              </div>
              
              <div>
                  <p class="font-bold uppercase text-[#3e2723] text-sm md:text-base mb-1">Tại Gia đình Nhà Gái</p>
                  <p class="text-sm md:text-base">TDP Tân Tiến - Xã Kiến Xương - Hưng Yên</p>
              </div>
              
              <div class="w-full grid grid-cols-2 gap-4 text-xs uppercase tracking-wide border-t-2 border-[#b8860b]/30 pt-3 md:pt-4 opacity-80">
                  <div class="text-left"><span class="font-bold block text-[#7f1d1d] mb-1">Nhà Gái</span><p>Mẹ: Phạm Thị Báu</p></div>
                  <div class="text-right"><span class="font-bold block text-[#7f1d1d] mb-1">Nhà Trai</span><p>Bố: Lều Văn Toàn</p><p>Mẹ: Vũ Thị Thuỷ</p></div>
              </div>
         </div>
    </div>

    <div ref="coverRef" class="absolute inset-0 z-20 w-full h-full shadow-[0_10px_30px_rgba(0,0,0,0.5)] will-change-transform overflow-hidden flex flex-col justify-between bg-[#2b0a0a]">
        
        <div class="absolute inset-0 z-0">
            <img src="/images/cover-anime.jpg" class="w-full h-full object-cover scale-105" alt="Cover" />
            <div class="absolute inset-0 bg-gradient-to-b from-[#3E2723]/20 via-transparent to-[#3E2723]/60 mix-blend-multiply"></div>
        </div>

        <div class="cover-content relative z-30 w-full pt-12 md:pt-16 px-4 flex flex-col items-center">
            <img 
                src="/images/title.png" 
                alt="Phương Huyền & Văn Hiếu" 
                class="w-[90%] md:w-[70%] object-contain drop-shadow-xl animate-float-slow"
            />
        </div>

        <div ref="openButtonRef"
             @click="openInvitation"
             class="relative z-30 mb-[8vh] flex flex-col items-center cursor-pointer animate-bounce-slow w-full hover:scale-105 transition-transform">
            
            <div class="w-[6vh] h-[6vh] rounded-full bg-[#FFD54F] border-[0.5vh] border-[#3E2723] flex items-center justify-center shadow-cartoon-heavy relative z-10">
                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="4" stroke="#3E2723" class="w-[4vh] h-[4vh] animate-pulse">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M4.5 15.75l7.5-7.5 7.5 7.5" />
                </svg>
            </div>
        </div>

    </div>

  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Pangolin&family=Patrick+Hand&display=swap');

.font-pangolin { font-family: 'Pangolin', cursive; }
.font-patrick { font-family: 'Patrick Hand', cursive; }

/* Bóng đổ khối cho các element */
.shadow-cartoon {
    box-shadow: 4px 4px 0px #3E2723;
}
.shadow-cartoon-heavy {
    box-shadow: 0.6vh 0.6vh 0px #261410;
}

/* Animation nảy chậm */
.animate-bounce-slow {
    animation: bounce 2s infinite;
}

/* Animation nhấp nhô nhẹ cho logo */
@keyframes float {
    0%, 100% { transform: translateY(0); }
    50% { transform: translateY(-10px); }
}
.animate-float-slow {
    animation: float 4s ease-in-out infinite;
}
</style>