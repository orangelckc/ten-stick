<script lang="ts" setup>
import Item from './Item.vue'

import type { ItemProps } from '.'

const SIZE = 80
const GAP = 20
const COUNT = 10
const THRESHOLD = 2

function generate(index: number) {
  return {
    value: 1,
    startX: index * SIZE + GAP * (index + 1),
    startY: SIZE,
    endX: (SIZE + GAP) * (index + 1),
    endY: SIZE * 2,
    x: index * SIZE + GAP * (index + 1),
    y: SIZE,
    bgColor: 'bg-red',
    visible: true,
  }
}

const list = ref<ItemProps[]>([])
const refs = ref<any[]>([])
const actionQueue: ItemProps[][] = []

const isWinning = computed(() => {
  return list.value.filter(item => item.value === 0 || item.value === THRESHOLD).length === COUNT
})

function checkRegion(from: ItemProps) {
  const fromIndex = list.value.findIndex(item => item === from)
  const targetIndex = list.value.findIndex((item) => {
    return (
      from.x > item.startX
      && from.x < item.endX
      && from.y > item.startY
      && from.y < item.endY
    )
  })

  if (targetIndex === -1 || fromIndex === targetIndex)
    return

  let valueSum = 0
  const start = Math.min(fromIndex, targetIndex)
  const end = Math.max(fromIndex, targetIndex)
  for (let i = start + 1; i < end; i++)
    valueSum += list.value[i].value

  const target = list.value[targetIndex]

  if (target.value === 1 && target.visible && valueSum === THRESHOLD) {
    actionQueue.push(JSON.parse(JSON.stringify(list.value)))

    target.bgColor = 'bg-green'
    target.value += 1
    from.visible = false
    from.value = 0
  }
}

function handleReposition() {
  list.value.forEach((_, index) => {
    refs.value[index].handleReset()
  })
}

function handleUndo() {
  if (actionQueue.length) {
    list.value = actionQueue.pop()!
    handleReposition()
  }
}

function handleInit() {
  list.value = [...Array.from({ length: COUNT }, (_, index) => generate(index))]
  nextTick(() => {
    handleReposition()
  })
}

onMounted(handleInit)
</script>

<template>
  <div class="flex gap-5">
    <Item
      v-for="(item, index) in list"
      :ref="el => (refs[index] = el)"
      :key="index"
      v-model="list[index]"
      @move=" checkRegion(item)"
    />
  </div>

  <div v-if="isWinning" class="text-red">
    WINNING
  </div>
  <div class="flex gap-3">
    <button class="btn btn-ghost" @click="handleUndo">
      Undo
    </button>
    <button class="btn btn-ghost" @click="handleInit">
      Reset
    </button>
  </div>
</template>

<style lang="scss" scoped></style>
