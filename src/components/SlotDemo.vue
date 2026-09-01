<!--
  8.19 插槽与动态组件
  演示：
    一、默认插槽：InfoCard 挖的"洞"，父组件往洞里填内容
    二、具名插槽：ArticleLayout 三个洞 #title / #content / #footer
    三、作用域插槽：SlotList 数据在子、渲染由父定（子 → 插槽属性 → 父）
    四、动态组件：<component :is> 切换 TabHome / TabNews
-->
<script setup lang="ts">
import { ref } from 'vue'
import InfoCard from './InfoCard.vue'
import ArticleLayout from './ArticleLayout.vue'
import SlotList from './SlotList.vue'
import TabHome from './TabHome.vue'
import TabNews from './TabNews.vue'

// 四、动态组件：currentTab 存"组件对象"，点按钮换它，<component :is> 就渲染对应组件
const currentTab = ref<typeof TabHome | typeof TabNews>(TabHome)

function switchTab(tab: typeof TabHome | typeof TabNews) {
  currentTab.value = tab
}
</script>

<template>
  <div class="page">
    <h1>🧱 8.19 插槽与动态组件</h1>
    <p class="subtitle">插槽 = 组件挖的"洞"，父组件往里填内容 · &lt;component :is&gt; = 动态切换组件</p>

    <!-- ============ 一、默认插槽 ============ -->
    <section class="section">
      <h2>一、默认插槽：组件挖洞，父组件填内容</h2>
      <div class="card">
        <InfoCard title="什么是插槽？">
          <p>组件标签<b>中间</b>写的内容，会渲染到子组件的 <code>&lt;slot /&gt;</code> 位置。</p>
          <p>比如这句话就不是 InfoCard 自己写的，是父组件塞进来的。</p>
        </InfoCard>
        <code v-pre>&lt;InfoCard title="什么是插槽？"&gt;
  &lt;p&gt;这句话是父组件塞进来的&lt;/p&gt;   &lt;!-- 默认插槽的内容 --&gt;
&lt;/InfoCard&gt;</code>
        <code v-pre>&lt;div class="card-body"&gt;&lt;slot /&gt;&lt;/div&gt;   &lt;!-- 子组件：洞在这里 --&gt;</code>
        <p class="tip">💡 插槽解决了 8.18 的痛点：父组件只用 props 传数据的话，子组件内部长什么样完全写死。有了插槽，<b>结构也能由父组件决定</b>。</p>
      </div>
    </section>

    <!-- ============ 二、具名插槽 ============ -->
    <section class="section">
      <h2>二、具名插槽：一个组件挖多个洞</h2>
      <div class="card">
        <ArticleLayout>
          <template #title>📖 今天学到了什么？</template>
          <template #content>
            <p>具名插槽用 <b>name</b> 区分不同的洞。父组件用 <code>&lt;template #title&gt;</code> 指名道姓地往对应洞里填。</p>
            <p>没被填充的洞显示子组件的默认内容（比如这里的页脚）。</p>
          </template>
          <template #footer>作者：前端小学生 · 8.19</template>
        </ArticleLayout>
        <code v-pre>&lt;ArticleLayout&gt;
  &lt;template #title&gt;...&lt;/template&gt;     &lt;!-- 填 title 洞 --&gt;
  &lt;template #content&gt;...&lt;/template&gt;   &lt;!-- 填 content 洞 --&gt;
  &lt;template #footer&gt;...&lt;/template&gt;     &lt;!-- 填 footer 洞（不填则显示默认） --&gt;
&lt;/ArticleLayout&gt;</code>
        <code v-pre>&lt;slot name="title" /&gt;    &lt;!-- 子组件：具名洞；不写 name 的就是默认洞 --&gt;</code>
        <p class="hint">#title 是 v-slot:title 的缩写 · 想同时用默认洞 + 具名洞时，默认洞写成 #default</p>
      </div>
    </section>

    <!-- ============ 三、作用域插槽 ============ -->
    <section class="section">
      <h2>三、作用域插槽：把子组件的数据"递"给父组件渲染</h2>
      <div class="card">
        <p class="result">① 父组件不接管，用子组件的默认渲染：</p>
        <SlotList />
        <p class="result">② 父组件接管：用 <code>#default="{ item, index }"</code> 接收数据，自己定渲染：</p>
        <SlotList>
          <template #default="{ item, index }">
            <span class="badge">{{ index }}</span> {{ item }}（父组件自定义样式）
          </template>
        </SlotList>
        <code v-pre>&lt;li v-for="(item, index) in items" :key="item"&gt;
  &lt;slot :item="item" :index="index"&gt;{{ index + 1 }}. {{ item }}&lt;/slot&gt;
