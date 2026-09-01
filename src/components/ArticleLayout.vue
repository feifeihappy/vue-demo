<!--
  具名插槽演示组件（8.19）
  一个布局组件挖了三个"洞"：title / content / footer，
  父组件用 <template #xxx> 分别填充
-->
<template>
  <article class="article">
    <header class="article-header">
      <!-- name="title" 的具名插槽：父组件用 <template #title> 填充；不填就显示默认文字 -->
      <slot name="title">（默认标题）</slot>
    </header>
    <main class="article-body">
      <slot name="content" />
    </main>
    <footer class="article-footer">
      <slot name="footer">（默认页脚）</slot>
    </footer>
  </article>
</template>

<style scoped>
.article {
  border: 1px solid #e5e5e5;
  border-radius: 10px;
  overflow: hidden;
  background: #fff;
}

.article-header {
  background: #fff4e6;
  padding: 0.75rem 1.25rem;
  font-weight: bold;
  border-bottom: 1px solid #f1d9bd;
}

.article-body {
  padding: 1rem 1.25rem;
  line-height: 1.7;
  color: #444;
}

.article-footer {
  background: #f6f8fa;
  padding: 0.5rem 1.25rem;
  font-size: 0.85rem;
  color: #888;
  border-top: 1px solid #eee;
}
</style>
