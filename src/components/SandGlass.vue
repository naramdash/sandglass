<script setup lang="ts">
import { onMounted, onUnmounted, ref } from 'vue';

const props = defineProps<{ body?: HTMLBodyElement, isPIP: boolean }>()

if (props.isPIP && props.body) {
  props.body.style.backgroundColor = 'black'
  props.body.style.margin = '0'
}

const containerRef = ref<HTMLDivElement>()
const canvasRef = ref<HTMLCanvasElement>()
const context2D = ref<CanvasRenderingContext2D>()

const startTime = ref(new Date(localStorage.getItem('start')!))
const lunchTime = ref(new Date(localStorage.getItem('lunch')!))
const passColor = ref(localStorage.getItem('passColor')!)
const remainColor = ref(localStorage.getItem('remainColor')!)
const lunchTerm = ref(localStorage.getItem('lunchTerm')!)

const goHomeVisible = ref(false)
const secondGoesBy = ref(0)
const configLoadIntervalId = setInterval(() => {
  startTime.value = new Date(localStorage.getItem('start')!)
  lunchTime.value = new Date(localStorage.getItem('lunch')!)
  passColor.value = localStorage.getItem('passColor')!
  remainColor.value = localStorage.getItem('remainColor')!
  lunchTerm.value = localStorage.getItem('lunchTerm')!
}, 1000)

const canvasRenderIntervalId = setInterval(() => {
  if (canvasRef.value == null || context2D.value == null) return

  secondGoesBy.value = Math.floor((new Date().getTime() - startTime.value.getTime()) / 1000)
  goHomeVisible.value = secondGoesBy.value >= 32400 && secondGoesBy.value % 2 === 0

  const todaylunchTime = new Date(startTime.value)
  todaylunchTime.setHours(lunchTime.value.getHours())
  todaylunchTime.setMinutes(lunchTime.value.getMinutes())
  todaylunchTime.setSeconds(0)
  const secondsToLunchFromStart = Math.floor((todaylunchTime.getTime() - startTime.value.getTime()) / 1000)
  
  const todayLunchEnd = new Date(todaylunchTime.getTime() + parseInt(lunchTerm.value) * 60 * 1000)
  const secondsToLunchEndFromStart = Math.floor((todayLunchEnd.getTime() - startTime.value.getTime()) / 1000)
  
  const renderer = context2D.value

  const rowWidth = canvasRef.value.width
  const rowHeight = canvasRef.value.height / (30 * 10)
  const cellWidth = rowWidth / (18 * 6)

  renderer.font = 'bold 15px system-ui'
  renderer.lineWidth = 1
  renderer.setLineDash([6, 16])
  renderer.clearRect(0, 0, canvasRef.value.width, canvasRef.value.height)

  for (const row of Array(30 * 10).keys()) {
    renderer.fillStyle = passColor.value
    if (secondGoesBy.value >= (row + 1) * (18 * 6)) {
      // 이미 지난 시간
      renderer.fillRect(0, rowHeight * row, rowWidth, rowHeight)
    } else if ((row + 1) * (18 * 6) >= secondGoesBy.value && secondGoesBy.value >= (row) * (18 * 6)) {
      // 지나고 있는 시간
      const diff = secondGoesBy.value - (row) * (18 * 6)
      renderer.fillRect(0, rowHeight * row, diff * cellWidth, rowHeight)
    }

    if((row + 1) * (18 * 6) >= secondsToLunchFromStart && secondsToLunchFromStart >= (row) * (18 * 6)) {
      renderer.strokeStyle = 'gold'
      renderer.fillStyle = 'gold'
      renderer.beginPath()
      renderer.moveTo(0, rowHeight * row)
      renderer.lineTo(rowWidth, rowHeight * row)
      renderer.closePath()
      renderer.stroke()
      renderer.fillText(`Lunch Start`, rowWidth - 90, rowHeight * row - 5)
    }

    if((row + 1) * (18 * 6) >= secondsToLunchEndFromStart && secondsToLunchEndFromStart >= (row) * (18 * 6)) {
      renderer.strokeStyle = 'coral'
      renderer.fillStyle = 'coral'
      renderer.beginPath()
      renderer.moveTo(0, rowHeight * row)
      renderer.lineTo(rowWidth, rowHeight * row)
      renderer.closePath()
      renderer.stroke()
      renderer.fillText(`Lunch End`, rowWidth - 82, rowHeight * row - 5)
    }
  }

  for (const row of Array(8).keys()) {
    const heightBottom = rowHeight * (300 / 9) * (row + 1)
    const heightTop = heightBottom - 8

    renderer.fillStyle = 'white'
    renderer.fillText(` ${(startTime.value.getHours() + row + 1).toString().padStart(2, '0')}:${(startTime.value.getMinutes()).toString().padStart(2, '0')}`, 0, heightTop)

    renderer.strokeStyle = 'black'
    renderer.fillStyle = ''
    renderer.beginPath()
    renderer.moveTo(0, heightBottom)
    renderer.lineTo(rowWidth, heightBottom)
    renderer.closePath()
    renderer.stroke()
  }

  renderer.font = 'bold 15px monospace'
  renderer.fillStyle = 'white'
  renderer.fillText(` ${(startTime.value.getHours() + 9).toString().padStart(2, '0')}:${(startTime.value.getMinutes()).toString().padStart(2, '0')}`, 0, canvasRef.value.height - 15)
}, 500)

const resizeObserver = new ResizeObserver(([container]) => {
  if (canvasRef.value == null) return

  canvasRef.value.width = container.contentRect.width
  canvasRef.value.height = container.contentRect.height
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


