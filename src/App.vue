<script setup lang="ts">
import { createApp, ref, watchEffect } from 'vue'
import SandGlass from './components/SandGlass.vue';

const passColor = ref(localStorage.getItem('passColor') ?? '#4aa587')
const remainColor = ref(localStorage.getItem('remainColor') ?? '#3b2121')
const lunchTerm = ref(localStorage.getItem('lunchTerm') ?? '60')

const start = ref((() => {
  const date = new Date(localStorage.getItem('start') ?? new Date().toISOString())
  return `${date.getHours().toString().padStart(2, '0')}:${date.getMinutes().toString().padStart(2, '0')}`
})())
const lunch = ref((() => {
  const date = new Date(localStorage.getItem('lunch') ?? "2020-01-01T12:00:00")
  return `${date.getHours().toString().padStart(2, '0')}:${date.getMinutes().toString().padStart(2, '0')}`
})())

watchEffect(() => {
  const [startHour, startMinute] = start.value.split(':')
  const startDate = new Date()
  startDate.setHours(parseInt(startHour))
  startDate.setMinutes(parseInt(startMinute))
  localStorage.setItem('start', startDate.toISOString())

  const [lunchHour, lunchMinute] = lunch.value.split(':')
  const lunchDate = new Date()
  lunchDate.setHours(parseInt(lunchHour))
  lunchDate.setMinutes(parseInt(lunchMinute))
  localStorage.setItem('lunch', lunchDate.toISOString())

  localStorage.setItem('lunchTerm', lunchTerm.value)
  localStorage.setItem('passColor', passColor.value)
  localStorage.setItem('remainColor', remainColor.value)
})

function setStartNow() {
  start.value = `${new Date().getHours().toString().padStart(2, '0')}:${new Date().getMinutes().toString().padStart(2, '0')}`
}
function setPassColorDefault() { passColor.value = '#4aa587' }
function setRemainColorDefault() { remainColor.value = '#3b2121' }

async function openPIP() {
  // @ts-ignore
  if ((window as unknown).documentPictureInPicture == null) {
    alert('try this on edge or chrome')
  } else {
    // @ts-ignore
    const pipWindow = await (window.documentPictureInPicture).requestWindow({
      width: 240,
      height: 600,
    }) as Window;

    const pipMountElement = pipWindow.document.createElement('div')
    pipWindow.document.body.id = 'pip-body'
    pipWindow.document.body.append(pipMountElement)

    createApp(SandGlass, { body: pipWindow.document.body, isPIP: true }).mount(pipMountElement)
  }
}
</script>

<template>
  <h1>IT JUST ONLY 32400 SECONDS (VERSION 2 wow)</h1>

  <label>
    Start Time
    <input type="time" v-model="start" />
    <button type="button" @click="setStartNow()">now</button>
  </label>
  <label>
    Lunch Time
    <input type="time" v-model="lunch" />
    <input type="range" v-model="lunchTerm" min="30" step="10" max="120" list="lunch-time-markers" />
    {{ lunchTerm }} minutes
    
    <datalist id="lunch-time-markers">
      <option value="30" label="good"></option>
      <option value="40" label="40"></option>
      <option value="50" label="50"></option>
      <option value="60" label="60"></option>
      <option value="70" label="70"></option>
      <option value="80" label="80"></option>
      <option value="90" label="90"></option>
      <option value="100" label="100"></option>
      <option value="110" label="110"></option>
      <option value="120" label="120"></option>
    </datalist>


  </label>
  <label>
    Pass color
    <input type="color" v-model="passColor" />
    <button type="button" class="reset" style="background-color: #4aa587;" @click="setPassColorDefault()">reset</button>
  </label>
  <label>
    Remain color
    <input type="color" v-model="remainColor" />
    <button type="button" class="reset" style="background-color: #3b2121;;"
      @click="setRemainColorDefault()">reset</button>
  </label>
  <hr />
  <button class="flip" @click="openPIP()">
    <span style="color: aquamarine">F </span>
    <span style="color: burlywood">L </span>
    <span style="color: cornflowerblue">I </span>
    <span style="color: darkmagenta">P </span>
    <span style="color: ">{{ ` __ ` }} {{ ` __ ` }}</span>
    <span style="color: darkorange">S </span>
    <span style="color: fuchsia">A </span>
    <span style="color: darkslateblue">N </span>
    <span style="color: green">D </span>
    <span style="color: deepskyblue">G </span>
    <span style="color: greenyellow">L </span>
    <span style="color: black">A </span>
    <span style="color: indianred">S </span>
    <span style="color: lightsalmon">S </span>
    (open in Document PIP)
  </button>

  <hr />

  <SandGlass :isPIP="false" />
</template>

<style scoped>
label {
  display: block;
  width: fit-content;
}

button.flip {
  width: 100%;
  height: 50px;
  font-size: larger;
  font-weight: bolder;
  background-color: gray;
}

button.reset {
  color: white;
  font-weight: bolder;
}
</style>

