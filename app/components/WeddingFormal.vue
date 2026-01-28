<script setup lang="ts">
import { onMounted, onUnmounted, ref, computed } from "vue";
import gsap from "gsap";

const props = withDefaults(
  defineProps<{
    side?: "bride" | "groom";
  }>(),
  {
    side: "bride",
  },
);

const WEDDING_DATA = {
  bride: {
    ceremonyTitle: "Lễ Vu Quy",
    mainName1: "Phương Huyền",
    mainName2: "Văn Hiếu",
    date: "22.02.2026",
    lunarDate: "(Tức ngày 06 tháng 01 năm Bính Ngọ)",
    time1: { label: "Bữa Cơm Thân Mật", value: "09:00" },
    time2: { label: "Lễ Vu Quy", value: "11:30" },
    locationTitle: "Tại Tư Gia Nhà Gái",
    address: "TDP Tân Tiến - Xã Kiến Xương - Hưng Yên",
    mapLink: "",
  },
  groom: {
    ceremonyTitle: "Lễ Thành Hôn",
    mainName1: "Văn Hiếu",
    mainName2: "Phương Huyền",
    date: "22.02.2026",
    lunarDate: "(Tức ngày 06 tháng 01 năm Bính Ngọ)",
    time1: { label: "Tiệc Mừng", value: "09:00" },
    time2: { label: "Lễ Thành Hôn", value: "11:30" },
    locationTitle: "Tại Tư Gia Nhà Trai",
    address: "Thôn Thái Công Nam - Xã Hồng Vũ - Hưng Yên",
    mapLink: "",
  },
};

const data = computed(() => WEDDING_DATA[props.side]);
const isOpened = ref(false);

// --- LOGIC PHẢN XẠ ÁNH SÁNG MỚI (Dựa trên Position X/Y) ---
// Giá trị đích (Target)
let targetX = 50; // %
let targetY = 50; // %

// Giá trị hiện tại (Current - dùng để lerp cho mượt)
let currentX = 50;
let currentY = 50;
let animationFrameId: number;

const handleDeviceOrientation = (event: DeviceOrientationEvent) => {
  // Gamma (Trái/Phải): -90 đến 90.
  // Ta map khoảng -45 đến 45 thành 0% đến 100% position X
  const gamma = event.gamma || 0;
  // Beta (Trước/Sau): -180 đến 180. Trừ 45 độ (góc cầm tay).
  // Ta map khoảng -45 đến 45 thành 0% đến 100% position Y
  const beta = (event.beta || 0) - 45;

  // Công thức Map: (Value + Max) / (Max * 2) * 100
  // Đảo ngược dấu (-) ở gamma để nghiêng trái thì sáng chạy sang trái (tự nhiên hơn)
  targetX = 50 + gamma * 1.5;
  targetY = 50 + beta * 1.5;

  // Clamp giá trị trong khoảng 0-100 để không bị vỡ background
  targetX = Math.max(0, Math.min(100, targetX));
  targetY = Math.max(0, Math.min(100, targetY));
};

// Fallback chuột (cho Desktop)
const handleMouseMove = (event: MouseEvent) => {
  const { innerWidth, innerHeight } = window;
  const x = event.clientX / innerWidth;
  const y = event.clientY / innerHeight;

  // Map chuột 0-1 thành 0-100%
  targetX = x * 100;
  targetY = y * 100;
};

const updateLightLoop = () => {
  // Lerp để chuyển động mượt mà (Hệ số 0.08)
  currentX += (targetX - currentX) * 0.08;
  currentY += (targetY - currentY) * 0.08;

  if (containerRef.value) {
    // Cập nhật biến CSS --shine-x và --shine-y
    containerRef.value.style.setProperty("--shine-x", `${currentX}%`);
    containerRef.value.style.setProperty("--shine-y", `${currentY}%`);
  }
  animationFrameId = requestAnimationFrame(updateLightLoop);
};

// --- Refs & Animation ---
const containerRef = ref<HTMLElement | null>(null);
const coverRef = ref<HTMLElement | null>(null);
const innerRef = ref<HTMLElement | null>(null);
const contentElementsRef = ref<HTMLElement | null>(null);
const openButtonRef = ref<HTMLElement | null>(null);
const closeButtonRef = ref<HTMLElement | null>(null);
let ctx: gsap.Context;
let tl: gsap.core.Timeline;

