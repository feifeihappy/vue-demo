<!--
  默认插槽演示组件（8.19）
  props 传标题（复习 8.18），标签中间的内容走"默认插槽"
-->
<script setup lang="ts">
defineProps<{ title: string }>()
</script>

<template>
  <div class="info-card">
    <h3 class="card-title">{{ title }}</h3>
    <!-- 默认插槽：父组件写在 <InfoCard> 标签中间的内容都会出现在这里 -->
    <div class="card-body">
      <slot />
    </div>
  </div>
</template>

<style scoped>
.info-card {
  border: 1px solid #e5e5e5;
  border-radius: 10px;
  padding: 1rem 1.25rem;
  background: #fff;
}

.card-title {
  margin: 0 0 0.75rem;
  color: #e8590c;
}

.card-body {
  color: #444;
  line-height: 1.7;
}
</style>
