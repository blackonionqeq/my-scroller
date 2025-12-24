<template>
  <div class="my-scroller-wrapper">
    <!-- <MyScroller class="my-scroller" /> -->
    <PullToRefreshWrapper class="my-scroller">
      <div v-for="i in 30" :key="i" style="padding: 16px; border-bottom: 1px solid #ccc;">
        Item {{ i }}
      </div>
    </PullToRefreshWrapper>
  </div>
</template>

<script setup>
// import MyScroller from './MyScroller.vue';
import PullToRefreshWrapper from './PullToRefreshWrapper.vue'
</script>

<style>
html[theme="dark"]{
  background-color: #333;
  color: #eee;
}
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