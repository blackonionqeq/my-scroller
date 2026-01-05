<template>
  <div class="swiper-test">
    <MySwiper v-model:current="current">
      <MySwiperItemWrapper v-for="n in 5" :key="n-1" :index="n-1" />
    </MySwiper>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from 'vue';
import MySwiper from './MySwiper.vue';
import MySwiperItemWrapper from './test/MySwiperItemWrapper.vue';
// import MySwiperItem from './MySwiperItem.vue';

const current = defineModel({ type: Number, default: 0 })
watch(
  () => current.value,
  (newVal) => {
    console.log('Current Swiper Index in Test:', newVal)
  },
  { immediate: true }
)
</script>

<style scoped>
.swiper-test {
  width: 300px;
  height: 200px;
  border: 1px solid #ccc;
  overflow: hidden;
}
</style>