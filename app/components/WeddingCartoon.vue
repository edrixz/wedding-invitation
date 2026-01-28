<script setup lang="ts">
import { onMounted, onUnmounted, ref, computed } from "vue";
import gsap from "gsap";

const props = defineProps<{ side: "bride" | "groom" }>();

const WEDDING_DATA = {
  bride: {
    ceremonyTitle: "Lễ Vu Quy",
    mainName1: "Phương Huyền",
    mainName2: "Văn Hiếu",
    date: "22.02.2026",
    lunarDate: "(Tức ngày 06 tháng 01 năm Bính Ngọ)",
    time1: { label: "Bữa Cơm Thân Mật", value: "09:00", icon: "🍚" },
    time2: { label: "Lễ Vu Quy", value: "11:30", icon: "💍" },
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
    time1: { label: "Tiệc Mừng", value: "09:00", icon: "🍷" },
    time2: { label: "Lễ Thành Hôn", value: "11:30", icon: "💞" },
    locationTitle: "Tại Tư Gia Nhà Trai",
    address: "Thôn Thái Công Nam - Xã Hồng Vũ - Hưng Yên",
    mapLink: "",
  },
};

const data = computed(() => WEDDING_DATA[props.side]);
const isOpened = ref(false);

const containerRef = ref<HTMLElement | null>(null);
const coverRef = ref<HTMLElement | null>(null);
const innerRef = ref<HTMLElement | null>(null);
const contentElementsRef = ref<HTMLElement | null>(null);
const openButtonRef = ref<HTMLElement | null>(null);
const closeButtonRef = ref<HTMLElement | null>(null);

let ctx: gsap.Context;
let tl: gsap.core.Timeline;
const paperTexture =
  "https://www.transparenttextures.com/patterns/cream-paper.png";

const openInvitation = () => {
  isOpened.value = true;
  if (tl) tl.play();
};

const closeInvitation = () => {
  if (tl) tl.reverse();
};

defineExpose({
  isOpened,
  closeInvitation,
});

onMounted(() => {
  ctx = gsap.context(() => {
    tl = gsap.timeline({
      paused: true,
      onReverseComplete: () => {
        isOpened.value = false;
      },
    });

    tl.to([openButtonRef.value, ".cover-content"], {
      opacity: 0,
      scale: 0.8,
      duration: 0.3,
      ease: "back.in(1.7)",
      pointerEvents: "none",
    })
      .to(coverRef.value, { y: "-100%", duration: 1.2, ease: "power4.inOut" })
      .fromTo(
        innerRef.value,
        { scale: 0.8, rotation: -2 },
        { scale: 1, rotation: 0, duration: 1.2, ease: "elastic.out(1, 0.5)" },
        "-=0.8",
      )
      .from(
        contentElementsRef.value!.children,
        { y: 50, opacity: 0, duration: 0.8, stagger: 0.1, ease: "back.out(2)" },
        "-=0.8",
      )
      .to(closeButtonRef.value, {
        opacity: 1,
        scale: 1,
        pointerEvents: "auto",
        duration: 0.4,
      });
  }, containerRef.value!);
});

onUnmounted(() => ctx?.revert());
</script>

