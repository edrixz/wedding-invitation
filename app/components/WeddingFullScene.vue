<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue'
import gsap from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'

gsap.registerPlugin(ScrollTrigger)

// Refs
const containerRef = ref<HTMLElement | null>(null)
const sceneRef = ref<HTMLElement | null>(null)
const coverPageRef = ref<HTMLElement | null>(null)
const innerPageRef = ref<HTMLElement | null>(null)
const shadowRef = ref<HTMLElement | null>(null)
const lightSheenRef = ref<HTMLElement | null>(null)
const guideTextRef = ref<HTMLElement | null>(null)

// Các phần tử nội dung
const contentHeader = ref<HTMLElement | null>(null)
const contentNames = ref<HTMLElement | null>(null)
const contentDate = ref<HTMLElement | null>(null)
const contentPlace = ref<HTMLElement | null>(null)
const contentMap = ref<HTMLElement | null>(null)
const contentFooter = ref<HTMLElement | null>(null)

let ctx: gsap.Context

// Texture giấy kem
const paperTexture = "https://www.transparenttextures.com/patterns/cream-paper.png"

onMounted(() => {
  ctx = gsap.context(() => {
    
    // Timeline chính
    const tl = gsap.timeline({
      scrollTrigger: {
        trigger: containerRef.value!,
        start: 'top top',
        end: '+=2500', // Rút ngắn quãng đường scroll lại để vuốt nhanh tới đích hơn
        scrub: 1,
        pin: sceneRef.value!,
        invalidateOnRefresh: true,
        
        // --- TÍNH NĂNG SNAP SCROLL (CUỘN DÍNH) ---
        snap: {
            snapTo: [0, 1], // Chỉ cho phép dừng ở: 0 (Đóng kín) hoặc 1 (Mở hết)
            duration: { min: 0.3, max: 0.8 }, // Tốc độ tự động trượt (nhanh/chậm tùy lực vuốt)
            delay: 0.1, // Chờ 0.1s sau khi thả tay rồi mới snap
            ease: "power2.inOut", // Hiệu ứng trượt mượt mà
            inertia: false // Tắt quán tính để snap dứt khoát hơn
        }
      }
    })

    // 0. Ẩn hướng dẫn
    tl.to(guideTextRef.value, { opacity: 0, duration: 0.2 })

    // --- GIAI ĐOẠN 1: MỞ BÌA ---
    tl.to(coverPageRef.value, {
        rotateY: -180,
        duration: 5,
        ease: 'power1.inOut'
    })
    
    // Hiệu ứng ánh sáng
    .fromTo(lightSheenRef.value, 
        { x: '-100%', opacity: 0 }, 
        { x: '200%', opacity: 0.6, duration: 3, ease: 'power2.inOut' }, 
        "<"
    )

    // Bóng đổ mờ dần
    .to(shadowRef.value, {
        opacity: 0,
        duration: 4,
        ease: 'power2.in'
    }, "<+=0.5")


    // --- GIAI ĐOẠN 2: NỘI DUNG XUẤT HIỆN LIÊN TIẾP (STAGGER) ---
    const contentGroup = [
        contentHeader.value,
        contentNames.value,
        contentDate.value,
        contentPlace.value,
        contentMap.value,
        contentFooter.value
    ];

    tl.from(contentGroup, {
        y: 30,          // Bay lên nhẹ nhàng
        opacity: 0,     // Hiện dần
        scale: 0.95,    // Phóng to nhẹ
        duration: 2,
        stagger: 0.15,  // Xuất hiện dồn dập, nhanh chóng
        ease: 'back.out(1.2)'
    }, "-=2") // Bắt đầu hiện ngay khi bìa vừa mở được một nửa

  }, containerRef.value!)
})

onUnmounted(() => ctx?.revert())
</script>

