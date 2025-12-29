<template>
  <div class="my-swiper-item" ref="item">
    <slot>
      MySwiperItem content goes here
    </slot>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, inject, type Ref } from 'vue'

const swiperCurrent = inject<Ref<number>>('swiper-current')
const swiperItemWidth = inject<string>('swiper-item-width', '100%')

const props = defineProps({
  index: {
    type: Number,
    default: 0
  }
})

// onMounted(() => {
  // You can access swiperCurrent and props.index here
  console.log('Swiper Current:', swiperCurrent.value)
  console.log('Item Index:', props.index)
// })
</script>

<style scoped>
.my-swiper-item {
  width: v-bind("swiperItemWidth");
  min-width: v-bind("swiperItemWidth");
  /* width: 100%; */
  height: 100%;
  /* box-sizing: border-box; */
}
</style>