const openInvitation = () => {
  isOpened.value = true;
  if (tl) tl.play();
};

const closeInvitation = () => {
  if (tl) tl.reverse();
};

defineExpose({ isOpened, closeInvitation });

onMounted(() => {
  updateLightLoop(); // Chạy loop ngay lập tức

  if (window.DeviceOrientationEvent) {
    window.addEventListener("deviceorientation", handleDeviceOrientation, true);
  }
  window.addEventListener("mousemove", handleMouseMove);

  ctx = gsap.context(() => {
    tl = gsap.timeline({
      paused: true,
      onReverseComplete: () => {
        isOpened.value = false;
      },
    });

    tl.to([openButtonRef.value, ".cover-content"], {
      opacity: 0,
      scale: 0.95,
      duration: 0.5,
      pointerEvents: "none",
      ease: "power2.inOut",
    })
      .to(coverRef.value, { y: "-100%", duration: 1.8, ease: "power3.inOut" })
      .fromTo(
        innerRef.value,
        { scale: 0.95, opacity: 0 },
        { scale: 1, opacity: 1, duration: 1.5, ease: "power2.out" },
        "-=1.2",
      )
      .from(
        contentElementsRef.value!.children,
        { y: 20, opacity: 0, duration: 1, stagger: 0.15, ease: "power2.out" },
        "-=0.5",
      )
      .to(closeButtonRef.value, {
        opacity: 1,
        scale: 1,
        pointerEvents: "auto",
        duration: 0.5,
      });
  }, containerRef.value!);
});

onUnmounted(() => {
  if (window.DeviceOrientationEvent) {
    window.removeEventListener("deviceorientation", handleDeviceOrientation);
  }
  window.removeEventListener("mousemove", handleMouseMove);
  cancelAnimationFrame(animationFrameId);
  ctx?.revert();
});
</script>

