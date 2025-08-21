<template>
  <main class="container">
    <h1>ShowPassport — 前端雛形</h1>

    <!-- 1) 上傳與本地遮蔽 -->
    <section class="card">
      <h2>1) 上傳與本地遮蔽</h2>
      <TicketMask @ready="onMaskedReady" />
    </section>

    <!-- 2) OCR / 自動填入（四欄） -->
    <section class="card">
      <h2>2) OCR / 表單自動填入（四欄）</h2>
      <OcrSubmit :maskedImage="maskedImage" @ocr="onOcr" />
    </section>

    <!-- 3) 整頁戳章（只在 OCR 成功後顯示；並以 key 觸發動畫） -->
    <section class="card">
      <h2>3) 整頁戳章（OCR 成功後）</h2>

      <div v-if="!ocrReady" class="placeholder">
        送出 /api/ocr 成功後，將顯示整頁戳章與活動資訊。<br />
        （目前支援 DEMO 模式，無後端也能測試）
      </div>

      <FullPageStamp
        v-else
        :key="ocrKey"
        :venueSlug="ocr.venueSlug"
        :performer="ocr.performer"
        :title="ocr.title"
        :datetime="ocr.datetime"
      />
      <p class="hint" v-if="ocrReady">
        ※ 圖片請放到 <code>public/stamps/Nippon_Budokan.png</code>（可依 venueSlug 擴充映射）。
      </p>
    </section>
  </main>
</template>

<script setup>
import { ref, computed } from 'vue'
import TicketMask from './components/TicketMask.vue'
import OcrSubmit from './components/OcrSubmit.vue'
import FullPageStamp from './components/FullPageStamp.vue'

const maskedImage = ref(null) // data URL (PNG)
const ocr = ref({
  venueSlug: '',
  performer: '',
  title: '',
  datetime: ''
})

// 1) TicketMask 完成遮蔽
function onMaskedReady(dataUrl) {
  maskedImage.value = dataUrl
}

// 2) OcrSubmit 成功後 emit('ocr', payload) 會進來這裡
function onOcr(payload) {
  ocr.value = {
    venueSlug: String(payload.venueSlug || ''),
    performer: String(payload.performer || ''),
    title: String(payload.title || ''),
    datetime: String(payload.datetime || '')
  }
  // 每次更新 key，強制 FullPageStamp 重新掛載以觸發 stamp 動畫
  ocrKey.value++
}

const ocrReady = computed(() => {
  return !!(ocr.value.venueSlug && ocr.value.title && ocr.value.datetime)
})

// 用來觸發 FullPageStamp 的進場動畫（每次成功都 +1）
const ocrKey = ref(0)
</script>

<style>
.container { max-width: 960px; margin: 24px auto; padding: 0 12px; }
.card {
  background: #0c0f12;
  border: 1px solid rgba(255,255,255,0.06);
  border-radius: 14px;
  padding: 16px;
  margin-bottom: 20px;
  box-shadow: 0 10px 26px rgba(0,0,0,0.2);
}
h1 { font-size: 22px; margin: 12px 0 20px; color: #e9f7ff; }
h2 { font-size: 18px; margin-bottom: 12px; color: #cae9ff; }
button { cursor: pointer; }
.hint { font-size: 12px; color: #9db6c6; margin-top: 8px; }
code { background: rgba(255,255,255,0.06); padding: 2px 6px; border-radius: 6px; }
body { background: #070b0e; color: #d8ecf7; }
.placeholder {
  font-size: 14px; color: #9fb3bf; padding: 12px;
  border: 1px dashed rgba(255,255,255,0.1); border-radius: 10px;
  background: rgba(255,255,255,0.02);
}
</style>