&lt;/li&gt;    &lt;!-- 子组件：数据通过插槽属性递出去，不接就显示默认内容 --&gt;</code>
        <code v-pre>&lt;template #default="{ item, index }"&gt;...&lt;/template&gt;   &lt;!-- 父组件：解构接收 --&gt;</code>
        <p class="tip">💡 关键区别：props 是<b>父传子</b>，作用域插槽是<b>子传父</b>（顺着插槽回流）——列表组件提供数据与布局，每项长什么样由使用它的父组件决定，组件就真正"通用"了。</p>
      </div>
    </section>

    <!-- ============ 四、动态组件 ============ -->
    <section class="section">
      <h2>四、动态组件：&lt;component :is&gt; 一键换组件</h2>
      <div class="card">
        <p class="result">点按钮切换 currentTab，同一个位置渲染不同的组件：</p>
        <div class="tabs">
          <button :class="{ active: currentTab === TabHome }" @click="switchTab(TabHome)">🏠 首页</button>
          <button :class="{ active: currentTab === TabNews }" @click="switchTab(TabNews)">📰 新闻</button>
        </div>
        <!-- is 的值是"组件对象"，换一个对象，这里就渲染成那个组件 -->
        <component :is="currentTab" />
        <code v-pre>const currentTab = ref(TabHome)   // 存组件对象（不是字符串！）</code>
        <code v-pre>&lt;component :is="currentTab" /&gt;   // 一处写死，渲染谁由变量决定</code>
        <p class="warning">⚠️ 试试：给首页计数器 +1 或新闻 Tab 输入文字，切走再切回来——状态<b>没了</b>！因为动态组件切换 = 旧组件被销毁、新组件被创建。想保留状态，需要 &lt;KeepAlive&gt; 包一层（明天/后续可以试）。</p>
        <p class="hint">对比 v-if：v-if 写死"渲染哪个组件"，component :is 把选择权交给变量——tab 切换、页面跳转都靠它</p>
      </div>
    </section>

    <!-- ============ 总结 ============ -->
    <section class="section">
      <h2>五、一句话总结</h2>
      <div class="card">
        <pre class="flow">插槽三种：
  默认插槽 &lt;slot /&gt;              → 父组件往中间填内容（填一个洞）
  具名插槽 &lt;slot name="x" /&gt;     → 父组件 &lt;template #x&gt; 指名道姓填（填多个洞）
  作用域插槽 &lt;slot :数据 /&gt;       → 子组件把数据递回给父组件渲染（子 → 父）

动态组件 &lt;component :is="变量" /&gt; → 渲染哪个组件由变量说了算</pre>
        <p class="footer">📌 口诀：<b>插槽是"洞"，父组件往里填；作用域插槽是"递数据"，子组件递、父组件接；component :is 是"遥控器"，切谁渲染谁。</b></p>
      </div>
    </section>
  </div>
</template>

<style scoped>
.page {
  max-width: 900px;
  margin: 0 auto;
  padding: 2rem 1.5rem 4rem;
}

h1 {
  color: var(--vt-c-text-1, #213547);
  margin-bottom: 0.25rem;
}

.subtitle {
  color: #888;
  margin-top: 0;
}

.section {
  margin-top: 2rem;
}

.section h2 {
  border-left: 4px solid #e8590c;
  padding-left: 0.75rem;
}

.card {
  border: 1px solid #e5e5e5;
  border-radius: 10px;
  padding: 1rem 1.25rem;
  margin-bottom: 1rem;
  background: #fff;
}

.result {
  font-size: 1.05rem;
  margin: 0.5rem 0;
}

button {
  margin: 0 6px 6px 0;
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

button.active {
  background: #e8590c;
  color: #fff;
}

code {
  display: block;
  background: #f6f8fa;
  border-radius: 6px;
  padding: 6px 10px;
  margin-top: 0.5rem;
  font-size: 0.85rem;
  color: #333;
  white-space: pre-wrap;
}

.tip {
  background: #f0f7ff;
  border: 1px solid #bcd9f7;
  border-radius: 6px;
  padding: 8px 12px;
  font-size: 0.85rem;
  color: #1a4b7a;
  margin-top: 0.5rem;
}

.warning {
  background: #fff7e6;
  border: 1px solid #ffd591;
  border-radius: 6px;
  padding: 8px 12px;
  font-size: 0.85rem;
  color: #874d00;
  margin-top: 0.5rem;
}

.hint {
  font-size: 0.85rem;
  color: #888;
  margin: 0.5rem 0 0;
}

.badge {
  display: inline-block;
  min-width: 22px;
  text-align: center;
  background: #fff4e6;
  color: #e8590c;
  border-radius: 4px;
  padding: 0 4px;
  font-size: 0.8rem;
  font-weight: bold;
  margin-right: 6px;
}

.tabs {
  display: flex;
  gap: 8px;
  margin-bottom: 12px;
}

.flow {
  background: #f6f8fa;
  border-radius: 8px;
  padding: 12px;
  font-size: 0.9rem;
  line-height: 1.7;
  overflow-x: auto;
  margin: 0;
}

.footer {
  margin-top: 2rem;
  color: #666;
  background: #fff4e6;
  border-radius: 8px;
  padding: 0.75rem 1rem;
}
</style>
