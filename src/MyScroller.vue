<template>
  <div class="wrapper" ref="wrapper" @scroll.prevent="debouncedScrollHandler">
    <div class="header-wrapper" ref="headerWrapper">
      <slot name="header">
        <!-- <div v-if="showTmp"> -->
        <div>aaa</div>
        <div>bbb</div>
        <div>ccc</div>
        <div>ccc</div>
        <div>ccc</div>
        <div>ccc</div>
        <!-- </div> -->
        <!-- <div style="height: 100px" v-if="showTmp">asdf</div> -->
      </slot>
    </div>
    <div class="fixed-btn">
      <button @click="addItems">add</button>
      <button @click="scrollTo(100)">scroll to 100</button>
      <button @click="scrollToIndex(20)">scroll to index 20</button>
    </div>
    <div class="phatom-scroller" :style="{ height: totalHeight + headerHeight + 'px' }" ></div>
    <div class="list-wrapper" :style="{ transform: `translateY(${listWrapperTransformY}px)` }">
      <SimpleItem
        v-for="(item, index) in showDatas" :key="item" class="list-item"
        :itemData="item">
        <template #default="{ data }">
          {{ data }}
        </template>
      </SimpleItem>
      <div v-if="isLoading" class="loading-wrapper">
        <div class="loading"></div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { nextTick, ref, onMounted, onUnmounted } from 'vue'
import { debounce } from './utils/debounce'
// import ScrollerItem from './ScrollerItem.vue'
import SimpleItem from './SimpleItem.vue'
const props = defineProps({
  itemHeight: {
    type: Number,
    default: 60
  },
  bufferLines: {
    type: Number,
    default: 1
  },
  lineItemCounts: {
    type: Number,
    default: 2
  },
  bottomThreshold: {
    type: Number,
    default: 100
  },
})

const emits = defineEmits<{
  (e: 'load-more'): void
}>()

const totalHeight = ref(0)
const bufferViewLines = ref(0)
const viewCount = Math.ceil(window.innerHeight / props.itemHeight)
// console.log('viewCount', viewCount)
if (props.itemHeight) {
  bufferViewLines.value = viewCount + props.bufferLines
  // console.log('bufferViewLines', bufferViewLines.value)
}
const wrapper = ref<HTMLElement | null>(null)


const datas = ref(Array.from({ length: 100 }).map((_, i) => `Item ${i + 1}`))

const showDatas = ref<string[]>([])
if (datas.value.length) {
  showDatas.value = datas.value.slice(0, bufferViewLines.value * props.lineItemCounts)

  totalHeight.value = Math.ceil(datas.value.length / props.lineItemCounts * props.itemHeight)
}
const listWrapperTransformY = ref(0)


const isLoading = ref(false)

function addItems() {
  isLoading.value = true
  setTimeout(() => {
    const currentLength = datas.value.length
    const newItems = Array.from({ length: 10 }).map((_, i) => `Item ${currentLength + i + 1}`)
    datas.value.push(...newItems)
    isLoading.value = false
    totalHeight.value = Math.ceil(datas.value.length / props.lineItemCounts * props.itemHeight)
    // rerender
    const scrollTop = listWrapperTransformY.value
    renderByScrollTop(scrollTop)
    // console.log('new totalHeight', totalHeight.value)
    showDatas
  }, 1000)
}

function handleScroll(event: Event) {
  // console.log('scroll event fired')
  const target = event.target as HTMLElement
  const scrollTop = target.scrollTop

  renderByScrollTop(scrollTop)

  // 判断是否接近底部，触发加载更多
  nextTick(() => {
    if (judgeShouldLoadMore()) {
      // addItems()
      isLoading.value = true
      emits('load-more')
    }
  })
}