<template>
  <div
    ref="containerRef"
    class="relative h-dvh w-full overflow-hidden bg-[#FFFBF0] font-serif text-[#3E2723]"
    style="--shine-x: 50%; --shine-y: 50%"
  >
    <div
      class="pointer-events-none fixed inset-0 z-9999 opacity-40 mix-blend-multiply"
      style="
        background-image: url(&quot;https://www.transparenttextures.com/patterns/cream-paper.png&quot;);
      "
    ></div>

    <div
      ref="closeButtonRef"
      @click="closeInvitation"
      class="absolute right-5 top-5 z-50 cursor-pointer rounded-full border border-red-800/30 bg-white/50 p-2 text-red-800 shadow-sm opacity-0 scale-90 backdrop-blur-sm pointer-events-none hover:bg-red-800 hover:text-white transition-all"
    >
      <svg
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
        stroke-width="2"
        stroke="currentColor"
        class="h-6 w-6"
      >
        <path
          stroke-linecap="round"
          stroke-linejoin="round"
          d="M6 18L18 6M6 6l12 12"
        />
      </svg>
    </div>

    <div
      ref="innerRef"
      class="absolute inset-0 z-0 flex flex-col items-center justify-center bg-[#FFFBF0]"
    >
      <div
        class="pointer-events-none absolute inset-0 bg-[url('https://www.transparenttextures.com/patterns/ornament.png')] opacity-10"
      ></div>
      <div
        class="pointer-events-none absolute inset-3 border-4 border-double border-yellow-700/40 md:inset-8"
      ></div>

      <div
        ref="contentElementsRef"
        class="relative z-10 flex h-full w-full flex-col items-center justify-between px-6 py-12 text-center md:py-16"
      >
        <div class="shrink-0">
          <p
            class="font-cinzel mb-1 text-[10px] uppercase text-yellow-700 md:mb-2 md:text-sm"
          >
            Trân trọng kính mời
          </p>
          <h2 class="font-serif text-sm italic text-[#5d4037] md:text-lg">
            Tới dự {{ data.ceremonyTitle }} của hai con chúng tôi
          </h2>
        </div>

        <div
          class="font-great-vibes shrink-0 py-1 text-5xl leading-tight drop-shadow-sm md:py-2 md:text-7xl"
        >
          <div class="gold-foil-text pb-1">{{ data.mainName1 }}</div>
          <div
            class="font-serif text-xl text-[#B8860B] italic md:text-3xl my-1"
          >
            &
          </div>
          <div class="gold-foil-text pb-1">{{ data.mainName2 }}</div>
        </div>

        <div
          class="relative w-full shrink-0 overflow-hidden border-y border-yellow-700/50 bg-yellow-700/5 py-3 md:py-5"
        >
          <div
            class="absolute inset-0 bg-[radial-gradient(circle_at_center,#B8860B_1px,transparent_1px)] bg-size-[10px_10px] opacity-30"
          />
          <div class="relative z-10">
            <div
              class="mb-2 border-b border-yellow-700/30 pb-2 md:mb-3 md:pb-3"
            >
              <div
                class="font-cinzel text-2xl font-bold tracking-[0.15em] text-red-900 md:text-5xl"
              >
                {{ data.date }}
              </div>
              <p
                class="font-cinzel mt-1 text-[9px] uppercase tracking-widest text-[#5d4037] md:mt-2 md:text-xs"
              >
                {{ data.lunarDate }}
              </p>
            </div>
            <div class="mt-2 flex flex-col space-y-2 px-4 md:px-8">
              <div
                class="flex items-baseline justify-between border-b border-dotted border-yellow-700/30 pb-1"
              >
                <span
                  class="font-cinzel text-xs uppercase text-[#5d4037] md:text-base font-bold"
                  >{{ data.time1.label }}</span
                >
                <span
                  class="font-cinzel text-lg font-bold text-red-900 md:text-2xl"
                  >{{ data.time1.value }}</span
                >
              </div>
              <div class="flex items-baseline justify-between">
                <span
                  class="font-cinzel text-xs uppercase text-[#5d4037] md:text-base font-bold"
                  >{{ data.time2.label }}</span
                >
                <span
                  class="font-cinzel text-lg font-bold text-red-900 md:text-2xl"
                  >{{ data.time2.value }}</span
                >
              </div>
            </div>
          </div>
        </div>

        <div class="shrink-0 space-y-1 md:space-y-2">
          <p
            class="font-cinzel mb-0.5 text-xs font-bold uppercase text-[#3e2723] md:mb-1 md:text-base"
          >
            {{ data.locationTitle }}
          </p>
          <p class="font-serif text-sm italic md:text-lg text-[#5d4037] px-4">
            {{ data.address }}
          </p>
          <a
            v-if="data.mapLink"
            :href="data.mapLink"
            target="_blank"
            class="inline-block mt-2 text-[10px] uppercase border border-[#B8860B] text-[#B8860B] px-3 py-1 rounded-full hover:bg-[#B8860B] hover:text-white transition-colors"
            >Xem bản đồ</a
          >
        </div>

        <div
          class="font-cinzel grid w-full shrink-0 grid-cols-2 gap-4 border-t border-yellow-700/30 pt-3 text-[9px] uppercase tracking-widest opacity-80 md:gap-6 md:pt-4 md:text-xs"
        >
          <div class="text-left">
            <span class="mb-1 block font-bold text-red-900">Nhà Gái</span>
            <p>Mẹ: Phạm Thị Báu</p>
          </div>
          <div class="text-right">
            <span class="mb-1 block font-bold text-red-900">Nhà Trai</span>
            <p>Bố: Lều Văn Toàn</p>
            <p>Mẹ: Vũ Thị Thuỷ</p>
          </div>
        </div>
      </div>
    </div>

    <div
      ref="coverRef"
      class="will-change-transform absolute inset-0 z-20 flex h-full w-full flex-col justify-between overflow-hidden bg-white shadow-2xl"
    >
      <div class="absolute inset-0 z-0">
        <img
          src="/images/cover.jpg"
          class="h-full w-full object-cover"
          alt="Cover"
          decoding="sync"
        />
      </div>

      <div
        class="cover-content relative z-30 flex w-full flex-col items-center bg-linear-to-b from-white via-white/80 to-transparent px-4 pb-12 pt-10 text-center backdrop-blur-[3px]"
        style="
          mask-image: linear-gradient(to bottom, black 60%, transparent 100%);
          -webkit-mask-image: linear-gradient(
            to bottom,
            black 60%,
            transparent 100%
          );
        "
      >
        <div class="mb-2 flex items-center justify-center space-x-4">
          <h2
            class="font-cinzel text-xs font-bold uppercase tracking-[4px] text-red-800"
          >
            Thiệp Mời
          </h2>
        </div>
        <div
          class="relative flex flex-col items-center justify-center py-4 text-red-800"
        >
          <span
            class="font-serif absolute left-1/2 top-[45%] -translate-x-1/2 -translate-y-1/2 select-none text-[8rem] leading-none text-[#B8860B] opacity-20 blur-[1px]"
            >&</span
          >
          <span
            class="font-great-vibes relative z-10 -mb-2 block text-5xl leading-normal red-foil-text px-2"
            >{{ data.mainName1 }}</span
          >
          <span
            class="font-great-vibes relative z-10 block text-5xl leading-normal red-foil-text px-2"
            >{{ data.mainName2 }}</span
          >
        </div>
        <p
          class="font-cinzel mt-2 border-y border-red-800/30 py-2 text-xs font-bold uppercase tracking-[4px] text-red-800"
        >
          22 . 02 . 2026
        </p>
      </div>

      <div
        ref="openButtonRef"
        @click="openInvitation"
        class="group relative z-30 mb-12 flex w-full cursor-pointer flex-col items-center"
      >
        <p
          class="font-cinzel mb-2 text-[9px] font-bold uppercase tracking-[3px] text-yellow-400 opacity-80 transition-all group-hover:tracking-[4px] group-hover:opacity-100 md:text-[10px]"
        >
          Chạm để mở
        </p>
        <div
          class="animate-pulse-slow relative z-10 flex h-14 w-14 items-center justify-center rounded-full border-2 border-red-800 bg-white/40 shadow-lg backdrop-blur-md transition-colors duration-500 group-hover:bg-red-800"
        >
          <div
            class="absolute inset-1 scale-90 rounded-full border border-red-800/40 transition-transform duration-500 group-hover:scale-100 group-hover:border-white"
          ></div>
          <svg
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke-width="1.5"
            stroke="#991B1B"
            class="h-7 w-7 transition-transform duration-500 group-hover:translate-y-0.5 group-hover:stroke-white"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M19.5 8.25l-7.5 7.5-7.5-7.5"
            />
          </svg>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* LOGIC MỚI: 
  - Dùng linear-gradient chéo (135deg).
  - Background size cực lớn (250%).
  - Di chuyển background-position theo biến --shine-x và --shine-y
*/

/* Red Foil (Cho bìa) */
.red-foil-text {
  background-image: linear-gradient(
    135deg,
    /* Góc cố định */ #880e4f 0%,
    #880e4f 20%,
    #d32f2f 40%,
    #ffcdd2 50%,
    /* Vệt sáng trắng hồng ở giữa */ #d32f2f 60%,
    #880e4f 80%,
    #880e4f 100%
  );
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  filter: drop-shadow(0 2px 0px rgba(255, 255, 255, 0.3));

  /* Quan trọng: Size lớn để có chỗ cho position di chuyển */
  background-size: 250% 250%;

  /* Nhận giá trị từ JS */
  background-position: var(--shine-x, 50%) var(--shine-y, 50%);

  /* Tối ưu hiệu năng */
  will-change: background-position;
}

/* Gold Foil (Cho nội dung bên trong) */
.gold-foil-text {
  background-image: linear-gradient(
    135deg,
    #8a6e2f 0%,
    #8a6e2f 20%,
    #e3d278 40%,
    #ffffe0 50%,
    /* Vệt sáng vàng nhạt ở giữa */ #e3d278 60%,
    #8a6e2f 80%,
    #8a6e2f 100%
  );
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  filter: drop-shadow(0 1px 0px rgba(139, 114, 55, 0.4));

  background-size: 250% 250%;
  background-position: var(--shine-x, 50%) var(--shine-y, 50%);
  will-change: background-position;
}

.animate-pulse-slow {
  animation: pulse 3s cubic-bezier(0.4, 0, 0.6, 1) infinite;
}
@keyframes pulse {
  0%,
  100% {
    transform: scale(1);
    box-shadow: 0 0 0 0 rgba(153, 27, 27, 0.4);
  }
  50% {
    transform: scale(0.95);
    box-shadow: 0 0 0 8px rgba(153, 27, 27, 0);
  }
}
</style>
