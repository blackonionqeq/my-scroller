<template>
  <div v-if="status === 'idle' || status === 'loading-next-page' || status === 'loading-next-page-error'" class="list-container">
    <template v-if="datas.length === 0">
      <slot name="empty">
        <div>No Data</div>
      </slot>
    </template>
    <slot v-else :datas>
      <div>Default slot content</div>
    </slot>
    <slot name="bottom">
      <div v-if="status === 'loading-next-page'">
        Loading more...
      </div>
      <div v-else-if="status === 'loading-next-page-error'" @click="handleRetryNextPage">
        Error loading more data. Click to retry.
      </div>
    </slot>
  </div>

  <slot v-else-if="status === 'loading'">
    <div>Loading...</div>
  </slot>
  <slot v-else-if="status === 'error'">
    <div>Error loading data.</div>
    <div v-if="showRetry" @click="handleRetry">
      Click here to retry.
    </div>
  </slot>
</template>

<script setup lang="ts" generic="T extends object">
import { ref } from 'vue'

const props = defineProps({
  datas: {
    type: Array as () => T[],
    default: () => []
  },
  showRetry: {
    type: Boolean,
    default: true
  },
  showRetryNextPage: {
    type: Boolean,
    default: true
  },
})

const status = ref<'idle' | 'loading' | 'error' | 'loading-next-page' | 'loading-next-page-error'>('idle')
const getStatus = () => {
  return status.value
}
// const setStatus = (newStatus: 'idle' | 'loading' | 'error') => {
//   status.value = newStatus
// }
const completeLoading = (isSuccess = true) => {
  status.value = isSuccess ? 'idle' : 'error'
}
const startLoading = () => {
  status.value = 'loading'
  return completeLoading
}
const completeLoadingNextPage = (isSuccess = true) => {
  status.value = isSuccess ? 'idle' : 'loading-next-page-error'
}
const startLoadingNextPage = () => {
  status.value = 'loading-next-page'
  return completeLoadingNextPage
}

const emits = defineEmits<{
  (e: 'retry'): void
  (e: 'retry-next-page'): void
}>()
const handleRetry = () => {
  if (props.showRetry) {
    emits('retry')
  }
}
const handleRetryNextPage = () => {
  if (props.showRetryNextPage) {
    emits('retry-next-page')
  }
}

defineExpose({
  getStatus,
  startLoading,
  startLoadingNextPage,
})
</script>