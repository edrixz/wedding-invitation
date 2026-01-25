<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import gsap from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'

gsap.registerPlugin(ScrollTrigger)

// Refs
const containerRef = ref<HTMLElement | null>(null)
const sceneRef = ref<HTMLElement | null>(null)
const coverRef = ref<HTMLElement | null>(null) // Lớp bìa
const innerRef = ref<HTMLElement | null>(null) // Lớp nội dung
const contentElementsRef = ref<HTMLElement | null>(null) // Nhóm text bên trong
const guideTextRef = ref<HTMLElement | null>(null) // Hướng dẫn vuốt

let ctx: gsap.Context

// Texture
const paperTexture = "https://www.transparenttextures.com/patterns/cream-paper.png"

onMounted(() => {
  ctx = gsap.context(() => {
    
    const tl = gsap.timeline({
      scrollTrigger: {
        trigger: containerRef.value!,
        start: 'top top',
        end: '+=2000', // Độ dài quãng đường cuộn (càng lớn cuộn càng chậm)
        scrub: 1,      // Mượt mà, dính theo ngón tay
        pin: sceneRef.value!, // Ghim màn hình lại
        invalidateOnRefresh: true
      }
    })

    // 0. Ẩn hướng dẫn khi bắt đầu cuộn
    tl.to(guideTextRef.value, { opacity: 0, duration: 0.2 })

    // --- ANIMATION CHÍNH ---
    
    // 1. Bìa trượt lên trên (Slide Up)
    tl.to(coverRef.value, {
        y: '-100%', 
        ease: 'none', // Chuyển động tuyến tính theo scroll
        duration: 10
    })

    // 2. Nội dung bên dưới hiện dần ra (Parallax & Fade In)
    // Nội dung sẽ hơi phóng to và rõ dần khi bìa trượt lên
    .fromTo(innerRef.value, 
        { scale: 0.9, opacity: 0.5, filter: 'blur(5px)' },
        { scale: 1, opacity: 1, filter: 'blur(0px)', duration: 8, ease: 'power1.out' },
        0 // Bắt đầu ngay từ giây 0 (cùng lúc bìa trượt)
    )

    // 3. Các dòng chữ bên trong bay lên (Stagger Text)
    // Khi bìa trượt được 1 nửa, chữ bên trong bắt đầu bay vào vị trí
    .from(contentElementsRef.value!.children, {
        y: 50,
        opacity: 0,
        duration: 5,
        stagger: 0.5, // Các dòng xuất hiện đuổi nhau
        ease: 'power2.out'
    }, 2) 

  }, containerRef.value!)
})

onUnmounted(() => ctx?.revert())
</script>

