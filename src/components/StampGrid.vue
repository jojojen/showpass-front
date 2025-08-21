<template>
  <div class="passport">
    <!-- 網格容器，展示多個印章格子 -->
    <div class="grid">
      <!-- 產生六個格子，可依需求增加或減少 -->
      <div class="cell" v-for="i in 6" :key="i">
        <!-- 當 shouldStamp 成立且索引相符時，顯示印章 -->
        <div v-if="isStamped(i)" class="stamp-wrapper">
          <img :src="budokanStampSrc" alt="stamp" class="stamp-image" />
        </div>
        <!-- 若尚未蓋章，顯示佔位符 -->
        <div v-else class="placeholder">+</div>
      </div>
    </div>
  </div>
</template>

<script setup>
// StampGrid 用來展示演出場館蓋章的格子，並在需要時觸發蓋章動畫。
import { computed } from 'vue'

// 接收父元件傳入的場館代碼與分類
const props = defineProps({
  venueSlug: { type: String, default: '' },
  category: { type: String, default: '' }
})

// 預設範例圖檔，可替換為各場館對應的圖檔
const budokanStampSrc = '/stamps/Nippon_Budokan.png'

// 判斷是否要蓋章，例：當類型為演唱會且場館為 nippon‑budokan
const shouldStamp = computed(() => {
  return props.category === 'concert' && props.venueSlug === 'nippon-budokan'
})

// 判斷指定格子是否被蓋章；這裡簡化為只有第一格蓋章
function isStamped(i) {
  return shouldStamp.value && i === 1
}
</script>

<style scoped>
/* 網格布局 */
.grid {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 10px;
}
.cell {
  width: 100%;
  aspect-ratio: 1 / 1;
  border: 1px dashed #bbb;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #fafafa;
}
.placeholder {
  color: #bbb;
  font-size: 20px;
}
.stamp-wrapper {
  position: relative;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}
/* 印章圖片樣式與動畫 */
.stamp-image {
  width: 90%;
  object-fit: contain;
  filter: grayscale(100%);
  animation: stamp-drop 0.8s ease-out forwards;
  transform-origin: center center;
}
@keyframes stamp-drop {
  0% {
    transform: translateY(-150%) scale(1.8) rotate(-15deg);
    opacity: 0;
  }
  60% {
    transform: translateY(0%) scale(1.05) rotate(2deg);
    opacity: 1;
  }
  100% {
    transform: translateY(0%) scale(1) rotate(0deg);
    opacity: 1;
    filter: grayscale(100%) drop-shadow(2px 2px 2px rgba(0, 0, 0, 0.3));
  }
}
</style>