<!--
  作用域插槽演示组件（8.19）
  数据归"子组件"所有，渲染由"父组件"决定：
  子组件把 item / index 通过插槽属性"递"给父组件
-->
<script setup lang="ts">
// 这次数据不在父组件，而是子组件自己的
const items = ['🍎 苹果', '🍌 香蕉', '🍇 葡萄', '🍉 西瓜']
</script>

<template>
  <ul class="slot-list">
    <li v-for="(item, index) in items" :key="item">
      <!--
        作用域插槽：把 item 和 index 递给父组件（插槽属性）
        父组件不接（不写 <template #default>）时，显示这里的默认内容兜底
      -->
      <slot :item="item" :index="index">{{ index + 1 }}. {{ item }}</slot>
    </li>
  </ul>
</template>

<style scoped>
.slot-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.slot-list li {
  padding: 6px 10px;
  border-bottom: 1px dashed #eee;
  font-size: 0.95rem;
}
</style>
