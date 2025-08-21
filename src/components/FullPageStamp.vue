<template>
  <section class="full-stamp" role="region" aria-label="Show stamp">
    <!-- 整頁大型戳章背景：只有在 OCR 完成且本元件被渲染時才會進場動畫 -->
    <img class="stamp-bg" :src="stampSrc" alt="venue stamp" />

    <!-- 文字資訊覆蓋層（由父層傳入、來自 API/OCR） -->
    <div class="info">
      <header class="title">{{ displayTitle }}</header>

      <div class="meta-row">
        <div class="meta">
          <span class="label">Performer</span>
          <span class="value">{{ displayPerformer }}</span>
        </div>
        <div class="meta">
          <span class="label">Venue</span>
          <span class="value">{{ displayVenue }}</span>
        </div>
      </div>

      <div class="meta-row">
        <div class="meta">
          <span class="label">Datetime</span>
          <span class="value">{{ displayDatetime }}</span>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { computed } from 'vue'

/**
 * Props（全部由父層在 OCR 成功後傳入）
 * - venueSlug：如 'nippon-budokan'
 * - performer：演出者
 * - title：表演名稱
 * - datetime：演出時間（字串）
 * - stampOverride：自訂戳章圖片路徑（可選）
 */
const props = defineProps({
  venueSlug: { type: String, default: '' },
  performer: { type: String, default: '' },
  title: { type: String, default: '' },
  datetime: { type: String, default: '' },
  stampOverride: { type: String, default: '' }
})

/** slug -> 圖檔名稱對照，可擴充 */
const STAMP_MAP = {
  'nippon-budokan': 'Nippon_Budokan.png'
}

const displayTitle = computed(() => (props.title || '').trim())
const displayPerformer = computed(() => (props.performer || '').trim())
const displayVenue = computed(() => (props.venueSlug || '').trim())
const displayDatetime = computed(() => (props.datetime || '').trim())

/** 戳章圖片路徑（優先使用 stampOverride） */
const stampSrc = computed(() => {
  if ((props.stampOverride || '').trim()) return props.stampOverride
  const key = displayVenue.value || 'nippon-budokan'
  const file = STAMP_MAP[key] || STAMP_MAP['nippon-budokan']
  // 圖片放在 public/stamps/ 底下
  return `/stamps/${file}`
})
</script>

<style scoped>
/* 整頁容器尺寸 */
.full-stamp {
  position: relative;
  width: 100%;
  height: min(72vh, 720px);
  overflow: hidden;
  background: #0c0f12 radial-gradient(120% 100% at 50% 0%, #0c2230 0%, #0b1923 60%, #070e14 100%);
  border-radius: 14px;
  border: 1px solid rgba(255,255,255,0.06);
  box-shadow: 0 12px 30px rgba(0,0,0,0.15);
}

/* 大型戳章：等比覆蓋，旋轉一點點像真實蓋章；進場動畫 */
.stamp-bg {
  position: absolute;
  inset: -6% -6% -6% -6%;
  width: 112%;
  height: 112%;
  object-fit: contain;
  opacity: 0.12; /* 水印感 */
  transform: rotate(-10deg) scale(1.3) translateY(-8%);
  filter: drop-shadow(0 10px 18px rgba(0,0,0,0.35)) saturate(0.9);
  animation: stampIn 900ms cubic-bezier(.2,.8,.2,1) both;
}

/* 文字資訊層：置中顯示 */
.info {
  position: relative;
  z-index: 2;
  width: min(820px, 92%);
  margin: 0 auto;
  height: 100%;
  color: #e7f5ff;
  display: grid;
  align-content: center;
  gap: 14px;
  text-shadow: 0 1px 0 rgba(0,0,0,0.4);
}

.title {
  font-size: clamp(22px, 4.6vw, 42px);
  font-weight: 800;
  letter-spacing: .02em;
  line-height: 1.15;
  filter: drop-shadow(0 10px 20px rgba(0,0,0,0.3));
  animation: fadeUp 500ms 220ms ease-out both;
}

.meta-row {
  display: grid;
  grid-template-columns: repeat(2, minmax(0,1fr));
  gap: 12px;
  animation: fadeUp 500ms 320ms ease-out both;
}
.meta-row + .meta-row { animation-delay: 420ms; }

.meta {
  display: grid;
  grid-template-columns: 110px 1fr;
  align-items: baseline;
  gap: 10px;
  padding: 10px 12px;
  border-radius: 10px;
  background: rgba(255,255,255,0.04);
  border: 1px solid rgba(255,255,255,0.06);
  backdrop-filter: blur(4px);
}

.label {
  font-size: 12px;
  letter-spacing: .12em;
  text-transform: uppercase;
  color: #b3d6ea;
  opacity: .85;
}

.value {
  font-size: clamp(14px, 2.2vw, 18px);
  color: #e9f7ff;
  font-weight: 600;
}

/* 進場動畫 */
@keyframes stampIn {
  0%   { opacity: 0; transform: rotate(-20deg) scale(1.8) translateY(-40%); }
  60%  { opacity: .18; transform: rotate(-12deg) scale(1.35) translateY(-10%); }
  100% { opacity: .12; transform: rotate(-10deg) scale(1.3) translateY(-8%); }
}
@keyframes fadeUp {
  0%   { opacity: 0; transform: translateY(10px); }
  100% { opacity: 1; transform: translateY(0); }
}

@media (max-width: 560px) {
  .meta-row { grid-template-columns: 1fr; }
  .meta { grid-template-columns: 96px 1fr; }
}
</style>
