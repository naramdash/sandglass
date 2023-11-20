<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue';

const props = defineProps<{ body: HTMLBodyElement }>()

props.body.style.backgroundColor = 'black'
props.body.style.margin = '0'

const cellWidth = `${100 / 108}vw`
const cellHeight = `${100 / 300}vh`

// 60 * 9 = 30 * 10 * 18 * 6
const goHomeVisible = ref(false)
const secondGoesBy = ref(0)
function onInterval() {
  const start = new Date(localStorage.getItem('start')!)
  secondGoesBy.value = Math.floor((new Date().getTime() - start.getTime()) / 1000)
  goHomeVisible.value = secondGoesBy.value >= 32400 && secondGoesBy.value % 2 === 0
}

let intervalId: ReturnType<typeof setInterval>
onMounted(() => {
  intervalId = setInterval(onInterval, 1000);
})

onUnmounted(() => {
  clearInterval(intervalId)
})

function cellColor(row: number, column: number) {
  const second = ((row - 1) * 108 + column)
  const passColor = localStorage.getItem('passColor')!
  const remainColor = localStorage.getItem('remainColor')!

  return secondGoesBy.value > second ? passColor : remainColor
}

const minuteLeftVisible = ref(false)
let timeoutId: ReturnType<typeof setInterval>
props.body.addEventListener('click', () => {
  clearTimeout(timeoutId)
  minuteLeftVisible.value = true
  timeoutId = setTimeout(() => {
    minuteLeftVisible.value = false
  }, 2000);
})
</script>

<template>
  <div v-for="row in 300" style="margin: 0; padding: 0; display: flex; flex-direction: row;">
    <div v-for="column in 108" style="margin: 0; padding: 0; "
      :style="{ width: cellWidth, height: cellHeight, backgroundColor: cellColor(row, column) }">
    </div>
  </div>

  <div v-if="goHomeVisible"
    style="position: fixed; top: 0; left: 0; margin: 0; padding: 0; width: 100vw; height: 100vh; display: flex; justify-content: center; align-items: center; pointer-events: none;">
    <span style="font-size: xx-large; font-weight: bolder;">
      GO HOME
    </span>
  </div>

  <div v-if="minuteLeftVisible"
    style="position: fixed; top: 0; left: 0; margin: 0; padding: 0; width: 100vw; height: 100vh; display: flex; justify-content: center; align-items: center; pointer-events: none;">
    <span style="font-size: large; color: white;">
      {{ `M-${Math.floor((32400 - secondGoesBy) / 60)}` }}
    </span>
  </div>
</template>


