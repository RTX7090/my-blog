<script setup>
import { watch, onMounted, nextTick, ref } from "vue";
import { music, staticData } from "@/store";
import { storeToRefs } from "pinia";
import { gsapTransLyric } from "@/utils/transform";

import SpecialTitle from "./special-title.vue";

let lyricBox, timer1, timer2;

const { mainTheme } = storeToRefs(staticData());

const {
  getMusicInfo,
  getIsToggleImg,
  getMusicDescription,
  getIsPaused,
  getCurrentLyticIndex,
  getShowLyricBoard,
  getLyricType,
} = storeToRefs(music());

const replaceUrl = ref("");
const fileList = ref([]);

const scrollToMiddle = () => {
  nextTick(() => {
    let current = document.getElementById("currentLyticIndex");

    if (!current) return;

    let h = current
      ? current.offsetTop -
        Math.round(lyricBox.clientHeight / 2) +
        Math.round(current.offsetHeight / 2) +
        -30
      : 0;

    lyricBox &&
      lyricBox.scrollTo({
        top: h,
        behavior: "smooth",
      });
  });
};

const fullScreen = () => {
  const app = document.querySelector(".lyric-mask");
  if (document.fullscreenElement) {
    document.exitFullscreen();
  } else {
    app.requestFullscreen();
  }
};

const goToIndex = (index) => {
  music().setCurrentTimeByClickLyric(index);
};

const toggleLyricType = (type) => {
  music().setLyricType(type);

  if (type == "COMMON") {
    nextTick(() => {
      scrollToMiddle();
    });
  }
};

const calcLyricDuration = () => {
  const nextTime = getMusicInfo.value.lyricTimeList[getCurrentLyticIndex.value + 1];
  const currentTime = getMusicInfo.value.lyricTimeList[getCurrentLyticIndex.value];

  const duration = nextTime - currentTime;

  const fromDuration = duration >= 2000 ? 1.5 : 0;
  const leaveDuration = duration >= 2000 ? 0.3 : 0;

  return {
    duration,
    fromDuration,
    leaveDuration,
  };
};

const animateSpecial = () => {
  const { duration, fromDuration, leaveDuration } = calcLyricDuration();

  if (timer1) {
    clearTimeout(timer1);
    timer1 = null;
  }
  timer1 = setTimeout(() => {
    gsapTransLyric(".special-lyric", fromDuration);
  });
  if (timer2) {
    clearTimeout(timer2);
    timer2 = null;
  }
  timer2 = setTimeout(
    () => {
      gsapTransLyric(".special-lyric", leaveDuration, true);
    },
    duration - leaveDuration * 1000
  );
};

const replaceImage = (file) => {
  replaceUrl.value = URL.createObjectURL(file.raw);
};

watch(
  () => getCurrentLyticIndex.value,
  () => {
    if (getLyricType.value == "COMMON") {
      scrollToMiddle();
    } else {
      nextTick(() => {
        animateSpecial();
      });
    }
  }
);

watch(
  () => getShowLyricBoard.value,
  (newV) => {
    if (newV) {
      nextTick(() => {
        scrollToMiddle();
      });
    }
  }
);

onMounted(() => {
  lyricBox = document.getElementById("lyricBox");
});
</script>

