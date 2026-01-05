<template>
  <MySwiperItem v-bind="$attrs" :index>
    <div class="item-content" v-show="shouldRender">
      <h3>Swiper Item</h3>
      <p>This is a swiper item content.</p>
      <div>index is {{ props.index }}</div>
    </div>
  </MySwiperItem>
</template>

<script setup lang="ts">
import { inject, onMounted, ref, watch, type Ref } from 'vue';
import MySwiperItem from '../MySwiperItem.vue';

const props = defineProps({
  index: {
    type: Number,
    default: 0
  }
})

const shouldRender = ref(false)
const current = inject<Ref<number>>('swiper-current')
watch(
  () => current?.value,
  (newVal) => {
    console.log('Current Swiper Index:', newVal)
    const abs = Math.abs(newVal - props.index)
    shouldRender.value = abs <= 1
  },
  { immediate: true }
)

console.log('Wrapper Item Index:', props.index)
</script>