<!-- 动态组件演示用：首页 Tab（8.19） -->
<script setup lang="ts">
import { ref } from 'vue'

const count = ref(0)
</script>

<template>
  <div class="tab-page">
    <h3>🏠 首页 Tab</h3>
    <p>计数器：<b>{{ count }}</b> —— 切走再切回来，计数会归零（组件被销毁重建了）</p>
    <button @click="count++">+1</button>
  </div>
</template>

<style scoped>
.tab-page {
  background: #fff;
  border: 1px solid #e5e5e5;
  border-radius: 10px;
  padding: 1rem 1.25rem;
}

.tab-page h3 {
  margin: 0 0 0.5rem;
  color: #e8590c;
}

button {
  padding: 4px 12px;
  border: 1px solid #e8590c;
  border-radius: 6px;
  background: #fff;
  color: #e8590c;
  cursor: pointer;
}

button:hover {
  background: #e8590c;
  color: #fff;
}
</style>
