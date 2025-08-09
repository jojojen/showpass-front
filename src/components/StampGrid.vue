<!-- src/components/StampGrid.vue -->
<template>
  <div class="grid">
    <!-- 範例：僅示範「concert」類型顯示 Budokan 章。
         實務上你會用 category + venueSlug 決定顯示何種章戳。 -->
    <div
      v-for="i in 12"
      :key="i"
      class="cell"
    >
      <div v-if="isStamped(i)" class="stamp">
        <img :src="budokanStampSrc" alt="Nippon Budokan Concert Stamp" />
      </div>
      <div v-else class="placeholder">＋</div>
    </div>
  </div>
  <p style="margin-top:8px; font-size:12px;">
    規則：如果 <code>category === 'concert'</code> 且 <code>venueSlug</code> 為 <code>nippon-budokan</code>，就在下一個空格蓋上「日本武道館」章。
  </p>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  venueSlug: { type: String, default: '' },
  category: { type: String, default: '' }
})

// 章戳圖檔（放 public 下可直連）
const budokanStampSrc = '/stamps/Nippon_Budokan.png'

// 假設：每送一次符合條件的紀錄，就蓋一枚章。
// 這裡為簡化：若條件成立，就顯示第一格章；可改為來自後端的歷史數組決定多枚章。
const shouldStamp = computed(() => {
  return props.category === 'concert' && props.venueSlug === 'nippon-budokan'
})

function isStamped(i) {
  // 簡化邏輯：只有第一格顯示
  return shouldStamp.value && i === 1
}
</script>

<style>
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
  display:flex; align-items:center; justify-content:center;
  background: #fafafa;
}
.placeholder { color: #bbb; font-size: 20px; }
.stamp img { width: 90%; object-fit: contain; filter: grayscale(100%); }
</style>
