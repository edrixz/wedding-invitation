<script setup lang="ts">
import { onMounted, onUnmounted, ref, computed } from "vue";
import gsap from "gsap";

// Refs
const containerRef = ref<HTMLElement | null>(null);
const coverRef = ref<HTMLElement | null>(null);
const innerRef = ref<HTMLElement | null>(null);
const contentElementsRef = ref<HTMLElement | null>(null);
const openButtonRef = ref<HTMLElement | null>(null);
const closeButtonRef = ref<HTMLElement | null>(null);

// State
const isOpened = ref(false);
const tiltAngle = ref(135); // Góc ánh sáng mặc định
const hasSensorAccess = ref(false);

// --- 1. LOGIC TÍNH TOÁN GÓC NGHIÊNG 360 ĐỘ ---
const handleDeviceOrientation = (event: DeviceOrientationEvent) => {
  const gamma = event.gamma || 0; // Nghiêng trái/phải
  // Trừ 45 độ ở trục beta để lấy vị trí cầm điện thoại tự nhiên làm mốc chuẩn
  const beta = (event.beta || 0) - 45;

  // Math.atan2 giúp tính góc xoay tròn 360 độ dựa trên cả 2 trục
  // Kết quả giúp ánh sáng xoay vòng quanh chữ khi bạn lắc điện thoại
  const angleRad = Math.atan2(gamma, beta);
  const angleDeg = angleRad * (180 / Math.PI);

  // Offset 135 độ để ánh sáng chiếu chéo đẹp nhất
  tiltAngle.value = 135 - angleDeg;
};

// --- 2. XIN QUYỀN TRUY CẬP (IOS 13+) ---
const requestMotionPermission = async () => {
  if (hasSensorAccess.value) return;

  if (
    typeof DeviceOrientationEvent !== "undefined" &&
    typeof (DeviceOrientationEvent as any).requestPermission === "function"
  ) {
    try {
      const permissionState = await (
        DeviceOrientationEvent as any
      ).requestPermission();
      if (permissionState === "granted") {
        window.addEventListener(
          "deviceorientation",
          handleDeviceOrientation,
          true,
        );
        hasSensorAccess.value = true;
      } else {
        startAutoShine();
      }
    } catch (e) {
      startAutoShine();
    }
  } else {
    if (window.DeviceOrientationEvent) {
      window.addEventListener(
        "deviceorientation",
        handleDeviceOrientation,
        true,
      );
      hasSensorAccess.value = true;
    } else {
      startAutoShine();
    }
  }
};

// --- 3. TỰ ĐỘNG LẤP LÁNH (FALLBACK) ---
const startAutoShine = () => {
  let angle = 135;
  setInterval(() => {
    angle += 2; // Xoay nhanh hơn chút để thấy rõ hiệu ứng lóe
    tiltAngle.value = angle;
  }, 30);
};

// --- STYLE MẠ VÀNG LÓE SÁNG (HIGH CONTRAST) ---
const dynamicGoldStyle = computed(() => ({
  // Gradient: Nâu -> Vàng -> TRẮNG (Điểm lóe sáng) -> Vàng -> Nâu
  backgroundImage: `linear-gradient(${tiltAngle.value}deg, #FFD700 0%, #FFED4E 25%, #C5A028 50%, #FFD700 75%, #FFED4E 100%)`,
  webkitBackgroundClip: "text",
  backgroundClip: "text",
  color: "transparent",
  // Bóng đổ nhẹ tách nền
  filter: "drop-shadow(0px 1px 0px rgba(100, 80, 20, 0.4))",
  // Kích thước background lớn để dải màu mượt
  backgroundSize: "200% auto",
  willChange: "background-image", // Tối ưu hiệu năng render
}));

let ctx: gsap.Context;
let tl: gsap.core.Timeline;

const paperTexture =
  "https://www.transparenttextures.com/patterns/cream-paper.png";

const openInvitation = async () => {
  await requestMotionPermission();
  isOpened.value = true;
  if (tl) tl.play();
};

const closeInvitation = () => {
  if (tl) tl.reverse();
};

