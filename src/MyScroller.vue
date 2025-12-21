<template>
  <div class="wrapper" @scroll.prevent="debouncedScrollHandler">
    <button class="fixed-btn" @click="addItems">add</button>
    <div class="phatom-scroller" :style="{ height: totalHeight + 'px' }" ></div>
    <div class="list-wrapper" :style="{ transform: `translateY(${listWrapperTransformY}px)` }">
      <SimpleItem
        v-for="(item, index) in showDatas" :key="item" class="list-item"
        :itemData="item">
        <template #default="{ data }">
          {{ data }}
        </template>
      </SimpleItem>
      <!-- <div v-if="isLoading" class="loading-wrapper">
        <div class="loading"></div>
      </div> -->
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { debounce } from './utils/debounce';
// import ScrollerItem from './ScrollerItem.vue';
import SimpleItem from './SimpleItem.vue';
const props = defineProps({
  itemHeight: {
    type: Number,
    default: 60
  },
  bufferLines: {
    type: Number,
    default: 2
  },
  lineItemCounts: {
    type: Number,
    default: 2
  },
});

const totalHeight = ref(0)
const bufferViewLines = ref(0)
const viewCount = Math.ceil(window.innerHeight / props.itemHeight);
// console.log('viewCount', viewCount);
if (props.itemHeight) {
  bufferViewLines.value = viewCount + props.bufferLines;
  // console.log('bufferViewLines', bufferViewLines.value);
}


const datas = ref(Array.from({ length: 100 }).map((_, i) => `Item ${i + 1}`));

const showDatas = ref<string[]>([])
if (datas.value.length) {
  showDatas.value = datas.value.slice(0, bufferViewLines.value * props.lineItemCounts);

  totalHeight.value = Math.ceil(datas.value.length / props.lineItemCounts * props.itemHeight)
}
const listWrapperTransformY = ref(0);


// const isLoading = ref(false);

function addItems() {
  // isLoading.value = true;
  // setTimeout(() => {
    const currentLength = datas.value.length;
    const newItems = Array.from({ length: 10 }).map((_, i) => `Item ${currentLength + i + 1}`);
    datas.value.push(...newItems);
    // isLoading.value = false;
    totalHeight.value = Math.ceil(datas.value.length / props.lineItemCounts * props.itemHeight)
    console.log('new totalHeight', totalHeight.value);
  // }, 1000);
}

function handleScroll(event: Event) {
  // console.log('scroll event fired');
  const target = event.target as HTMLElement;
  const scrollTop = target.scrollTop;

  let startLine = Math.floor(scrollTop / props.itemHeight);
  if (startLine === 0) {
    showDatas.value = datas.value.slice(0, bufferViewLines.value * props.lineItemCounts);
    listWrapperTransformY.value = 0
    return;
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

const debouncedScrollHandler = debounce(handleScroll, 10);
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