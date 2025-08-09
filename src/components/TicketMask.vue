<!-- src/components/TicketMask.vue -->
<template>
  <div>
    <input type="file" accept="image/*" @change="onFile" />
    <div v-if="imgLoaded" style="position:relative; display:inline-block; margin-top:10px;">
      <canvas ref="canvas" @mousedown="start" @mousemove="move" @mouseup="end"></canvas>
      <div style="margin-top:8px;">
        <button @click="clearMask">清除遮蔽</button>
        <button @click="exportPng">完成並輸出（不離開瀏覽器）</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const canvas = ref(null)
let ctx, baseImg = new Image()
let drawing = false
let startX = 0, startY = 0
let imgLoaded = ref(false)
const emit = defineEmits(['ready'])

function onFile(e) {
  const file = e.target.files?.[0]
  if (!file) return
  const reader = new FileReader()
  reader.onload = () => {
    baseImg.onload = () => {
      const c = canvas.value
      c.width = baseImg.width
      c.height = baseImg.height
      ctx = c.getContext('2d')
      ctx.drawImage(baseImg, 0, 0)
      imgLoaded.value = true
    }
    baseImg.src = reader.result
  }
  reader.readAsDataURL(file)
}

function start(e) {
  if (!ctx) return
  drawing = true
  const rect = canvas.value.getBoundingClientRect()
  startX = e.clientX - rect.left
  startY = e.clientY - rect.top
}

function move(e) {
  if (!drawing || !ctx) return
  const c = canvas.value
  const rect = c.getBoundingClientRect()
  const x = e.clientX - rect.left
  const y = e.clientY - rect.top
  // 重畫底圖與半透明遮罩框
  ctx.drawImage(baseImg, 0, 0)
  ctx.fillStyle = 'rgba(0,0,0,0.5)'
  const w = x - startX
  const h = y - startY
  ctx.fillRect(startX, startY, w, h)
}

function end(e) {
  if (!drawing || !ctx) return
  drawing = false
  // 最終將遮罩畫成純黑（不透明）
  const rect = canvas.value.getBoundingClientRect()
  const x = e.clientX - rect.left
  const y = e.clientY - rect.top
  const w = x - startX
  const h = y - startY
  ctx.drawImage(baseImg, 0, 0)
  ctx.fillStyle = '#000'
  ctx.fillRect(startX, startY, w, h)
}

function clearMask() {
  if (!ctx) return
  ctx.drawImage(baseImg, 0, 0)
}

function exportPng() {
  if (!canvas.value) return
  const dataUrl = canvas.value.toDataURL('image/png')
  emit('ready', dataUrl) // 傳給父層（作為遮蔽後要送 OCR 的圖）
}
</script>