<template>
  <div
    :class="['lyric-mask', getShowLyricBoard ? 'lyric-mask-show' : 'lyric-mask-hide']"
    :style="{
      backgroundImage:
        getLyricType == 'SPECIAL' ? `url(${replaceUrl || getMusicDescription.al.picUrl})` : '',
    }"
  >
    <div v-if="getLyricType == 'SPECIAL'" class="special-mask"></div>
    <div class="toggle-type">
      <span class="type-btn mr-[10px]" v-show="getLyricType == 'SPECIAL'">
        <el-upload
          v-model:file-list="fileList"
          action="#"
          :multiple="false"
          :auto-upload="false"
          :show-file-list="false"
          :on-change="replaceImage"
        >
          ReplaceBg
        </el-upload>
      </span>
      <span class="type-btn" v-show="getLyricType == 'SPECIAL'" @click="toggleLyricType('COMMON')"
        >Normal</span
      >
      <span class="type-btn" v-show="getLyricType == 'COMMON'" @click="toggleLyricType('SPECIAL')"
        >Special</span
      >
    </div>
    <div
      v-show="getLyricType == 'COMMON'"
      class="!w-[100%] !h-[100%] flex justify-between items-center"
    >
      <div class="left">
        <div class="text-4xl font-semibold">
          {{ getMusicDescription?.name }}
        </div>
        <div class="disc-box">
          <svg-icon
            class="disc"
            :name="mainTheme ? 'darkDisc' : 'lightDisc'"
            :width="22"
          ></svg-icon>
          <img
            :class="[
              'music-img',
              'animate__animated',
              'animate__fadeIn',
              getIsToggleImg ? '' : 'disc-rotate',
              getIsPaused ? 'paused' : '',
            ]"
            :src="getMusicDescription?.al?.picUrl"
            @click="fullScreen"
          />
        </div>
      </div>
      <div id="lyricBox" class="right">
        <div class="!p-[10px]">
          <div>
            <div class="text-2xl leading-loose text-center">
              {{ getMusicDescription?.name }}
            </div>
          </div>
          <div
            :id="getCurrentLyticIndex == index ? 'currentLyticIndex' : ''"
            :class="['lyric-word', getCurrentLyticIndex == index ? 'current' : '']"
            v-for="(item, index) in getMusicInfo.lyricList"
            :key="index"
            @click="goToIndex(index)"
          >
            {{ item }}
          </div>
        </div>
      </div>
    </div>
    <div
      v-show="getLyricType == 'SPECIAL'"
      class="special !w-[100%] !h-[100%] flex flex-col justify-center items-center"
    >
      <div class="title" @click="fullScreen">
        <SpecialTitle :title="`《 ${getMusicDescription?.name} 》`" />
      </div>
      <span class="special-lyric text-3xl">
        {{ getMusicInfo.lyricList[getCurrentLyticIndex] }}
      </span>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.lyric-mask {
  position: absolute;
  top: 0px;
  left: 0;
  right: 0;
  bottom: -5px;
  z-index: 2000;
  padding-top: 35px;
  background-color: #fff;
  overflow: hidden;
  display: none;
  background-position: center center;
  background-size: cover;
  background-repeat: no-repeat;

  .special-mask {
    position: absolute;
    top: 0;
    left: 0;
    bottom: 0;
    right: 0;
    background-color: rgba(0, 0, 0, 0.5);
    z-index: 1;
  }

  .toggle-type {
    position: absolute;
    top: 10px;
    right: 60px;
    z-index: 1;
    display: flex;

    .type-btn {
      cursor: pointer;
      color: #62c28a;

      &:hover {
        color: #62c28a;
      }
    }
  }

  .left {
    width: 50%;
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }

  .right {
    padding: 20% 5%;
    width: 50%;
    height: 100%;
    display: flex;
    align-items: flex-start;
    justify-content: center;
    overflow: auto;
    scrollbar-width: none;
    &::-webkit-scrollbar {
      display: none;
    }
  }
}
.special {
  .title {
    color: #fff;
    cursor: pointer;
    z-index: 2;
    height: 200px;
  }

  .special-lyric {
    color: #fff;
    text-align: center;
    z-index: 2;
    font-smooth: never;
    height: 60px;
    line-height: 60px;
    text-shadow: 0px 0px 5px #ffffff;
  }
}

.lyric-mask-show {
  display: block;
  animation: showBoard 0.3s ease-in-out forwards;
}
.lyric-mask-hide {
  animation: hideBoard 0.3s ease-in-out forwards;
}
@keyframes showBoard {
  0% {
    top: 100%;
  }
  100% {
    top: 0;
  }
}

@keyframes hideBoard {
  0% {
    top: 0;
    display: block;
  }
  100% {
    top: 100%;
  }
}

.disc-rotate {
  animation: rotate360 36s infinite linear;
}

.paused {
  animation-play-state: paused;
}

.disc-box {
  position: relative;
  width: 30rem;
  height: 30rem;
  display: grid;
  place-items: center;

  .disc {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
  }
  .music-img {
    width: 15rem;
    height: 15rem;
    border-radius: 7.5rem;
    object-fit: cover;
    cursor: pointer;
  }
}

@keyframes rotate360 {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}
.lyric-word {
  text-align: center;
  line-height: 2;
  cursor: pointer;
  font-size: 1.2rem;
  transition: all 0.3s;
  opacity: 0.5;
}
.current {
  font-size: 1.5rem;
  font-weight: bold;
  opacity: 1;
}

// mobile
@media screen and (max-width: 768px) {
  .lyric-mask {
    .right {
      width: 100%;
    }
    .left {
      display: none;
    }
  }

  .toggle-type {
    top: 60px !important;
  }
}

:fullscreen {
  /* 代码 */
  .toggle-type {
    display: none !important;
  }
}
</style>
