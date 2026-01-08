<template>
  <div class="p2r-wrapper" @touchstart.passive="handleTouchStart" @touchmove="handleTouchMove" @touchend="handleTouchEnd">
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
    <div class="p2r-content" ref="content" :style="{ transform: `translateY(${pullDistance}px)`, overflowY: disableScrollBehavoir ? 'hidden' : 'auto' }">
      <slot>
        Content goes here
      </slot>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'

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
  maxDelta: {
    type: Number,
    default: 100
  },
  prefixDeltaHeight: {
    type: Number,
    default: 20
  }
})

const emit = defineEmits<{
  (e: 'refresh'): void
}>()

const status = ref<'pull' | 'release' | 'refreshing' | null>(null)

const prefixHeight = ref(0)
const ptr = ref<HTMLElement | null>(null)
onMounted(() => {
  if (ptr.value) {
    prefixHeight.value = -ptr.value.offsetHeight - props.prefixDeltaHeight
  }
})
const content = ref<HTMLElement | null>(null)
const pullDistance = ref(0)
const disableScrollBehavoir = computed(() => {
  return pullDistance.value > 0
})

let startY = 0
let tmpY = 0
let isTouching = false
function handleTouchStart(event: TouchEvent) {
  if (status.value === 'refreshing') return
  if (content.value) {
    isTouching = true
    if (content.value.scrollTop > 0) {
      return
    }
    startY = event.touches[0].clientY
    status.value = 'pull'
  }
}
function handleTouchMove(event: TouchEvent) {
  if (!isTouching || status.value === 'refreshing') return
  const currentY = event.touches[0].clientY
  let diffY = currentY - startY
  if (status.value === null) {
    if (content.value?.scrollTop > 0) {
      return
    } else if (content.value?.scrollTop === 0) {
      // Trigger touch start to reset startY
      handleTouchStart(event)
      return
    }
  }
  if (diffY >= 0) {
    pullDistance.value = Math.min(diffY, props.maxDelta)
    // console.log('pullDistance', pullDistance.value)
    if (pullDistance.value === 0) {
      tmpY = currentY
    }
    if (diffY >= props.refreshThreshold) {
      status.value = 'release'
    } else {
      status.value = 'pull'
    }
  } else if (tmpY !== 0) {
    const deltaY = currentY - tmpY
    content.value!.scrollTop = -deltaY
  } else if (diffY < 0) {
    tmpY = currentY
  }
}

function handleTouchEnd() {
  if (!isTouching) return
  isTouching = false
  tmpY = 0
  // downScrollTop = 0
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
}
.p2r-ptr {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  /* flex-direction: column; */
  display: flex;
  align-items: center;
  justify-content: center;
}
.p2r-ptr-icon {
  transition: transform 0.3s;
}
.p2r-content {
  position: static;
  width: 100%;
  height: 100%;
  overflow: auto;
}
</style>