<!-- 动态组件演示用：新闻 Tab（8.19） -->
<script setup lang="ts">
import { ref } from 'vue'

const keyword = ref('')
</script>

<template>
  <div class="tab-page">
    <h3>📰 新闻 Tab</h3>
    <input v-model="keyword" placeholder="搜点什么…（复习 8.15 的 v-model）" />
    <p>你输入了：<b>{{ keyword || '（空）' }}</b> —— 切走再切回来，输入会被清空</p>
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

input {
  border: 1px solid #e5e5e5;
  border-radius: 6px;
  padding: 4px 10px;
  font-size: 0.9rem;
}
</style>
