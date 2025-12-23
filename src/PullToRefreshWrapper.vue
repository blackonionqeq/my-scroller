<template>
  <div class="p2r-wrapper">
    <div class="p2r-ptr" ref="ptr" :style="{ transform: `translateY(${prefixHeight}px)` }">
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
    <div class="p2r-content" ref="content">
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

const status = ref<'pull' | 'release' | 'refreshing' | null>(null)

const prefixHeight = ref(0)
const ptr = ref<HTMLElement | null>(null)
onMounted(() => {
  if (ptr.value) {
    prefixHeight.value = -ptr.value.offsetHeight - 12
  }
})
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