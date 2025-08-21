<template>
  <div class="passport-container">
    <div class="book">
      <!-- 書本封面 -->
      <div class="page cover" :style="pageStyle(-1)" @click="openBook">
        <slot name="cover">
          <h2>My Show Passport</h2>
        </slot>
      </div>
      <!-- 動態產生頁面並依序排列，翻頁時依 currentPage 決定旋轉角度 -->
      <div
        v-for="(page, index) in pages"
        :key="index"
        class="page"
        :class="'page-' + (index + 1)"
        :style="pageStyle(index)"
      >
        <!-- 預留插槽顯示頁面內容，例如 StampGrid -->
        <slot :name="'page-' + (index + 1)" :page="page">
          <div class="page-content">{{ page }}</div>
        </slot>
      </div>
    </div>
    <!-- 翻頁控制按鈕 -->
    <div class="nav">
      <button class="prev" @click="prevPage" :disabled="currentPage === 0">Prev</button>
      <button class="next" @click="nextPage" :disabled="currentPage >= pages.length - 1">Next</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// 接收 pages 陣列，內容可以是任何用於插槽的資料
const props = defineProps({ pages: { type: Array, required: true } })

// 目前翻到第幾頁
const currentPage = ref(0)
// 開啟書本的旗標（目前未用於控制動畫，可擴充）
const bookOpened = ref(false)

// 啟動打開書本
function openBook() {
  bookOpened.value = true
}

// 下一頁
function nextPage() {
  if (currentPage.value < props.pages.length - 1) currentPage.value++
}

// 上一頁
function prevPage() {
  if (currentPage.value > 0) currentPage.value--
}

/*
 * 根據頁面索引計算 CSS transform：
 * - 封面 index = -1；翻開封面後固定於左側
 * - 若索引 < currentPage：頁面已被翻過，旋轉 -180deg
 * - 若索引 >= currentPage：頁面保持 0 度
 */
function pageStyle(index) {
  // 封面處理
  if (index < 0) {
    return {
      transform: bookOpened.value
        ? 'translateX(-4px) rotateY(-170deg)'
        : 'rotateY(0deg)',
      zIndex: props.pages.length + 2,
    }
  }
  const offsetX = index * 2
  const rotated = index < currentPage.value
  return {
    transform: rotated
      ? `translateX(${offsetX}px) rotateY(-180deg)`
      : `translateX(${offsetX}px) rotateY(0deg)`,
    zIndex: props.pages.length - index,
  }
}
</script>

<style scoped>
/* 容器使用 perspective 讓內部 3D 旋轉具有深度 */
.passport-container {
  width: 100%;
  max-width: 600px;
  margin: 20px auto;
  perspective: 800px;
}

.book {
  position: relative;
  width: 100%;
  height: 350px;
  transform-style: preserve-3d;
  transition: transform 1s;
}

.page {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #fff;
  border: 1px solid #ddd;
  border-radius: 6px;
  backface-visibility: hidden;
  transform-style: preserve-3d;
  transition: transform 1s, z-index 0s;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
}

.cover {
  background: #f0f0f0;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  z-index: 999;
  font-weight: bold;
}

.page-content {
  padding: 20px;
  text-align: center;
}

.nav {
  margin-top: 16px;
  display: flex;
  justify-content: space-between;
}

.prev,
.next {
  padding: 6px 14px;
  border: 1px solid #ccc;
  border-radius: 4px;
  background: #fafafa;
  cursor: pointer;
}
.prev:disabled,
.next:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>