function renderByScrollTop(scrollTop: number) {
  if (scrollTop < headerHeight.value) {
    showDatas.value = datas.value.slice(0, bufferViewLines.value * props.lineItemCounts)
    listWrapperTransformY.value = 0
    return
  }
  const _scrollTop = Math.max(0, scrollTop - headerHeight.value)
  let startLine = Math.floor(_scrollTop / props.itemHeight)
  if (startLine <= 0) {
    showDatas.value = datas.value.slice(0, bufferViewLines.value * props.lineItemCounts)
    listWrapperTransformY.value = 0
    return
  }
  const deltaLines = startLine - props.bufferLines
  startLine = Math.max(0, deltaLines)
  const startIndex = startLine * props.lineItemCounts
  if (deltaLines > 0) {
    showDatas.value = datas.value.slice(startIndex, startIndex + (bufferViewLines.value + props.bufferLines) * props.lineItemCounts)
    listWrapperTransformY.value = startLine * props.itemHeight
  } else {
    showDatas.value = datas.value.slice(0, (bufferViewLines.value + props.bufferLines) * props.lineItemCounts)
    listWrapperTransformY.value = 0
  }
}

const debouncedScrollHandler = debounce(handleScroll, 10)

function judgeShouldLoadMore() {
  if (isLoading.value) return false
  const scrollTop = wrapper.value?.scrollTop || 0
  const scrollHeight = wrapper.value?.scrollHeight || 0
  const clientHeight = wrapper.value?.clientHeight || 0
  if (scrollHeight - (scrollTop + clientHeight) < props.bottomThreshold) {
    return true
  }
  return false
}

function scrollTo(scrollTop: number) {
  if (wrapper.value) {
    wrapper.value.scrollTop = scrollTop
    renderByScrollTop(scrollTop)
  }
}
function scrollToIndex(index: number) {
  const scrollTop = Math.floor(Math.max(0, index - props.lineItemCounts) / props.lineItemCounts) * props.itemHeight + headerHeight.value
  scrollTo(Math.max(0, scrollTop))
}

function completeLoading() {
  isLoading.value = false
  // rerender
  const scrollTop = wrapper.value?.scrollTop || 0
  renderByScrollTop(scrollTop)
}

const headerWrapper = ref<HTMLElement | null>(null)
const headerHeight = ref(0)
const headerObserver = new ResizeObserver((enties) => {
  for (const entry of enties) {
    const newHeight = entry.contentRect.height
    if (newHeight !== headerHeight.value) {
      headerHeight.value = newHeight
      renderByScrollTop(wrapper.value?.scrollTop || 0)
    }
  }
})
// const showTmp = ref(false)
onMounted(() => {
  if (headerWrapper.value && wrapper.value) {
    headerObserver.observe(headerWrapper.value)
  }
  // setTimeout(() => {
  //   showTmp.value = true
  // }, 1e3)
})
onUnmounted(() => {
  headerObserver.disconnect()
})

defineExpose({
  scrollTo,
  scrollToIndex,
  completeLoading,
})
</script>

<style scoped>
.wrapper {
  position: relative;
  height: 100%;
  overflow-y: auto;
  overflow-x: hidden;
}
.phatom-scroller {
  width: 100%;
  position: absolute;
  inset: 0;
  /* pointer-events: none; */
  /* opacity: 0; */
  z-index: 1;
}
.list-item {
  padding: 20px;
  border-bottom: 1px solid gray;
  /* width: 100%; */
  
  box-sizing: border-box;
  height: 60px;
  width: 50%;
}
.list-wrapper {
  position: absolute;
  /* inset: 0; */
  left: 0;
  width: 100%;
  box-sizing: border-box;
  overflow-y: auto;
  display: flex;
  flex-wrap: wrap;
  /* flex-direction: column; */
  /* will-change: transform; */
  /* flex-direction: column-reverse;  */
  /* overflow-anchor: ; */
}
.fixed-btn {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 20;

  display: flex;
  flex-direction: column;
}
.loading-wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
}
.loading {
  border: 4px solid #f3f3f3;
  border-top: 4px solid #3498db;
  border-radius: 50%;
  width: 24px;
  height: 24px;
  animation: spin 2s linear infinite;
}
@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
</style>