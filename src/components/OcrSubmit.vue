<!-- src/components/OcrSubmit.vue -->
<template>
  <div>
    <p>遮蔽後影像：<strong>{{ maskedImage ? '已就緒' : '尚未產生' }}</strong></p>
    <button :disabled="!maskedImage || loading" @click="callOcr">送出 /api/ocr</button>
    <button style="margin-left:8px" @click="mockOcr">（無後端時）使用本地 Mock</button>

    <div v-if="result" style="margin-top:12px;">
      <h3>OCR 結果（四欄）</h3>
      <ul>
        <li>場地 slug：{{ result.venueSlug }}</li>
        <li>演出者：{{ result.performer }}</li>
        <li>活動標題：{{ result.title }}</li>
        <li>日期時間：{{ result.datetime }}</li>
      </ul>
      <p>類型（示範）：<code>{{ result.category }}</code></p>
      <button @click="submitVisit">送 /api/visit（寫入 DB）</button>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  maskedImage: String
})
const emit = defineEmits(['ocr'])
const loading = ref(false)
const result = ref(null)

watch(result, v => {
  if (v) emit('ocr', v)
})

async function callOcr() {
  if (!props.maskedImage) return
  loading.value = true
  try {
    const res = await fetch('/api/ocr', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ image: props.maskedImage })
    })
    const data = await res.json()
    // 後端應回：venueSlug, performer, title, datetime
    result.value = {
      venueSlug: data.venueSlug,
      performer: data.performer,
      title: data.title,
      datetime: data.datetime,
      // 如果是演唱會類型，我們示範一律標記 "concert"
      category: 'concert'
    }
  } catch (e) {
    alert('OCR 呼叫失敗，改用本地 Mock。')
    mockOcr()
  } finally {
    loading.value = false
  }
}

function mockOcr() {
  result.value = {
    venueSlug: 'nippon-budokan',
    performer: 'Artist Name',
    title: 'Live at Budokan',
    datetime: '2025-08-08 19:00',
    category: 'concert'
  }
}

async function submitVisit() {
  if (!result.value) return
  try {
    const res = await fetch('/api/visit', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(result.value)
    })
    const data = await res.json()
    alert(`已記錄：${result.value.venueSlug}，該場地累計：${data.visitCount || '(未知)'}`)
  } catch (e) {
    alert('提交 visit 失敗（你可以之後接上後端再試）')
  }
}
</script>