<template>
  <div
    ref="containerRef"
    class="w-full h-dvh bg-black overflow-hidden relative font-mali text-[#5d4037]"
  >
    <div
      ref="closeButtonRef"
      @click="closeInvitation"
      class="absolute top-6 right-6 z-50 cursor-pointer opacity-0 scale-50 pointer-events-none p-2 rounded-full bg-[#3E2723] border-2 border-[#FFD54F] hover:bg-[#5D4037] active:scale-90 transition-transform shadow-cartoon text-[#FFD54F]"
    >
      <svg
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
        stroke-width="4"
        stroke="currentColor"
        class="w-6 h-6"
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
      :style="{ backgroundImage: `url('${paperTexture}')` }"
    >
      <div
        ref="contentElementsRef"
        class="relative z-10 flex flex-col items-center text-center space-y-3 md:space-y-5 px-6 md:px-0 w-full max-w-md"
      >
        <div>
          <p
            class="text-xs md:text-sm uppercase tracking-[2px] mb-1 font-bold opacity-70"
          >
            Trân trọng kính mời
          </p>
          <h2 class="text-sm md:text-base font-bold">
            Tới dự {{ data.ceremonyTitle }} của hai con chúng tôi
          </h2>
        </div>

        <div
          class="text-4xl md:text-6xl text-[#ff0000] leading-tight py-2 font-bold drop-shadow-sm transform -rotate-2"
        >
          <span class="block">{{ data.mainName1 }}</span>
          <span class="text-2xl md:text-4xl text-[#3E2723] inline-block my-1"
            >&</span
          >
          <span class="block">{{ data.mainName2 }}</span>
        </div>

        <div
          class="w-full border-2 border-[#3E2723] py-3 bg-[#FFECB3] rounded-xl shadow-cartoon relative overflow-hidden"
        >
          <div
            class="absolute top-0 left-0 w-full h-2 bg-[#FFF8E1] opacity-50"
          ></div>
          <div class="mb-2 border-b-2 border-[#3E2723]/20 pb-2 border-dashed">
            <div
              class="text-3xl md:text-5xl font-bold text-[#BF360C] tracking-widest"
            >
              {{ data.date }}
            </div>
            <p
              class="text-[10px] md:text-xs uppercase tracking-widest font-bold mt-1 opacity-80"
            >
              {{ data.lunarDate }}
            </p>
          </div>
          <div class="flex flex-col space-y-2 mt-2 px-4">
            <div
              class="flex justify-between items-center bg-white/40 p-1.5 rounded-lg"
            >
              <div class="flex items-center space-x-2">
                <span class="text-lg">{{ data.time1.icon }}</span
                ><span class="font-bold text-xs md:text-sm uppercase"
                  >{{ data.time1.label }}:</span
                >
              </div>
              <span class="text-xl md:text-2xl font-bold text-[#3E2723]">{{
                data.time1.value
              }}</span>
            </div>
            <div
              class="flex justify-between items-center bg-white/40 p-1.5 rounded-lg"
            >
              <div class="flex items-center space-x-2">
                <span class="text-lg">{{ data.time2.icon }}</span
                ><span class="font-bold text-xs md:text-sm uppercase"
                  >{{ data.time2.label }}:</span
                >
              </div>
              <span class="text-xl md:text-2xl font-bold text-[#D84315]">{{
                data.time2.value
              }}</span>
            </div>
          </div>
        </div>

        <div>
          <p
            class="font-bold uppercase text-[#3e2723] text-xs md:text-sm mb-1 bg-[#FFD54F] inline-block px-2 py-0.5 rounded-md border border-[#3E2723] shadow-sm transform rotate-1"
          >
            {{ data.locationTitle }}
          </p>
          <p class="text-sm md:text-base mt-1 font-bold">
            {{ data.address }}
          </p>
          <a
            v-if="data.mapLink"
            :href="data.mapLink"
            target="_blank"
            class="inline-block mt-2 text-[10px] uppercase font-bold text-white bg-[#5D4037] px-3 py-1 rounded-full hover:bg-[#3E2723] transition-colors shadow-sm"
            >📍 Xem bản đồ</a
          >
        </div>

        <div
          class="w-full grid grid-cols-2 gap-4 text-[10px] md:text-xs uppercase tracking-wide border-t-2 border-[#3E2723]/20 pt-3 opacity-90 font-bold"
        >
          <div class="text-left">
            <span class="block text-[#BF360C] mb-1">Nhà Gái</span>
            <p>Mẹ: Phạm Thị Báu</p>
          </div>
          <div class="text-right">
            <span class="block text-[#BF360C] mb-1">Nhà Trai</span>
            <p>Bố: Lều Văn Toàn</p>
            <p>Mẹ: Vũ Thị Thuỷ</p>
          </div>
        </div>
      </div>
    </div>

    <div
      ref="coverRef"
      class="absolute inset-0 z-20 w-full h-full shadow-[0_10px_30px_rgba(0,0,0,0.5)] will-change-transform overflow-hidden flex flex-col justify-between bg-[#2b0a0a]"
    >
      <div class="absolute inset-0 z-0">
        <img
          src="/images/cover-anime.jpg"
          class="w-full h-full object-cover scale-105"
          alt="Cover"
        />
        <div
          class="absolute inset-0 bg-linear-to-b from-[#3E2723]/20 via-transparent to-[#3E2723]/70 mix-blend-multiply"
        ></div>
      </div>
      <div
        class="cover-content relative z-30 w-full pt-16 md:pt-20 px-4 flex flex-col items-center"
      >
        <img
          src="/images/title.png"
          alt="Phương Huyền & Văn Hiếu"
          class="w-[90%] md:w-[70%] object-contain drop-shadow-xl animate-float-slow"
        />
      </div>
      <div
        ref="openButtonRef"
        @click="openInvitation"
        class="relative z-30 mb-8 flex flex-col items-center cursor-pointer animate-bounce-slow w-full hover:scale-105 transition-transform"
      >
        <div
          class="w-[6vh] h-[6vh] rounded-full bg-[#FFD54F] border-[0.5vh] border-[#3E2723] flex items-center justify-center shadow-cartoon-heavy relative z-10"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke-width="4"
            stroke="currentColor"
            class="w-[3vh] h-[3vh] animate-pulse"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              d="M4.5 15.75l7.5-7.5 7.5 7.5"
            />
          </svg>
        </div>
        <p
          class="text-[#FFD54F] text-sm mt-2 font-bold shadow-black drop-shadow-md tracking-wider"
        >
          MỞ THIỆP
        </p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.shadow-cartoon {
  box-shadow: 4px 4px 0px #3e2723;
}
.shadow-cartoon-heavy {
  box-shadow: 0.6vh 0.6vh 0px #261410;
}
.animate-bounce-slow {
  animation: bounce 2s infinite;
}
.animate-float-slow {
  animation: float 4s ease-in-out infinite;
}
@keyframes float {
  0%,
  100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-10px);
  }
}
</style>
