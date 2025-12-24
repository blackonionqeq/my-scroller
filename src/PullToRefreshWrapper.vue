<template>
  <div class="p2r-wrapper" @touchstart="handleTouchStart" @touchmove="handleTouchMove" @touchend="handleTouchEnd">
    <div class="p2r-ptr" ref="ptr" :style="{ transform: `translateY(${prefixHeight + pullDistance}px)` }">
      <slot name="ptr">
        <div v-if="status !== 'refreshing'"
          class="p2r-ptr-icon" :class="{rotate180: status === 'release'}">  
          <img v-if="ptrIcon" :src="ptrIcon" alt="ptr icon" />
          <span v-else>↓</span>
        </div>
        <div class="p2r-ptr-text" v-if="status === 'pull'">{{ ptrText }}</div>
        <div class="p2r-ptr-text" v-else-if="status === 'release'">{{ refreshText }}</div>
        <div class="p2r-ptr-text" v-else-if="status === 'refreshing'">{{ refreshingText }}</div>
      </slot>
    </div>
    <div class="p2r-content" ref="content" :style="{ transform: `translateY(${pullDistance}px)` }">
      <slot>
        Content goes here
      </slot>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue'

const props = defineProps({
  ptrText: {
    type: String,
    default: "Pull to refresh"
  },
  ptrIcon: {
    type: String,
    default: ""
  },
  refreshText: {
    type: String,
    default: "Leave to refresh"
  },
  refreshingText: {
    type: String,
    default: "Refreshing..."
  },
  refreshThreshold: {
    type: Number,
    default: 60
  },
})

const emit = defineEmits<{
  (e: 'refresh'): void
}>()

const status = ref<'pull' | 'release' | 'refreshing' | null>(null)

const prefixHeight = ref(0)
const ptr = ref<HTMLElement | null>(null)
onMounted(() => {
  if (ptr.value) {
    prefixHeight.value = -ptr.value.offsetHeight - 12
  }
})
const content = ref<HTMLElement | null>(null)
const pullDistance = ref(0)

let startY = 0
let isTouching = false
function handleTouchStart(event: TouchEvent) {
  // console.log('handleTouchStart', status.value)
  if (status.value === 'refreshing') return
  // if ((event.target as HTMLElement).closest('.p2r-content') && (event.target as HTMLElement).closest('.p2r-content')!.scrollTop === 0) {
  if (content.value && content.value.scrollTop === 0) {
    isTouching = true
    startY = event.touches[0].clientY
    status.value = 'pull'
    // console.log('touch start')
  }
}
function handleTouchMove(event: TouchEvent) {
  event.preventDefault()
  if (!isTouching || status.value === 'refreshing') return
  const currentY = event.touches[0].clientY
  const diffY = currentY - startY
  // console.log('handleTouchMove', diffY)
  if (diffY > 0) {
    // event.preventDefault()
    pullDistance.value = diffY
    // prefixHeight.value = Math.min(diffY / 2 + (-ptr.value!.offsetHeight - 12), 150)
    if (diffY >= props.refreshThreshold) {
      status.value = 'release'
      // console.log('release')
    } else {
      status.value = 'pull'
      // console.log('pull')
    }
  }
}

function handleTouchEnd() {
  if (!isTouching) return
  isTouching = false
  if (status.value === 'release') {
    status.value = 'refreshing'
    // console.log('refreshing')
    pullDistance.value = props.refreshThreshold
    // Simulate refresh action
    setTimeout(() => {
      pullDistance.value = 0
      status.value = null
    }, 2000)
  } else {
    pullDistance.value = 0
    status.value = null
  }
}
</script>

<style scoped>
.p2r-wrapper {
  overflow: hidden;
  position: relative;
  height: 100%;
  width: 100%;
}
.rotate180 {
  transform: rotate(180deg);
  transition: transform 0.3s;
}
.p2r-ptr {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
</style>