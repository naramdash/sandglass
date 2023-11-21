<script setup lang="ts">
import { createApp, ref, watchEffect } from 'vue'
import SandGlass from './components/SandGlass.vue';

const passColor = ref(localStorage.getItem('passColor') ?? '#4aa587')
const remainColor = ref(localStorage.getItem('remainColor') ?? '#3b2121')
const start = ref((() => {
  const date = new Date(localStorage.getItem('start') ?? new Date().toISOString())
  return `${date.getHours().toString().padStart(2, '0')}:${date.getMinutes().toString().padStart(2, '0')}`
})())

watchEffect(() => {
  const [h, m] = start.value.split(':')
  const date = new Date()
  date.setHours(parseInt(h))
  date.setMinutes(parseInt(m))
  localStorage.setItem('start', date.toISOString())
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
    Start time
    <input type="time" v-model="start" />
    <button type="button" @click="setStartNow()">now</button>
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

<style>
label {
  display: block;
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

