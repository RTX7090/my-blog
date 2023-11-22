<!--友链列表  -->
<script setup>
import { ref, reactive, onMounted, onBeforeUnmount } from "vue";
import { getFriendLinks } from "@/api/links";

import SkeletonItem from "@/components/SkeletonItem/skeleton-item.vue";

const active = ref(0);
const activeType = ref("");
const loading = ref(false);
const params = reactive({
  current: 1,
  size: 6,
  status: 2,
});

const linksList = ref([]);
const total = ref(0);
let observe;
let box;

// 鼠标进入触发
const mouseEnterItem = (type, index) => {
  activeType.value = type;
  active.value = index;
};
// 鼠标离开触发
const mouseLeaveItem = () => {
  activeType.value = "";
  active.value = 0;
};
const goToSite = (url) => {
  window.open(url);
};

const observeBox = () => {
  // 获取要监听的元素
  box = document.querySelector(".observer");
  observe = new IntersectionObserver(
    (entries) => {
      entries.forEach(async (e) => {
        if (e.isIntersecting && e.intersectionRatio > 0) {
          if (total.value > linksList.value.length) {
            params.current++;
            pageGetLinksList(e);
          }
        }
      });
    },
    { threshold: [1] }
  );
  observe.observe(box);
};

const returnUrl = (url) => {
  const end = url.substring(url.length - 1);
  return end != "/" ? url + "/favicon.ico" : url + "favicon.ico";
};

const pageGetLinksList = async () => {
  if (params.current == 1) {
    loading.value = true;
  }
  const res = await getFriendLinks(params);
  if (res && res.code == 0) {
    linksList.value =
      params.current == 1 ? res.result.list : linksList.value.concat(res.result.list);
    total.value = res.result.total - 0;
    loading.value = false;
  }
};

onMounted(async () => {
  await pageGetLinksList();
  if (linksList.value.length < total.value) {
    observeBox();
  }
});

onBeforeUnmount(() => {
  observe && observe.unobserve(box);
  observe = null;
});
</script>

<template>
  <PageHeader :loading="loading" />
  <div class="center_box">
    <div class="w-[100%] !m-[10px] flex justify-center">
      <span class="apply-button" @click="$router.push('/link/apply')">申请友链</span>
    </div>
    <el-skeleton :loading="loading" style="height: 100%" animated>
      <template #template>
        <div class="flex justify-start w-[100%] !mt-[10px]" v-for="i in 2" :key="i">
          <div class="flex justify-between w-[100%] !p-[10px]">
            <div class="w-[47%] flex">
              <SkeletonItem variant="circle" width="60px" height="60px" />
              <div class="w-[60%] flex flex-col justify-start !ml-[10px]">
                <SkeletonItem variant="text" width="20%" height="20px" />
                <SkeletonItem variant="text" width="90%" height="30px" top="5px" />
              </div>
            </div>
            <div class="w-[47%] flex">
              <SkeletonItem variant="circle" width="60px" height="60px" />
              <div class="w-[60%] flex flex-col justify-start !ml-[10px]">
                <SkeletonItem variant="text" width="20%" height="20px" />
                <SkeletonItem variant="text" width="90%" height="30px" top="5px" />
              </div>
            </div>
          </div>
        </div>
      </template>
      <el-row class="site">
        <el-col :xs="24" :sm="8" v-for="(item, index) in linksList" :key="index">
          <el-card class="card-hover">
            <div
              :style="{
                backgroundImage: `url(${item.site_avatar})`,
              }"
              :class="[
                'site-item',
                'animate__animated',
                'animate__fadeIn',
                activeType == 'site' && active == index ? 'site-mask' : '',
              ]"
              @mouseenter="mouseEnterItem('site', index)"
              @mouseleave="mouseLeaveItem"
            >
              <div class="left">
                <el-avatar
                  :class="[activeType == 'site' && active == index ? 'avatar-hover' : 'avatar']"
                  fit="cover"
                  :size="80"
                  :src="item.site_avatar || returnUrl(item.url)"
                >
                  <span class="avatar-font">{{ item.site_name }}</span></el-avatar
                >
              </div>
              <div class="right">
                <div class="w-[100%] flex justify-between items-center">
                  <span :title="item.site_name" class="name">{{ item.site_name }}</span>
                  <span
                    class="iconfont icon-link cursor-pointer"
                    @click="goToSite(item.url)"
                  ></span>
                </div>

                <span
                  :style="{ height: activeType == 'site' && active == index ? '4.6rem' : '0' }"
                  :title="item.site_desc"
                  class="desc"
                >
                  {{ item.site_desc }}</span
                >
              </div>
            </div>
          </el-card>
        </el-col>
      </el-row>
      <div class="observer">{{ total <= linksList.length ? "没有更多了" : "加载更多" }}</div>
    </el-skeleton>
  </div>
</template>

<style lang="scss" scoped>
.site {
  &-item {
    display: flex;
    justify-content: space-between;
    padding: 20px 10px;
    cursor: pointer;
    position: relative;
    min-height: 11rem;
    background-position: center;
    background-size: cover;
    background-color: rgba(255, 255, 255, 0.5);
    .left {
      position: absolute;
      left: 50%;
      top: 20px;
      transform: translateX(-50%);
      transition: all 0.8s;

      .avatar-hover {
        animation: avatarHover 0.8s forwards;
      }

      .avatar {
        animation: avatar 0.8s forwards;
      }
    }

    .right {
      width: 100%;
      display: flex;
      flex-direction: column;
      justify-content: flex-end;
      align-items: flex-start;
      z-index: 2;

      .name {
        font-size: 1.8rem;
        font-weight: bold;
        line-height: 1.7;
        color: var(--font-color);
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
        text-decoration: none;

        &:hover {
          color: var(--primary);
        }
      }

      .icon-link {
        font-size: 18px;
        font-weight: 600;
        color: #9d269d;
        &:hover {
          color: var(--primary);
        }
      }

      .desc {
        transition: all 0.8s;
        display: -webkit-box;
        width: 100%;
        font-weight: bold;
        height: 4.6rem;
        color: var(--font-color);
        line-height: 1.2;
        font-size: 1.3rem;
        text-overflow: -o-ellipsis-lastline;
        overflow: hidden;
        text-overflow: ellipsis;
        -webkit-line-clamp: 3;
        line-clamp: 3;
        -webkit-box-orient: vertical;
      }
    }
  }
}
.site-mask::before {
  content: "";
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(255, 255, 255, 0.5);
}

@keyframes avatarHover {
  0% {
    transform: translateY(0);
  }

  100% {
    transform: translateY(-100px);
  }
}

@keyframes avatar {
  0% {
    transform: translateY(-100px);
  }

  100% {
    transform: translateY(0);
  }
}

.observer {
  text-align: center;
  font-size: 1rem;
  color: var(--font-color);
  margin-top: 3px;
  letter-spacing: 1px;
}

:deep(.el-avatar) {
  color: var(--font-color);
  background: linear-gradient(
    90deg,
    #dfd2d2 1%,
    #ead2ea 10.2%,
    #e4d5ee 19.6%,
    #d1def3 36.8%,
    #b5dee5 62.2%,
    #cfebf3 88.9%,
    #dde7ea 99%
  ) !important;
}
</style>
