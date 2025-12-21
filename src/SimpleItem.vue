<template>
  <div class="item">
    <slot :data="itemData">{{ itemData[keyId] }}</slot>
  </div>
</template>

<script setup lang="ts" generic="T extends object | string | number">

defineProps({
  itemData: {
    type: [Object, String, Number],
    required: true
  },
  keyId: {
    type: [String, Number],
    default: 'id'
  },
})

</script>