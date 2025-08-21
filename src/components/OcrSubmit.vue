<template>
  <div class="ocr-card">
    <div class="row">
      <button class="primary" :disabled="loading || !maskedImage" @click="submitOcr">
        {{ loading ? (demo ? 'Mocking…' : 'Analyzing…') : '送出 /api/ocr' }}
      </button>

      <label class="demo-toggle">
        <input type="checkbox" v-model="demo" />
        無後端 DEMO
      </label>
      <span class="tip">（需先完成上傳與遮蔽，才會有 maskedImage）</span>
    </div>

    <!-- DEMO 模式下，可先在前端調整欄位，再觸發 emit -->
    <details v-if="demo" class="demo-panel">
      <summary>DEMO 欄位（可修改）</summary>
      <div class="grid">
        <label>venueSlug <input v-model="demoForm.venueSlug" /></label>
        <label>performer <input v-model="demoForm.performer" /></label>
        <label>title <input v-model="demoForm.title" /></label>
        <label>datetime <input v-model="demoForm.datetime" /></label>
      </div>
    </details>

    <div v-if="error" class="error">{{ error }}</div>

    <div v-if="result" class="result">
      <div class="field"><span class="k">venueSlug</span><span class="v">{{ result.venueSlug }}</span></div>
      <div class="field"><span class="k">performer</span><span class="v">{{ result.performer }}</span></div>
      <div class="field"><span class="k">title</span><span class="v">{{ result.title }}</span></div>
      <div class="field"><span class="k">datetime</span><span class="v">{{ result.datetime }}</span></div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  maskedImage: { type: String, default: '' } // data URL
})
const emit = defineEmits(['ocr'])

const loading = ref(false)
const error = ref('')
const result = ref(null)

// DEMO（無後端）開關與預設欄位
const demo = ref(true)
const demoForm = ref({
  venueSlug: 'nippon-budokan',
  performer: 'Hoshino Gen',
  title: 'POP VIRUS LIVE TOUR',
  datetime: '2025-08-08 19:00'
})

async function submitOcr() {
  error.value = ''
  result.value = null
  if (!props.maskedImage) {
    error.value = '尚未取得遮蔽後影像（maskedImage）'
    return
  }
  loading.value = true
  try {
    if (demo.value) {
      // —— DEMO：前端直接產出 payload ——
      await new Promise(r => setTimeout(r, 600))
      const payload = { ...demoForm.value }
      result.value = payload
      emit('ocr', payload)
      return
    }

    // —— 真實呼叫 /api/ocr ——
    const blob = await (await fetch(props.maskedImage)).blob()
    const form = new FormData()
    form.append('image', blob, 'masked.png')

    const resp = await fetch('/api/ocr', { method: 'POST', body: form })
    if (!resp.ok) throw new Error(`/api/ocr 失敗：${resp.status}`)
    const data = await resp.json()

    const payload = {
      venueSlug: data.venueSlug || data.venue || '',
      performer: data.performer || data.artist || '',
      title: data.title || data.eventTitle || '',
      datetime: data.datetime || data.eventDate || ''
    }
    result.value = payload
    emit('ocr', payload)

    // （可選）回報 visit
    try {
      await fetch('/api/visit', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      })
    } catch (e) {
      console.warn('visit failed', e)
    }
  } catch (e) {
    console.error(e)
    // 若關閉 demo 且後端失敗，可回退到 DEMO 體驗（避免卡死）
    error.value = e.message || 'OCR 發生錯誤'
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.ocr-card { display: grid; gap: 10px; }
.row { display: flex; align-items: center; gap: 12px; }
.primary {
  padding: 10px 14px; border-radius: 10px; background:#0ea5e9; color:#fff;
  border: none; cursor: pointer; font-weight: 700;
}
.primary:disabled { opacity:.5; cursor:not-allowed; }
.demo-toggle { display:flex; align-items:center; gap:6px; font-size:13px; color:#568; }
.tip { font-size:12px; color:#667; }
.error { color:#d33; font-size: 13px; }
.result {
  display: grid; gap: 6px; padding: 10px; border: 1px solid #e5e7eb;
  border-radius: 10px; background: #fafafa;
}
.field { display: grid; grid-template-columns: 120px 1fr; gap: 8px; }
.k { color:#555; font-size: 12px; letter-spacing:.08em; text-transform:uppercase; }
.v { font-weight: 700; }
.demo-panel { padding: 8px 10px; background:#0b1320; color:#d2e8ff; border-radius:10px; }
.demo-panel .grid { display:grid; grid-template-columns: repeat(2,minmax(0,1fr)); gap:8px; margin-top:8px; }
.demo-panel input { width:100%; padding:8px 10px; border-radius:8px; border:1px solid #2b3a4a; background:#0f1b2a; color:#eaf6ff; }
@media (max-width:560px){ .demo-panel .grid{ grid-template-columns: 1fr; } }
</style>