<!-- src/components/TicketMask.vue -->
<template>
  <div class="tm-wrap">
    <input class="tm-file" type="file" accept="image/*" @change="onFile" />

    <div class="tm-canvas-wrap" v-show="imgLoaded">
      <!-- 高對比浮動工具列 -->
      <div class="tm-toolbar">
        <button type="button" @click="undoMask" :disabled="masks.length === 0">復原</button>
        <button type="button" @click="clearAll" :disabled="masks.length === 0">全部清除</button>
        <button type="button" class="primary" @click="exportPng">完成並輸出</button>
      </div>

      <canvas
        ref="canvas"
        class="tm-canvas"
        @mousedown="start"
        @mousemove="move"
        @mouseup="end"
        @mouseleave="cancel"
        @touchstart.prevent="startTouch"
        @touchmove.prevent="moveTouch"
        @touchend.prevent="endTouch"
      ></canvas>
    </div>

    <!-- 匯出預覽與下載（保底機制） -->
    <div class="tm-export" v-if="exportedUrl">
      <p>已匯出預覽（同時也已 emit 給父層）</p>
      <img :src="exportedUrl" alt="masked preview" class="tm-export-img" />
      <a :href="exportedUrl" download="masked_ticket.png">下載 PNG</a>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

/**
 * 設計重點
 * - Canvas 內部尺寸 = 圖片原尺寸（確保輸出清晰）
 * - 以 CSS 等比縮放至容器寬度（行動裝置 OK）
 * - 事件座標需由顯示尺寸換算回 Canvas 內部座標
 */

const canvas = ref(null)
let ctx
const baseImg = new Image()

const imgLoaded = ref(false)
const exportedUrl = ref('')

const masks = ref([]) // 多遮蔽：{ x, y, w, h }（Canvas 內部座標）
let dragging = false
let startX = 0
let startY = 0
let previewRect = null

const emit = defineEmits(['ready'])

function onFile(e) {
  exportedUrl.value = ''
  masks.value = []
  previewRect = null
  const file = e.target.files?.[0]
  if (!file) return

  const reader = new FileReader()
  reader.onload = () => {
    baseImg.onload = () => {
      const c = canvas.value
      c.width = baseImg.width
      c.height = baseImg.height
      ctx = c.getContext('2d')
      redraw()
      imgLoaded.value = true
    }
    baseImg.src = reader.result
  }
  reader.readAsDataURL(file)
}

function toCanvasCoords(clientX, clientY) {
  const c = canvas.value
  const rect = c.getBoundingClientRect()
  const scaleX = c.width / rect.width
  const scaleY = c.height / rect.height
  return { x: (clientX - rect.left) * scaleX, y: (clientY - rect.top) * scaleY }
}

function normalizeRect(x0, y0, x1, y1) {
  const x = Math.min(x0, x1)
  const y = Math.min(y0, y1)
  const w = Math.abs(x1 - x0)
  const h = Math.abs(y1 - y0)
  return { x, y, w, h }
}

function redraw() {
  if (!ctx) return
  ctx.clearRect(0, 0, canvas.value.width, canvas.value.height)
  ctx.drawImage(baseImg, 0, 0)

  // 既有遮蔽（純黑、不透明）
  ctx.fillStyle = '#000'
  for (const r of masks.value) ctx.fillRect(r.x, r.y, r.w, r.h)

  // 目前預覽（半透明）
  if (previewRect) {
    ctx.fillStyle = 'rgba(0,0,0,0.4)'
    ctx.fillRect(previewRect.x, previewRect.y, previewRect.w, previewRect.h)
  }
}

/* 滑鼠 */
function start(e) {
  if (!ctx) return
  dragging = true
  const p = toCanvasCoords(e.clientX, e.clientY)
  startX = p.x; startY = p.y
  previewRect = { x: startX, y: startY, w: 0, h: 0 }
  redraw()
}
function move(e) {
  if (!dragging || !ctx) return
  const p = toCanvasCoords(e.clientX, e.clientY)
  previewRect = normalizeRect(startX, startY, p.x, p.y)
  redraw()
}
function end(e) {
  if (!dragging || !ctx) return
  dragging = false
  const p = toCanvasCoords(e.clientX, e.clientY)
  const rect = normalizeRect(startX, startY, p.x, p.y)
  if (rect.w > 2 && rect.h > 2) masks.value.push(rect)
  previewRect = null
  redraw()
}
function cancel() {
  if (!dragging) return
  dragging = false
  previewRect = null
  redraw()
}

/* 觸控 */
function startTouch(e) {
  const t = e.touches[0]
  start({ clientX: t.clientX, clientY: t.clientY })
}
function moveTouch(e) {
  const t = e.touches[0]
  move({ clientX: t.clientX, clientY: t.clientY })
}
function endTouch() {
  end({ clientX: startX, clientY: startY })
}

/* 操作 */
function undoMask() {
  if (!masks.value.length) return
  masks.value.pop()
  redraw()
}
function clearAll() {
  masks.value = []
  previewRect = null
  redraw()
}
function exportPng() {
  if (!canvas.value) return
  previewRect = null
  redraw()
  const dataUrl = canvas.value.toDataURL('image/png')
  exportedUrl.value = dataUrl
  emit('ready', dataUrl)
}
</script>

<style scoped>
.tm-wrap {
  max-width: 960px;
  margin: 0 auto;
  padding: 12px;
}
.tm-file { display: block; }

/* 畫布容器（定位用） */
.tm-canvas-wrap {
  position: relative;
  width: 100%;
  max-width: 820px;
  margin-top: 10px;
}

/* 高對比浮動工具列：固定在右上角，任何底圖上都清楚 */
.tm-toolbar {
  position: absolute;
  top: 8px;
  right: 8px;
  z-index: 20;
  display: flex;
  gap: 8px;
  padding: 8px;
  border-radius: 12px;
  background: rgba(0, 0, 0, 0.55); /* 深色半透明底 */
  backdrop-filter: blur(6px);       /* 玻璃霧面，讓字更清楚 */
  box-shadow: 0 2px 8px rgba(0,0,0,.25);
}
.tm-toolbar button {
  font-size: 14px;
  line-height: 1;
  padding: 8px 12px;
  border-radius: 10px;
  border: 1px solid rgba(255,255,255,0.35);
  color: #fff;                      /* 白字 */
  background: rgba(255,255,255,0.08);
  text-shadow: 0 1px 2px rgba(0,0,0,0.6); /* 文字陰影提升對比 */
  cursor: pointer;
}
.tm-toolbar button.primary {
  border-color: rgba(255,255,255,0.5);
  background: rgba(255,255,255,0.16);
  font-weight: 700;
}
.tm-toolbar button:disabled {
  opacity: 0.55;
  cursor: not-allowed;
}

/* Canvas 響應式：外部縮放、內部保持原始像素 */
.tm-canvas {
  width: 100%;
  height: auto;
  display: block;
  border: 1px solid #e5e5e5;
  border-radius: 10px;
  touch-action: none; /* 避免觸控捲動干擾 */
}

/* 匯出區 */
.tm-export { margin-top: 12px; }
.tm-export-img {
  max-width: 360px;
  width: 100%;
  height: auto;
  border: 1px solid #eee;
  border-radius: 8px;
  display: block;
  margin-bottom: 6px;
}
</style>
