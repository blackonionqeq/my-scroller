<template>
  <div class="my-scroller-wrapper" :style="{direction: isLTR ? 'ltr' : 'rtl'}">
    <!-- <MyScroller class="my-scroller" /> -->
    <!-- <PullToRefreshWrapper class="my-scroller">
      <div v-for="i in 30" :key="i" style="padding: 16px; border-bottom: 1px solid #ccc;">
        Item {{ i }}
      </div>
    </PullToRefreshWrapper> -->
    <MySwiperTest class="my-scroller" :is-l-t-r="isLTR" />
  </div>
</template>

<script setup>
// import MyScroller from './MyScroller.vue';
// import PullToRefreshWrapper from './PullToRefreshWrapper.vue'
import MySwiperTest from './MySwiperTest.vue';
import { ref } from 'vue';

const isLTR = ref(false);

setTimeout(() => {
  isLTR.value = true;
}, 2000);
</script>

<style>
html[theme="dark"]{
  background-color: #333;
  color: #eee;
}
/* html {
  direction: rtl;
} */
.my-scroller-wrapper {
  height: 100vh;
  overflow: hidden;
  /* position: relative; */
}
.my-scroller {
  height: fit-content;
  max-height: 100vh;
}
</style>