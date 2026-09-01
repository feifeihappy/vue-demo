# 第9天 · 插槽与动态组件（学习笔记）

> 日期：2026-08-31（补 8.19 第9天内容）· 学习计划第 2 周周三

## 📑 目录

- [✅ 今日完成](#-今日完成)
- [一、默认插槽](#一默认插槽组件挖洞父组件填内容)
- [二、具名插槽](#二具名插槽一个组件挖多个洞)
- [三、作用域插槽](#三作用域插槽把子组件的数据递给父组件渲染)
- [四、动态组件 component :is](#四动态组件-component-is-一键换组件)
- [📌 小作业](#-小作业20-分钟巩固今天内容)

## ✅ 今日完成

- 创建演示组件：`src/components/SlotDemo.vue`（主演示）+ 5 个子组件，替换进 App.vue
  - `InfoCard.vue` —— 默认插槽
  - `ArticleLayout.vue` —— 具名插槽（#title / #content / #footer）
  - `SlotList.vue` —— 作用域插槽（数据递回给父组件渲染）
  - `TabHome.vue` + `TabNews.vue` —— 动态组件 `<component :is>` 的两个 tab 页
- 完成了计划任务：**默认/具名/作用域插槽，`<component :is>`**

## 一、默认插槽：组件挖洞，父组件填内容

```vue
<!-- 父组件：标签中间的内容 = 默认插槽的内容 -->
<InfoCard title="什么是插槽？">
  <p>这句话是父组件塞进来的</p>
</InfoCard>

<!-- 子组件 InfoCard：洞在这里 -->
<div class="card-body"><slot /></div>
```

- 插槽 = 组件挖的"洞"，**父组件往洞里填内容**
- 8.18 的 props 只能传"数据"，插槽让父组件连"结构"都能决定 → 组件从"写死的卡片"变成"通用容器"

## 二、具名插槽：一个组件挖多个洞

```vue
<!-- 父组件：指名道姓地往洞里填 -->
<ArticleLayout>
  <template #title>标题</template>
  <template #content>正文</template>
  <template #footer>页脚（不填则显示子组件默认内容）</template>
</ArticleLayout>

<!-- 子组件：每个洞都有自己的 name；没写 name 的就是默认洞 -->
<slot name="title" />
<slot name="content" />
<slot name="footer">（默认页脚）</slot>
```

- `#title` 是 `v-slot:title` 的缩写
- 同时用默认洞 + 具名洞时，默认洞写成 `#default`
- 没被填充的洞显示子组件的默认内容

## 三、作用域插槽：把子组件的数据"递"给父组件渲染

```vue
<!-- 子组件 SlotList：数据是自己的，通过插槽属性递出去 -->
<li v-for="(item, index) in items" :key="item">
  <slot :item="item" :index="index">{{ index + 1 }}. {{ item }}</slot>
</li>

<!-- 父组件：解构接收，自己定渲染 -->
<SlotList>
  <template #default="{ item, index }">
    <b>{{ index }}</b> {{ item }}（父组件自定义样式）
  </template>
</SlotList>
```

- 方向对比：**props 是父传子，作用域插槽是子传父**（顺着插槽回流）
- 作用：列表组件提供"数据 + 布局"，每项长什么样由使用方决定 → 组件才真正通用
- 父组件不接管时，显示子组件写在 `<slot>` 标签里的默认内容

## 四、动态组件 `<component :is>`：一键换组件

```vue
<script setup>
import TabHome from './TabHome.vue'
import TabNews from './TabNews.vue'

// is 的值是"组件对象"（不是字符串！）
const currentTab = ref(TabHome)
function switchTab(tab) {
  currentTab.value = tab
}
</script>

<template>
  <button @click="switchTab(TabHome)">🏠 首页</button>
  <button @click="switchTab(TabNews)">📰 新闻</button>
  <component :is="currentTab" />
</template>
```

- 一个位置写死，**渲染谁由变量说了算**——tab 切换、页面跳转都靠它
- 对比 v-if：v-if 写死"渲染哪个组件"；`:is` 把选择权交给变量
- ⚠️ 现象（demo 里亲手试）：切走再切回，tab 里的计数器/输入框状态**没了**——动态组件切换 = 旧组件销毁、新组件创建
- 想保留状态 → 用 `<KeepAlive>` 包一层（可作延伸练习）
- 进阶：存组件对象用 `shallowRef` 更优（组件对象不需要深层响应式）

## 📌 小作业（20 分钟，巩固今天内容）

1. 在浏览器里：给首页 Tab 计数器 +1、新闻 Tab 输入文字，切走再切回来，观察状态丢失现象。
2. 给 `ArticleLayout` 再挖一个 `#aside` 洞，父组件填一段"侧边栏"内容（不填时显示默认提示）。
3. 给 `SlotList` 的第二种用法改成"隔行变色"：`index` 是偶数显示浅橙色背景。
4. 思考题：为什么作用域插槽的方向和 props 相反？如果子组件不递数据，父组件怎么能自定义列表项？
5. 延伸题（做了更好）：用 `<KeepAlive>` 包住 `<component :is>`，验证 tab 状态是否保留。
