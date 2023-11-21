<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue';

const props = defineProps<{ body?: HTMLBodyElement, isPIP: boolean }>()

if(props.isPIP && props.body) {
  props.body.style.backgroundColor = 'black'
  props.body.style.margin = '0'
}

const containerRef = ref<HTMLDivElement>()
const canvasRef = ref<HTMLCanvasElement>()
const context2D = ref<CanvasRenderingContext2D>()

const startTime = ref(new Date(localStorage.getItem('start')!))
const passColor = ref(localStorage.getItem('passColor')!)
const remainColor = ref(localStorage.getItem('remainColor')!)

const goHomeVisible = ref(false)
const secondGoesBy = ref(0)
const configLoadIntervalId = setInterval(() => {
  startTime.value = new Date(localStorage.getItem('start')!)
  passColor.value = localStorage.getItem('passColor')!
  remainColor.value = localStorage.getItem('remainColor')!
}, 1000)

const canvasRenderIntervalId = setInterval(() => {
  if (canvasRef.value == null || context2D.value == null) return

  secondGoesBy.value = Math.floor((new Date().getTime() - startTime.value.getTime()) / 1000)
  goHomeVisible.value = secondGoesBy.value >= 32400 && secondGoesBy.value % 2 === 0

  const renderer = context2D.value
  renderer.fillStyle = passColor.value

  const rowWidth = canvasRef.value.width
  const rowHeight = canvasRef.value.height / (30 * 10)
  const cellWidth = rowWidth / (18 * 6)

  renderer.clearRect(0, 0, canvasRef.value.width, canvasRef.value.height)
  for (const row of Array(30 * 10).keys()) {
    if (secondGoesBy.value >= (row + 1) * (18 * 6)) {
      renderer.fillRect(0, rowHeight * row, rowWidth, rowHeight)
    } else if ((row + 1) * (18 * 6) >= secondGoesBy.value && secondGoesBy.value >= (row) * (18 * 6)) {
      const diff = secondGoesBy.value - (row) * (18 * 6)
      renderer.fillRect(0, rowHeight * row, diff * cellWidth, rowHeight)
    }
  }

  renderer.strokeStyle = 'black'
  renderer.lineWidth = 1
  renderer.setLineDash([6, 16])
  renderer.fillStyle = ''
  for (const row of Array(7).keys()) {
    renderer.beginPath()
    renderer.moveTo(0, rowHeight * (300 / 8) * (row + 1))
    renderer.lineTo(rowWidth, rowHeight * (300 / 8) * (row + 1))
    renderer.closePath()
    renderer.stroke()
  }
}, 500)

const resizeObserver = new ResizeObserver(([container]) => {
  const { width, height } = container.contentRect

  const canvas = canvasRef.value
  if (canvas == null) return

  canvas.width = width
  canvas.height = height
})

onMounted(() => {
  resizeObserver.observe(containerRef.value!)
  context2D.value = canvasRef.value!.getContext('2d') ?? undefined
})
onUnmounted(() => {
  resizeObserver.disconnect()
  clearInterval(configLoadIntervalId)
  clearInterval(canvasRenderIntervalId)
})


const minuteLeftVisible = ref(false)
let minuteVisibleTimeoutId: ReturnType<typeof setInterval>
function onShowMinutesLeft() {
  clearTimeout(minuteVisibleTimeoutId)
  minuteLeftVisible.value = true
  minuteVisibleTimeoutId = setTimeout(() => { minuteLeftVisible.value = false }, 2000);
}
</script>

<template>
  <div ref="containerRef" style="margin: 0; padding: 0; position: relative;"
    :style="{ width: props.isPIP ? '100vw' : '250px', height: props.isPIP ? '100vh' : '600px', backgroundColor: remainColor }">
    <canvas ref="canvasRef" width="10" height="10" @click="onShowMinutesLeft()" />
    <div v-if="secondGoesBy < 32400 && minuteLeftVisible"
      style="position: absolute; top: 0; left: 0; margin: 0; padding: 0; width: 100%; height: 100%; display: flex; justify-content: center; align-items: center; pointer-events: none;">
      <span style="font-size: x-large; color: white; text-shadow: #000000 0px 0 5px;">
        {{ `M-${Math.ceil((32400 - secondGoesBy) / 60)}` }}
      </span>
    </div>
    <div v-if="goHomeVisible"
      style="position: absolute; top: 0; left: 0; margin: 0; padding: 0; width: 100%; height: 100%; display: flex; justify-content: center; align-items: center; pointer-events: none;">
      <span style="font-size: xx-large; font-weight: bolder; text-shadow: #ffffff 0px 0 10px;">
        GO HOME
      </span>
    </div>
  </div>
</template>