<template>
  <div ref="containerRef" class="w-full h-[300vh] bg-black">
    
    <div ref="sceneRef" class="w-full h-[100dvh] overflow-hidden flex items-center justify-center relative perspective-container bg-silk-table">
      
      <div class="relative w-[90vw] h-[85vh] md:w-[500px] md:h-[720px] rounded-xl shadow-[0_30px_60px_rgba(0,0,0,0.9)] preserve-3d will-change-transform my-auto">
        
        <div ref="innerPageRef" class="absolute inset-0 bg-[#FFFBF0] flex flex-col items-center rounded-r-xl overflow-hidden"
             :style="{ backgroundImage: `url('${paperTexture}')` }">
             
             <div class="absolute top-0 bottom-0 left-0 w-6 bg-gradient-to-r from-black/25 to-transparent z-10 pointer-events-none"></div>

             <div ref="shadowRef" class="absolute inset-0 bg-black/75 z-20 pointer-events-none"></div>

             <div class="w-full h-full px-5 py-8 md:px-12 md:py-20 flex flex-col items-center text-center font-serif text-[#3e2723] justify-between relative z-0">
                
                <div class="w-full flex justify-between opacity-50">
                    <div class="w-10 h-10 md:w-14 md:h-14 border-t-[2px] border-l-[2px] border-[#b8860b]"></div>
                    <div class="w-10 h-10 md:w-14 md:h-14 border-t-[2px] border-r-[2px] border-[#b8860b]"></div>
                </div>

                <div class="flex-1 flex flex-col justify-center w-full space-y-6 md:space-y-10">
                    
                    <div ref="contentHeader">
                        <p class="text-xs md:text-sm uppercase tracking-[3px] text-[#8B4513] mb-2 font-bold">Trân trọng kính mời</p>
                        <h2 class="text-base md:text-xl italic text-[#5d4037] font-dancing">Tới dự lễ Vu Quy của hai con chúng tôi</h2>
                    </div>

                    <div ref="contentNames" class="font-dancing text-5xl md:text-7xl text-[#b8860b] leading-tight drop-shadow-sm py-2">
                        Phương Huyền <br/> 
                        <span class="text-xl md:text-2xl text-[#5d4037] font-serif">&</span> <br/> 
                        Văn Hiếu
                    </div>

                    <div ref="contentDate" class="w-full border-y border-[#b8860b]/30 py-4 bg-[#b8860b]/5">
                        <p class="text-[10px] md:text-xs font-bold uppercase text-[#5d4037] mb-2">Vào hồi 11 giờ 30</p>
                        <div class="text-4xl md:text-6xl font-bold text-[#7f1d1d] mb-2 tracking-widest">22.02.26</div>
                        <p class="text-[10px] md:text-xs uppercase tracking-widest text-[#5d4037]">Chủ Nhật (06/01 Âm Lịch)</p>
                    </div>

                    <div ref="contentPlace">
                        <p class="font-bold uppercase text-[#3e2723] text-sm md:text-base mb-1">Tại Tư Gia Nhà Gái</p>
                        <p class="text-base md:text-lg text-[#5d4037]">TDP Tân Tiến - Xã Kiến Xương - Hưng Yên</p>
                    </div>

                    <div ref="contentMap">
                        <button class="px-8 py-3 border border-[#b8860b] text-[#b8860b] text-xs md:text-sm uppercase font-bold hover:bg-[#b8860b] hover:text-white transition-colors tracking-widest shadow-sm rounded-sm">
                            Xem bản đồ
                        </button>
                    </div>
                </div>

                <div ref="contentFooter" class="w-full grid grid-cols-2 gap-4 text-[10px] md:text-xs uppercase tracking-wide border-t border-[#b8860b]/30 pt-4 opacity-80 mt-6">
                    <div class="text-left">
                        <span class="font-bold block text-[#7f1d1d] mb-1">Nhà Gái</span>
                        <p class="md:text-sm">Mẹ: Phạm Thị Báu</p>
                    </div>
                    <div class="text-right">
                        <span class="font-bold block text-[#7f1d1d] mb-1">Nhà Trai</span>
                        <p class="md:text-sm">Bố: Lều Văn Toàn</p>
                        <p class="md:text-sm">Mẹ: Vũ Thị Thuỷ</p>
                    </div>
                </div>

                <div class="w-full flex justify-between absolute bottom-10 left-0 px-5 md:px-12 opacity-50 pointer-events-none">
                    <div class="w-10 h-10 md:w-14 md:h-14 border-b-[2px] border-l-[2px] border-[#b8860b]"></div>
                    <div class="w-10 h-10 md:w-14 md:h-14 border-b-[2px] border-r-[2px] border-[#b8860b]"></div>
                </div>

             </div>
        </div>

        <div ref="coverPageRef" class="absolute inset-0 z-30 preserve-3d origin-left will-change-transform shadow-[-15px_0_40px_rgba(0,0,0,0.6)]">
            
            <div class="absolute inset-0 z-20 overflow-hidden rounded-r-xl backface-hidden bg-[#2b0a0a]">
                <img src="/images/DSC01773.jpg" class="w-full h-full object-cover opacity-90" alt="Cover" />
                <div class="absolute inset-0 bg-gradient-to-t from-black/90 via-transparent to-black/40"></div>
                
                <div ref="lightSheenRef" class="absolute inset-0 w-1/2 h-full bg-gradient-to-r from-transparent via-white/20 to-transparent -skew-x-12 z-30 pointer-events-none"></div>

                <div class="absolute inset-0 flex flex-col items-center justify-end pb-16 md:pb-24 z-20 text-center px-6">
                    <p class="text-[#FFD700] text-xs md:text-sm uppercase tracking-[6px] mb-4 font-bold text-shadow-gold">Thành Hôn</p>
                    <h1 class="font-dancing text-6xl md:text-8xl text-white drop-shadow-2xl gold-gradient-text mb-4">
                        Huyền & Hiếu
                    </h1>
                    <div class="w-16 md:w-24 h-[2px] bg-[#FFD700] mb-4 opacity-80"></div>
                    <p class="text-white/90 text-xs md:text-base uppercase tracking-[4px] font-bold">22 . 02 . 2026</p>
                </div>
                
                <div class="absolute inset-4 md:inset-6 border-2 border-[#FFD700]/30 pointer-events-none z-20 rounded-lg"></div>
            </div>

            <div class="absolute inset-0 z-10 bg-[#3e0000] rounded-l-xl backface-hidden"
                 style="transform: rotateY(180deg);">
                 <div class="absolute inset-0 opacity-20" 
                      style="background-image: radial-gradient(#FFD700 1px, transparent 1px); background-size: 20px 20px;"></div>
                 <div class="absolute top-0 bottom-0 right-0 w-16 bg-gradient-to-l from-black/60 to-transparent"></div>
            </div>

        </div>

      </div>

      <div ref="guideTextRef" class="absolute bottom-12 animate-bounce z-50 pointer-events-none text-center w-full">
         <p class="text-[#ffd700] text-[10px] md:text-xs uppercase tracking-[4px] mb-2 font-bold shadow-black drop-shadow-md">Vuốt để mở thiệp</p>
         <div class="w-[1px] h-8 bg-gradient-to-b from-[#ffd700] to-transparent mx-auto"></div>
      </div>

    </div>
  </div>
</template>

<style scoped>
.perspective-container { perspective: 1500px; }
.preserve-3d { transform-style: preserve-3d; }
.backface-hidden { backface-visibility: hidden; }
.font-dancing { font-family: 'Dancing Script', cursive; }

.bg-silk-table {
    background-color: #4a0000;
    background-image: 
        repeating-linear-gradient(45deg, rgba(0,0,0,0.1) 0px, rgba(0,0,0,0.1) 2px, transparent 2px, transparent 4px),
        repeating-linear-gradient(-45deg, rgba(255,255,255,0.05) 0px, rgba(255,255,255,0.05) 2px, transparent 2px, transparent 4px);
    background: radial-gradient(circle at center, #7c0a02 0%, #3e0000 60%, #1a0505 100%);
    background-blend-mode: multiply;
}

.gold-gradient-text {
    background: linear-gradient(to bottom, #fffeba, #FFD700, #daa520);
    -webkit-background-clip: text;
    background-clip: text;
    color: transparent;
    filter: drop-shadow(0 4px 6px rgba(0,0,0,0.8));
}
.text-shadow-gold {
    text-shadow: 0 2px 4px rgba(0,0,0,0.9);
}
</style>