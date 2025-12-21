<template>
  <div class="my-scroller-wrapper">
    <MyScroller class="my-scroller" />
  </div>
</template>

<script setup>
import MyScroller from './MyScroller.vue';
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
  max-height: 100%;
}
</style>