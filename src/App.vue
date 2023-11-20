<script setup lang="ts">
import { createApp, ref, watchEffect } from 'vue'
import SandGlass from './components/SandGlass.vue';

const startDate = localStorage.getItem('start') ? new Date(localStorage.getItem('start')!) : new Date()
const start = ref(`${startDate.getHours()}:${startDate.getMinutes()}`)
function setStartNow() {
  const now = new Date()
  start.value = `${now.getHours()}:${now.getMinutes()}`
}
watchEffect(() => {
  const [hour, minute] = start.value.split(':')

  const today = new Date()
  today.setHours(parseInt(hour))
  today.setMinutes(parseInt(minute))

  localStorage.setItem('start', today.toISOString())
})

const passColor = ref(localStorage.getItem('passColor') ??'#4aa587')
const remainColor = ref(localStorage.getItem('remainColor') ??'#3b2121')
watchEffect(() => {
  localStorage.setItem('passColor', passColor.value)
  localStorage.setItem('remainColor', remainColor.value)
})

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

    createApp(SandGlass, { body: pipWindow.document.body }).mount(pipMountElement)
  }
}
</script>

<template>
  <h1>IT JUST ONLY 32400 SECONDS</h1>

  <label>
    Start time
    <input type="time" v-model="start" />
    <button type="button" @click="setStartNow()">now</button>
  </label>
  <label>
    Pass color
    <input type="color" v-model="passColor" />
  </label>
  <label>
    Remain color
    <input type="color" v-model="remainColor" />
  </label>
  <hr />
  <button class="flip" @click="openPIP()">
    <span style="color: aquamarine">F </span>
    <span style="color: burlywood">L </span>
    <span style="color: cornflowerblue">I </span>
    <span style="color: darkmagenta">P </span>
    <span style="color: ">{{ ` __   ` }} {{ `   __ ` }}</span>
    <span style="color: darkorange">S </span>
    <span style="color: fuchsia">A </span>
    <span style="color: darkslateblue">N </span>
    <span style="color: green">D </span>
    <span style="color: deepskyblue">G </span>
    <span style="color: greenyellow">L </span>
    <span style="color: black">A </span>
    <span style="color: indianred">S </span>
    <span style="color: lightsalmon">S </span>

  </button>
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
</style>

