<!-- src/App.vue -->
<template>
  <main class="container">
    <h1>ShowPassport — 前端雛形</h1>
    <section class="card">
      <h2>1) 上傳與本地遮蔽</h2>
      <TicketMask @ready="onMaskedReady" />
    </section>

    <section class="card">
      <h2>2) OCR / 表單自動填入（四欄）</h2>
      <OcrSubmit :maskedImage="maskedImage" @ocr="onOcr" />
    </section>

    <section class="card">
      <h2>3) 章戳收藏（演唱會類型示例）</h2>
      <StampGrid :venueSlug="ocr.venueSlug" :category="ocr.category" />
    </section>
  </main>
</template>

<script setup>
import { ref } from 'vue'
import TicketMask from './components/TicketMask.vue'
import OcrSubmit from './components/OcrSubmit.vue'
import StampGrid from './components/StampGrid.vue'

const maskedImage = ref(null) // data URL (PNG)
const ocr = ref({
  venueSlug: '',
  performer: '',
  title: '',
  datetime: '',
  category: '' // 我們示範: 若偵測為演唱會 -> "concert"
})

function onMaskedReady(dataUrl) {
  maskedImage.value = dataUrl
}

function onOcr(payload) {
  ocr.value = payload
}
</script>

<style>
.container { max-width: 960px; margin: 24px auto; padding: 0 12px; }
.card { background: #fff; border: 1px solid #eee; border-radius: 10px; padding: 16px; margin-bottom: 20px; }
h1 { font-size: 22px; margin: 12px 0 20px; }
h2 { font-size: 18px; margin-bottom: 12px; }
button { cursor: pointer; }
</style>
