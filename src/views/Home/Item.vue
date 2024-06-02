<script setup lang="ts">
import { useDraggable } from '@vueuse/core'
import { ref } from 'vue'

import type { ItemProps } from '.'

const props = defineProps<{
  modelValue: ItemProps
}>()

const emits = defineEmits(['move', 'update:modelValue'])

const item = useVModel(props, 'modelValue', emits)

const el = ref<HTMLElement | null>(null)

const { style, x, y, isDragging } = useDraggable(el, {
  initialValue: { x: item.value.x, y: item.value.y },
  onStart: () => {
    if (item.value.value === 2)
      return false
  },
  onMove: (position) => {
    item.value.x = position.x
    item.value.y = position.y
  },
  onEnd: () => {
    emits('move')
    if (item.value.visible)
      handleReset()
  },
})

const bg = computed(() => {
  return isDragging.value ? 'bg-blue shadow-lg z-99' : item.value.bgColor
})

function handleReset() {
  x.value = item.value.startX
  y.value = item.value.startY
}

defineExpose({
  handleReset,
})
</script>

<template>
  <div
    v-if="item.visible"
    ref="el"
    :style="style"
    class="fixed h-20 w-20 center rounded-lg cursor-move"
    :class="bg"
  >
    <span @click="handleReset">
      {{ item.value }}
    </span>
  </div>
</template>
