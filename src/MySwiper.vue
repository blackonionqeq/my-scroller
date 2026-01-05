<template>
  <div class="my-swiper-wrapper" ref="wrapper" @touchstart="handleTouchStart" @touchmove="handleTouchMove" @touchend="handleTouchEnd" :style="{transform: `translateX(${currentTransformX}px)`, willChange: isHorizontalDragging ? 'transform' : '' }">
    <slot>
      <!-- MySwiper content goes here -->
      <MySwiperItem v-for="n in 3" :key="n" :index="n - 1">
        <div class="item-content">Item {{ n }}</div>
      </MySwiperItem>
    </slot>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, provide, defineAsyncComponent, computed, watch } from 'vue'

const MySwiperItem = defineAsyncComponent(() => import('./MySwiperItem.vue'))
const wrapper = ref<HTMLElement | null>(null)

const props = defineProps({
  current: {
    type: Number,
    default: 0
  },
  threshold: {
    type: Number,
    default: 100
  },
  duration: {
    type: Number,
    default: 300
  }
})

const emit = defineEmits<{
  (e: 'update:current', value: number): void
}>()

const selfCurrent = ref(props.current)
watch(
  () => props.current,
  (newVal) => {
    selfCurrent.value = newVal
  }
)
provide('swiper-current', selfCurrent)

const itemWidth = ref(0)
const itemWidthWithUnit = computed(() => `${itemWidth.value}px`)
provide('swiper-item-width', itemWidthWithUnit)
const isInitialized = ref(false)
onMounted(() => {
  if (wrapper.value) {
    itemWidth.value = wrapper.value.clientWidth
    isInitialized.value = true
    console.log('Wrapper Width as Item Width:', itemWidth.value)
  }
})

const startX = ref(0)
const startY = ref(0)
const currentX = ref(0)
const currentY = ref(0)
let isDragging = false
const isHorizontalDragging = ref(false)
const currentTransformX = ref(0)
const tmpTransformX = ref(0)
const handleTouchStart = (event: TouchEvent) => {
  startX.value = event.touches[0].clientX
  startY.value = event.touches[0].clientY
  tmpTransformX.value = currentTransformX.value
  // isDragging = true
  // Handle touch start event
}
const handleTouchMove = (event: TouchEvent) => {
  // first time to determine direction
  if (!isDragging) {
    // isHorizontalDragging = true
    // judge direction here
    const deltaX = Math.abs(event.touches[0].clientX - startX.value)
    const deltaY = Math.abs(event.touches[0].clientY - startY.value)
    if (deltaX > deltaY) {
      isHorizontalDragging.value = true
      isDragging = true
      event.preventDefault()
    } else {
      // Not a horizontal drag, ignore
      isDragging = true
      return
    }
  } else if (isHorizontalDragging) {
    event.preventDefault()
    currentX.value = event.touches[0].clientX
    currentY.value = event.touches[0].clientY

    // Handle touch move event
    const deltaX = currentX.value - startX.value
    currentTransformX.value = tmpTransformX.value + deltaX
    // currentTransformX.value = -props.current * itemWidth.value + deltaX
  }
}
const handleTouchEnd = (event: TouchEvent) => {
  // Handle touch end event
  isDragging = false
  isHorizontalDragging.value = false
  const deltaX = currentX.value - startX.value
  console.log('Delta X:', deltaX)
  if (Math.abs(deltaX) > props.threshold) {
    if (currentX.value < startX.value) {
      console.log('Swiped Left')
      emit('update:current', props.current + 1)
      // start transition to next item
    } else {
      console.log('Swiped Right')
      emit('update:current', props.current - 1)
      // start transition to previous item
    }
  } else {
    console.log('Swipe too short, stay on current item')
    // reset position to current item
    // currentTransformX.value = 0
    animateToCurrentItem()
  }
}

watch(
  () => selfCurrent.value,
  (newVal) => {
    if (isInitialized.value) {
      console.log('Animating to Current Item Index:', newVal)
      // currentTransformX.value = -newVal * itemWidth.value
      animateToCurrentItem()
    }
  },
  { immediate: true }
)
function animateToCurrentItem() {
  // currentTransformX.value = -selfCurrent.value * itemWidth.value
  // use request animationFrame for smooth animation
  const targetX = -selfCurrent.value * itemWidth.value
  const initialX = currentTransformX.value
  console.log('currentTransformX before animation:', currentTransformX.value)
  console.log('Animating to targetX:', targetX)
  const deltaX = targetX - initialX
  const startTime = performance.now()
  function animate(time: number) {
    const elapsed = time - startTime
    const progress = Math.min(elapsed / props.duration, 1)
    currentTransformX.value = initialX + deltaX * progress
    if (progress < 1) {
      requestAnimationFrame(animate)
    }
  }
  requestAnimationFrame(animate)
}

</script>

<style scoped>
.my-swiper-wrapper {
  display: flex;
  flex-wrap: nowrap;
  /* flex-shrink: 0;
  flex-grow: 0; */
  flex-basis: stretch;
  /* flex-direction: row; */
}
</style>