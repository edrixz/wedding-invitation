<script setup lang="ts">
import { onMounted, onUnmounted, ref, computed } from "vue";
import gsap from "gsap";

// --- CẤU HÌNH DỮ LIỆU CHO 2 NHÀ ---
const WEDDING_DATA = {
  bride: {
    // NHÀ GÁI
    ceremonyTitle: "Lễ Vu Quy",
    mainName1: "Phương Huyền", // Tên đứng trước
    mainName2: "Văn Hiếu", // Tên đứng sau
    date: "22.02.2026",
    lunarDate: "(Tức ngày 06 tháng 01 năm Bính Ngọ)",
    time1: { label: "Bữa Cơm Thân Mật", value: "09:00" },
    time2: { label: "Lễ Vu Quy", value: "11:30" }, // Giờ đón dâu
    locationTitle: "Tại Tư Gia Nhà Gái",
    address: "TDP Tân Tiến - Xã Kiến Xương - Hưng Yên",
    mapLink: "#", // Link Google Map nhà gái
  },
  groom: {
    // NHÀ TRAI
    ceremonyTitle: "Lễ Thành Hôn",
    mainName1: "Văn Hiếu", // Tên Chú rể đứng trước
    mainName2: "Phương Huyền", // Tên Cô dâu đứng sau
    date: "22.02.2026",
    lunarDate: "(Tức ngày 06 tháng 01 năm Bính Ngọ)",
    time1: { label: "Tiệc Mừng", value: "10:30" },
    time2: { label: "Lễ Thành Hôn", value: "12:00" }, // Giờ làm lễ
    locationTitle: "Tại Tư Gia Nhà Trai",
    address: "Thôn Thái Công Nam - Xã Hồng Vũ - Hưng Yên",
    mapLink: "#", // Link Google Map nhà trai
  },
};

// State
const isOpened = ref(false);
const currentSide = ref<"bride" | "groom">("bride"); // Mặc định

// Computed: Lấy dữ liệu dựa trên bên được chọn
const data = computed(() => WEDDING_DATA[currentSide.value]);

// --- LOGIC VẬT LÝ ÁNH SÁNG (GYROSCOPE) ---
const tiltAngle = ref(135);
const handleDeviceOrientation = (event: DeviceOrientationEvent) => {
  const gamma = event.gamma || 0;
  const beta = (event.beta || 0) - 45;
  const angleRad = Math.atan2(gamma, beta);
  tiltAngle.value = 135 - angleRad * (180 / Math.PI);
};

// Style Mạ Vàng Động
const dynamicGoldStyle = computed(() => ({
  backgroundImage: `linear-gradient(${tiltAngle.value}deg, #8a6e2f 0%, #e3d278 20%, #ffffe0 48%, #ffffe0 52%, #e3d278 80%, #8a6e2f 100%)`,
  webkitBackgroundClip: "text",
  backgroundClip: "text",
  color: "transparent",
  filter: "drop-shadow(0px 1px 0px rgba(139, 114, 55, 0.4))",
  backgroundSize: "200% auto",
  willChange: "background-image",
}));

// Refs & GSAP
const containerRef = ref<HTMLElement | null>(null);
const coverRef = ref<HTMLElement | null>(null);
const innerRef = ref<HTMLElement | null>(null);
const contentElementsRef = ref<HTMLElement | null>(null);
const closeButtonRef = ref<HTMLElement | null>(null);
let ctx: gsap.Context;
let tl: gsap.core.Timeline;

// Hàm mở thiệp theo phe (Nhà Trai / Nhà Gái)
const openFor = (side: "bride" | "groom") => {
  currentSide.value = side; // Cập nhật state
  isOpened.value = true;
  if (tl) tl.play();
};

const closeInvitation = () => {
  if (tl) tl.reverse();
};