<template>
  <div ref="containerRef" class="w-full h-[300vh] bg-black">
    
    <div ref="sceneRef" class="w-full h-[100dvh] overflow-hidden relative bg-[#2b0a0a]">
      
      <div ref="innerRef" class="absolute inset-0 z-0 flex flex-col items-center justify-center bg-[#FFFBF0]"
           :style="{ backgroundImage: `url('${paperTexture}')` }">
           
           <div class="absolute top-6 left-6 w-16 h-16 border-t-2 border-l-2 border-[#b8860b]/50"></div>
           <div class="absolute top-6 right-6 w-16 h-16 border-t-2 border-r-2 border-[#b8860b]/50"></div>
           <div class="absolute bottom-6 left-6 w-16 h-16 border-b-2 border-l-2 border-[#b8860b]/50"></div>
           <div class="absolute bottom-6 right-6 w-16 h-16 border-b-2 border-r-2 border-[#b8860b]/50"></div>

           <div ref="contentElementsRef" class="relative z-10 flex flex-col items-center text-center space-y-6 px-6 md:px-0 w-full max-w-md">
                
                <div>
                    <p class="text-xs uppercase tracking-[4px] text-[#8B4513] mb-2 font-bold">Trân trọng kính mời</p>
                    <h2 class="text-lg italic text-[#5d4037] font-dancing">Tới dự lễ Vu Quy của hai con chúng tôi</h2>
                </div>

                <div class="font-dancing text-6xl md:text-7xl text-[#b8860b] leading-tight py-2 drop-shadow-sm gold-gradient-text">
                    Phương Huyền <br/> 
                    <span class="text-2xl text-[#5d4037] font-serif">&</span> <br/> 
                    Văn Hiếu
                </div>

                <div class="w-full border-y border-[#b8860b]/30 py-4 bg-[#b8860b]/5">
                    <p class="text-[10px] font-bold uppercase text-[#5d4037] mb-2">Vào hồi 11 giờ 30</p>
                    <div class="text-5xl font-bold text-[#7f1d1d] mb-2 tracking-widest">22.02.26</div>
                    <p class="text-[10px] uppercase tracking-widest text-[#5d4037]">Chủ Nhật (06/01 Âm Lịch)</p>
                </div>

                <div>
                    <p class="font-bold uppercase text-[#3e2723] text-sm mb-1">Tại Tư Gia Nhà Gái</p>
                    <p class="text-base text-[#5d4037]">TDP Tân Tiến - Xã Kiến Xương - Hưng Yên</p>
                </div>

                <button class="px-8 py-3 border border-[#b8860b] text-[#b8860b] text-xs uppercase font-bold hover:bg-[#b8860b] hover:text-white transition-colors tracking-widest shadow-sm">
                    Xem bản đồ
                </button>

                <div class="w-full grid grid-cols-2 gap-4 text-[10px] uppercase tracking-wide border-t border-[#b8860b]/30 pt-4 opacity-80">
                    <div class="text-left">
                        <span class="font-bold block text-[#7f1d1d] mb-1">Nhà Gái</span>
                        <p>Mẹ: Phạm Thị Báu</p>
                    </div>
                    <div class="text-right">
                        <span class="font-bold block text-[#7f1d1d] mb-1">Nhà Trai</span>
                        <p>Bố: Lều Văn Toàn</p>
                        <p>Mẹ: Vũ Thị Thuỷ</p>
                    </div>
                </div>
           </div>
      </div>

      <div ref="coverRef" class="absolute inset-0 z-20 w-full h-full shadow-2xl">
          <img src="/images/DSC01773.jpg" class="w-full h-full object-cover" alt="Cover" />
          
          <div class="absolute inset-0 bg-gradient-to-t from-black/90 via-black/20 to-transparent"></div>

          <div class="absolute inset-0 flex flex-col items-center justify-end pb-24 md:pb-32 px-6 text-center">
              <p class="text-[#FFD700] text-sm uppercase tracking-[6px] mb-4 font-bold text-shadow-gold animate-pulse">Thành Hôn</p>
              
              <h1 class="font-dancing text-6xl md:text-8xl text-white drop-shadow-2xl gold-gradient-text mb-6">
                  Huyền & Hiếu
              </h1>
              
              <div class="w-24 h-[2px] bg-[#FFD700] mb-6 opacity-80"></div>
              
              <p class="text-white/90 text-sm uppercase tracking-[4px] font-bold mb-2">22 . 02 . 2026</p>
              <p class="text-white/60 text-xs italic font-serif">Vuốt lên để xem thiệp mời</p>
          </div>

          <div class="absolute inset-6 border border-[#FFD700]/40 rounded-sm pointer-events-none"></div>

          <div class="absolute -bottom-10 left-0 right-0 h-10 bg-gradient-to-t from-transparent to-black/50"></div>
      </div>

      <div ref="guideTextRef" class="absolute bottom-10 left-1/2 -translate-x-1/2 z-50 animate-bounce flex flex-col items-center pointer-events-none">
          <div class="w-[1px] h-12 bg-gradient-to-t from-[#FFD700] to-transparent"></div>
          <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="#FFD700" class="w-6 h-6 mt-2">
            <path stroke-linecap="round" stroke-linejoin="round" d="M19.5 8.25l-7.5 7.5-7.5-7.5" />
          </svg>
      </div>

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