onMounted(() => {
  // Kích hoạt lắng nghe cho Android/PC ngay khi vào
  if (
    !hasSensorAccess.value &&
    window.DeviceOrientationEvent &&
    typeof (DeviceOrientationEvent as any).requestPermission !== "function"
  ) {
    window.addEventListener("deviceorientation", handleDeviceOrientation, true);
  } else {
    // Nếu là PC không có cảm biến thì chạy auto luôn
    startAutoShine();
  }

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
      .to(coverRef.value, {
        y: "-100%",
        duration: 1.8,
        ease: "power3.inOut",
      })
      .fromTo(
        innerRef.value,
        { scale: 0.95, opacity: 0 },
        { scale: 1, opacity: 1, duration: 1.5, ease: "power2.out" },
        "-=1.2",
      )
      .from(
        contentElementsRef.value!.children,
        {
          y: 20,
          opacity: 0,
          duration: 1,
          stagger: 0.15,
          ease: "power2.out",
        },
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
  ctx?.revert();
});
</script>

<template>
  <div
    ref="containerRef"
    class="relative h-dvh w-full overflow-hidden bg-[#1a0505] font-serif text-[#3E2723]"
  >
    <div
      class="pointer-events-none fixed inset-0 z-9999 opacity-40 mix-blend-multiply"
      :style="{ backgroundImage: `url('${paperTexture}')` }"
    ></div>

    <div
      ref="closeButtonRef"
      @click="closeInvitation"
      class="absolute right-5 top-5 z-50 cursor-pointer rounded-full border border-red-800/30 bg-white/50 p-2 text-red-800 shadow-sm opacity-0 scale-90 backdrop-blur-sm transition-all duration-300 pointer-events-none hover:bg-red-800 hover:text-white"
    >
      <svg
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
        stroke-width="2"
        stroke="currentColor"
        class="h-5 w-5 md:h-8 md:w-8"
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
        class="relative z-10 flex h-full w-full max-w-md flex-col items-center justify-between px-6 py-12 text-center md:py-16 md:space-y-7 md:justify-center md:h-auto"
      >
        <div class="shrink-0">
          <p
            class="font-cinzel mb-1 text-[10px] uppercase tracking-[3px] text-yellow-700 md:mb-2 md:text-sm"
          >
            Trân trọng kính mời
          </p>
          <h2 class="font-serif text-sm italic text-[#5d4037] md:text-lg">
            Tới dự lễ Vu Quy của hai con chúng tôi
          </h2>
        </div>

        <div
          class="font-pinyon shrink-0 py-1 text-5xl leading-tight drop-shadow-sm md:py-2 md:text-7xl"
        >
          <div :style="dynamicGoldStyle" class="pb-1">Phương Huyền</div>
          <div
            class="font-serif text-xl text-[#B8860B] italic md:text-3xl my-1"
          >
            &
          </div>
          <div :style="dynamicGoldStyle" class="pb-1">Văn Hiếu</div>
        </div>

        <div
          class="relative w-full shrink-0 overflow-hidden border-y border-yellow-700/50 bg-yellow-700/5 py-2 md:py-4"
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
                22.02.2026
              </div>
              <p
                class="font-cinzel mt-1 text-[9px] uppercase tracking-widest text-[#5d4037] md:mt-2 md:text-xs"
              >
                (Tức ngày 06 tháng 01 năm Bính Ngọ)
              </p>
            </div>
            <div
              class="mt-2 flex flex-col space-y-1 px-4 md:mt-4 md:space-y-3 md:px-6"
            >
              <div
                class="flex items-baseline justify-between border-b border-dotted border-yellow-700/30 pb-1"
              >
                <span
                  class="font-cinzel text-xs uppercase text-[#5d4037] md:text-base"
                  >Bữa Cơm Thân Mật</span
                >
                <span
                  class="font-cinzel text-lg font-bold text-red-900 md:text-2xl"
                  >09:00</span
                >
              </div>
              <div class="flex items-baseline justify-between">
                <span
                  class="font-cinzel text-xs uppercase text-[#5d4037] md:text-base"
                  >Lễ Vu Quy</span
                >
                <span
                  class="font-cinzel text-lg font-bold text-red-900 md:text-2xl"
                  >11:30</span
                >
              </div>
            </div>
          </div>
        </div>

        <div class="shrink-0 space-y-2 md:space-y-4">
          <div>
            <p
              class="font-cinzel mb-0.5 text-xs font-bold uppercase text-[#3e2723] md:mb-1 md:text-base"
            >
              Tại Tư Gia Nhà Gái
            </p>
            <p class="font-serif text-xs italic md:text-base">
              TDP Tân Tiến - Xã Kiến Xương - Hưng Yên
            </p>
          </div>

          <div>
            <p
              class="font-cinzel mb-0.5 text-xs font-bold uppercase text-[#3e2723] md:mb-1 md:text-base"
            >
              Nhà Trai
            </p>
            <p class="font-serif text-xs italic md:text-base">
              Thôn Thái Công Nam - Xã Hồng Vũ - Hưng Yên
            </p>
          </div>
        </div>

        <div
          class="font-cinzel grid w-full shrink-0 grid-cols-2 gap-4 border-t border-yellow-700/30 pt-2 text-[9px] uppercase tracking-widest opacity-80 md:gap-6 md:pt-4 md:text-xs"
        >
          <div class="text-left">
            <span class="mb-1 block font-bold text-red-900 md:mb-2"
              >Nhà Gái</span
            >
            <p>Mẹ: Phạm Thị Báu</p>
          </div>
          <div class="text-right">
            <span class="mb-1 block font-bold text-red-900 md:mb-2"
              >Nhà Trai</span
            >
            <p>Bố: Lều Văn Toàn</p>
            <p>Mẹ: Vũ Thị Thuỷ</p>
          </div>
        </div>
      </div>
    </div>

    <div
      ref="coverRef"
      class="will-change-transform absolute inset-0 z-20 flex h-full w-full flex-col justify-between overflow-hidden bg-[#1a0505] shadow-2xl"
    >
      <div class="absolute inset-0 z-0">
        <img
          src="/images/cover.jpg"
          class="h-full w-full object-cover"
          alt="Cover Real"
        />
      </div>

      <div
        class="cover-content relative z-30 flex w-full flex-col items-center bg-linear-to-b from-white via-white/80 to-transparent px-4 pb-12 pt-16 text-center backdrop-blur-[2px] md:pb-16 md:pt-20"
      >
        <div
          class="mb-2 flex items-center justify-center space-x-4 opacity-100"
        >
          <div class="h-px w-8 bg-red-800 md:w-12"></div>
          <h2
            class="font-cinzel text-xs font-bold uppercase tracking-[4px] text-red-800 md:tracking-[6px] md:text-base"
          >
            Thiệp Mời
          </h2>
          <div class="h-px w-8 bg-red-800 md:w-12"></div>
        </div>

        <div
          class="relative flex flex-col items-center justify-center py-4 text-red-800"
        >
          <span
            class="font-serif absolute left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2 select-none text-[8rem] leading-none text-red-600 opacity-20 blur-[1px] md:text-[12rem] z-0"
          >
            &
          </span>

          <span
            class="font-pinyon relative z-10 -mb-2 block text-5xl leading-normal md:-mb-3 md:text-8xl"
          >
            Phương Huyền
          </span>

          <span
            class="font-pinyon relative z-10 block text-5xl leading-normal md:text-8xl"
          >
            Văn Hiếu
          </span>
        </div>

        <p
          class="font-cinzel mt-2 border-y border-red-800/30 py-2 text-xs font-bold uppercase tracking-[4px] text-red-800 md:text-base md:tracking-[5px]"
        >
          22 . 02 . 2026
        </p>
      </div>

      <div
        ref="openButtonRef"
        @click="openInvitation"
        class="group relative z-30 mb-8 flex w-full cursor-pointer flex-col items-center"
      >
        <p
          class="font-cinzel mb-2 text-[9px] font-bold uppercase tracking-[3px] text-yellow-400 opacity-80 transition-all group-hover:tracking-[4px] group-hover:opacity-100 md:text-[10px]"
        >
          Chạm để mở
        </p>

        <div
          class="animate-pulse-slow relative z-10 flex h-10 w-10 items-center justify-center rounded-full border-2 border-[#B8860B] bg-white/40 shadow-lg backdrop-blur-md transition-colors duration-500 group-hover:bg-[#B8860B] md:h-14 md:w-14"
        >
          <div
            class="absolute inset-1 scale-90 rounded-full border border-[#B8860B]/40 transition-transform duration-500 group-hover:scale-100 group-hover:border-white"
          ></div>
          <svg
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke-width="1.5"
            stroke="#B8860B"
            class="h-5 w-5 transition-transform duration-500 group-hover:translate-y-0.5 group-hover:stroke-white md:h-7 md:w-7"
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
@import url("https://fonts.googleapis.com/css2?family=Cinzel:wght@400;700&family=Cormorant+Garamond:ital,wght@0,400;0,600;1,400;1,600&family=Great+Vibes&display=swap");

/* Áp dụng font */
.font-cinzel {
  font-family: "Cinzel", serif;
}

/* CHÍNH XÁC FONT BẠN YÊU CẦU: GREAT VIBES */
.font-pinyon {
  font-family: "Great Vibes", cursive;
  transform: scale(1.1);
  transform-origin: center;
}

/* CHÍNH XÁC FONT BẠN YÊU CẦU: CORMORANT GARAMOND */
.font-serif {
  font-family: "Cormorant Garamond", serif;
  font-weight: 600;
}

.animate-pulse-slow {
  animation: pulse 3s cubic-bezier(0.4, 0, 0.6, 1) infinite;
}

@keyframes pulse {
  0%,
  100% {
    transform: scale(1);
    box-shadow: 0 0 0 0 rgba(184, 134, 11, 0.4);
  }
  50% {
    transform: scale(0.95);
    box-shadow: 0 0 0 8px rgba(184, 134, 11, 0);
  }
}
</style>