onMounted(() => {
  if (window.DeviceOrientationEvent) {
    window.addEventListener("deviceorientation", handleDeviceOrientation, true);
  }

  ctx = gsap.context(() => {
    tl = gsap.timeline({
      paused: true,
      onReverseComplete: () => {
        isOpened.value = false;
      },
    });

    // Animation mở thiệp
    tl.to(".cover-actions", {
      opacity: 0,
      scale: 0.9,
      duration: 0.4,
      pointerEvents: "none",
    })
      .to(".cover-content", { opacity: 0, y: -20, duration: 0.4 }, "<")
      .to(coverRef.value, { y: "-100%", duration: 1.5, ease: "power3.inOut" })
      .fromTo(
        innerRef.value,
        { scale: 0.95, opacity: 0 },
        { scale: 1, opacity: 1, duration: 1.2, ease: "power2.out" },
        "-=1.0",
      )
      .from(
        contentElementsRef.value!.children,
        {
          y: 30,
          opacity: 0,
          duration: 0.8,
          stagger: 0.1,
          ease: "back.out(1.2)",
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
    style="--gold-angle: 135deg"
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
        class="relative z-10 flex h-full w-full max-w-md flex-col items-center justify-between px-6 py-12 text-center md:py-16"
      >
        <div class="shrink-0">
          <p
            class="font-cinzel mb-1 text-[10px] uppercase tracking-[3px] text-yellow-700 md:mb-2 md:text-sm"
          >
            Trân trọng kính mời
          </p>
          <h2 class="font-serif text-sm italic text-[#5d4037] md:text-lg">
            Tới dự {{ data.ceremonyTitle }} của hai con chúng tôi
          </h2>
        </div>

        <div
          class="font-pinyon shrink-0 py-1 text-5xl leading-tight drop-shadow-sm md:py-2 md:text-7xl"
        >
          <div :style="dynamicGoldStyle" class="pb-1">{{ data.mainName1 }}</div>
          <div
            class="font-serif text-xl text-[#B8860B] italic md:text-3xl my-1"
          >
            &
          </div>
          <div :style="dynamicGoldStyle" class="pb-1">{{ data.mainName2 }}</div>
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
      class="will-change-transform absolute inset-0 z-20 flex h-full w-full flex-col justify-between overflow-hidden bg-[#1a0505] shadow-2xl"
    >
      <div class="absolute inset-0 z-0">
        <img
          src="/images/cover.jpg"
          class="h-full w-full object-cover"
          alt="Cover"
        />
      </div>

      <div
        class="cover-content relative z-30 flex w-full flex-col items-center bg-linear-to-b from-white via-white/80 to-transparent px-4 pb-12 pt-16 text-center backdrop-blur-[2px]"
      >
        <div class="mb-2 flex items-center justify-center space-x-4">
          <div class="h-px w-8 bg-red-800"></div>
          <h2
            class="font-cinzel text-xs font-bold uppercase tracking-[4px] text-red-800"
          >
            Thiệp Mời
          </h2>
          <div class="h-px w-8 bg-red-800"></div>
        </div>
        <div
          class="relative flex flex-col items-center justify-center py-4 text-red-800"
        >
          <span
            class="font-serif absolute left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2 select-none text-[8rem] leading-none text-[#B8860B] opacity-20 blur-[1px]"
            >&</span
          >
          <span
            class="font-pinyon relative z-10 -mb-2 block text-5xl leading-normal"
            >Phương Huyền</span
          >
          <span class="font-pinyon relative z-10 block text-5xl leading-normal"
            >Văn Hiếu</span
          >
        </div>
        <p
          class="font-cinzel mt-2 border-y border-red-800/30 py-2 text-xs font-bold uppercase tracking-[4px] text-red-800"
        >
          22 . 02 . 2026
        </p>
      </div>

      <div
        class="cover-actions relative z-30 mb-12 flex flex-col items-center w-full px-8 space-y-4"
      >
        <p
          class="font-cinzel text-[10px] uppercase tracking-widest text-white mb-2 opacity-80"
        >
          Trân trọng kính mời
        </p>

        <div class="flex flex-col w-full max-w-xs space-y-3">
          <button
            @click="openFor('bride')"
            class="group relative w-full overflow-hidden rounded-full border border-[#B8860B]/50 bg-white/60 py-3 text-[#5d4037] shadow-lg backdrop-blur-md transition-all hover:bg-[#B8860B] hover:text-white hover:border-transparent active:scale-95"
          >
            <span
              class="font-cinzel text-xs font-bold uppercase tracking-[2px] group-hover:tracking-[3px] transition-all"
              >Khách Nhà Gái</span
            >
          </button>

          <button
            @click="openFor('groom')"
            class="group relative w-full overflow-hidden rounded-full border border-[#B8860B]/50 bg-white/60 py-3 text-[#5d4037] shadow-lg backdrop-blur-md transition-all hover:bg-[#B8860B] hover:text-white hover:border-transparent active:scale-95"
          >
            <span
              class="font-cinzel text-xs font-bold uppercase tracking-[2px] group-hover:tracking-[3px] transition-all"
              >Khách Nhà Trai</span
            >
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Cinzel:wght@400;700&family=Cormorant+Garamond:ital,wght@0,400;0,600;1,400;1,600&family=Great+Vibes&display=swap");

.font-cinzel {
  font-family: "Cinzel", serif;
}
.font-pinyon {
  font-family: "Great Vibes", cursive;
  transform: scale(1.1);
  transform-origin: center;
}
.font-serif {
  font-family: "Cormorant Garamond", serif;
  font-weight: 600;
}

/* Các style khác giữ nguyên */
</style>
