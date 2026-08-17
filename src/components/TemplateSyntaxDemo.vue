<!--
  8.12 学习主题：模板语法与插值
  演示内容：
    一、{{ }} 插值 + 表达式 + 方法调用
    二、v-html（附安全警告）
    三、10 种动态 class / style 写法（每种都配了交互按钮，点击观察视图变化）
-->
<script setup lang="ts">
import { ref, computed } from 'vue'

/* ============ 一、插值与表达式 ============ */
const message = ref('Hello Vue 3!')
const price = ref(9.9)
const count = ref(3)

// 模板里可以调用方法（放纯展示逻辑，副作用逻辑放 script）
function formatPrice(n: number): string {
  return '¥' + n.toFixed(2)
}

// v-html 内容：业务代码里要谨慎使用，有 XSS 注入风险（见文末警告）
const htmlContent = ref('<strong style="color:#42b883">Vue</strong> 通过 v-html 渲染真实 HTML')

/* ============ 二、10 种动态 class / style ============ */

// ① 对象语法：布尔值决定类名开关
const isActive = ref(true)

// ② 对象语法：多个类名，各自独立控制
const isDanger = ref(false)

// ③ 数组语法：类名来自变量（空字符串 = 不渲染）
const activeClass = ref('highlight')
const errorClass = ref('')

// ④ 数组 + 对象混合
const isBold = ref(true)
const extraClass = ref('shadow-box')

// ⑤ 三元表达式：二选一
const option = ref<'A' | 'B'>('A')

// ⑥ 动态 style：颜色 / 字号随状态变化
const textColor = ref('#2c7be5')
const fontSize = ref(16)

// ⑦ style 数组：合并多个样式对象
const baseStyle = { fontFamily: 'Consolas, monospace' }
const boxBg = ref('#eef6ff')

// ⑧ CSS 变量：把 --xxx 传给子元素（CSS 里用 var(--xxx) 取用）
const themeColor = ref('#42b883')

// ⑨ computed 驱动：根据业务数据算出最终类名
const score = ref(75)
const scoreClass = computed(() => {
  if (score.value >= 90) return 'score-good'
  if (score.value >= 60) return 'score-pass'
  return 'score-fail'
})

// ⑩ v-bind 绑定普通属性（v-bind 不止能绑 class/style）
const isDisabled = ref(false)
const tip = ref('我是动态 title 提示，点击按钮会改变')
</script>

<template>
  <div class="page">
    <h1>📐 8.12 模板语法与插值</h1>
    <p class="subtitle">插值 / 表达式 / v-bind / v-html · 10 种动态 class &amp; style</p>

    <!-- ================= 一、插值与表达式 ================= -->
    <section class="section">
      <h2>一、插值与表达式 <code class="inline" v-pre>{{ }}</code></h2>

      <div class="card">
        <h3>1. 文本插值（ref 在模板中自动解包，不用写 .value）</h3>
        <p class="result"><b>{{ message }}</b></p>
        <code v-pre>&lt;p&gt;{{ message }}&lt;/p&gt;</code>
      </div>

      <div class="card">
        <h3>2. 表达式：支持任意 JS 表达式运算</h3>
        <p class="result">
          {{ price }} × {{ count }} = <b>{{ price * count }}</b>
        </p>
        <code v-pre>&lt;p&gt;{{ price }} × {{ count }} = {{ price * count }}&lt;/p&gt;</code>
      </div>

      <div class="card">
        <h3>3. 方法调用：模板里可以调用 script 中的函数</h3>
        <p class="result">合计：<b>{{ formatPrice(price * count) }}</b></p>
        <code v-pre>&lt;p&gt;{{ formatPrice(price * count) }}&lt;/p&gt;</code>
      </div>

      <div class="card">
        <h3>4. v-html：渲染真实 HTML</h3>
        <p class="result" v-html="htmlContent"></p>
        <code v-pre>&lt;p v-html="htmlContent"&gt;&lt;/p&gt;</code>
        <p class="warning">
          ⚠️ 注意：v-html 会把内容当 HTML 解析，永远不要用它渲染用户输入的内容（XSS 注入风险）。
        </p>
      </div>
    </section>

    <!-- ================= 二、10 种动态 class / style ================= -->
    <section class="section">
      <h2>二、10 种动态 class / style 写法</h2>

      <div class="card">
        <h3>① 对象语法（最常用）：布尔值控制类名开关</h3>
        <p class="result" :class="{ active: isActive }">点按钮试试：class 在 active / 普通 之间切换</p>
        <button @click="isActive = !isActive">切换 isActive：{{ isActive }}</button>
        <code v-pre>:class="{ active: isActive }"</code>
      </div>

      <div class="card">
        <h3>② 对象语法：多个类名，各自独立控制</h3>
        <p class="result" :class="{ active: isActive, 'text-danger': isDanger }">同时受两个状态控制</p>
        <button @click="isActive = !isActive">isActive：{{ isActive }}</button>
        <button @click="isDanger = !isDanger">isDanger：{{ isDanger }}</button>
        <code v-pre>:class="{ active: isActive, 'text-danger': isDanger }"</code>
      </div>

      <div class="card">
        <h3>③ 数组语法：类名来自变量（空字符串不渲染）</h3>
        <p class="result" :class="[activeClass, errorClass]">数组里的两个类名</p>
        <button @click="activeClass = activeClass === 'highlight' ? '' : 'highlight'">toggle highlight</button>
        <button @click="errorClass = errorClass === 'text-danger' ? '' : 'text-danger'">toggle danger</button>
        <code v-pre>:class="[activeClass, errorClass]"</code>
      </div>

      <div class="card">
        <h3>④ 数组 + 对象混合：既有固定类，又有条件类</h3>
        <p class="result" :class="['box', { 'text-bold': isBold }, extraClass]">三个来源组合</p>
        <button @click="isBold = !isBold">加粗：{{ isBold }}</button>
        <button @click="extraClass = extraClass === 'shadow-box' ? '' : 'shadow-box'">阴影</button>
        <code v-pre>:class="['box', { 'text-bold': isBold }, extraClass]"</code>
      </div>

      <div class="card">
        <h3>⑤ 三元表达式：条件二选一</h3>
        <p class="result" :class="option === 'A' ? 'pill-a' : 'pill-b'">当前选项：{{ option }}</p>
        <button @click="option = option === 'A' ? 'B' : 'A'">切换 A / B</button>
        <code v-pre>:class="option === 'A' ? 'pill-a' : 'pill-b'"</code>
      </div>

      <div class="card">
        <h3>⑥ 动态 style 对象：颜色、字号都跟着响应式数据走</h3>
        <p class="result" :style="{ color: textColor, fontSize: fontSize + 'px' }">我是动态样式的文本</p>
        <button @click="textColor = textColor === '#2c7be5' ? '#e05252' : '#2c7be5'">换颜色</button>
        <button @click="fontSize = fontSize >= 24 ? 16 : fontSize + 2">字号 +2（当前 {{ fontSize }}px）</button>
        <code v-pre>:style="{ color: textColor, fontSize: fontSize + 'px' }"</code>
      </div>

      <div class="card">
        <h3>⑦ style 数组：合并多个样式对象，后者覆盖前者</h3>
        <p class="result" :style="[baseStyle, { backgroundColor: boxBg }]">数组里两个样式对象</p>
        <button @click="boxBg = boxBg === '#eef6ff' ? '#fff3e6' : '#eef6ff'">换背景</button>
        <code v-pre>:style="[baseStyle, { backgroundColor: boxBg }]"</code>
      </div>

      <div class="card">
        <h3>⑧ CSS 变量：把动态值通过 --xxx 传给 CSS 使用</h3>
        <div class="ball" :style="{ '--theme-color': themeColor }"></div>
        <button @click="themeColor = themeColor === '#42b883' ? '#7c4dff' : '#42b883'">换主题色</button>
        <code v-pre>:style="{ '--theme-color': themeColor }"</code>
        <code v-pre>/* CSS 里用：background: var(--theme-color) */</code>
      </div>

      <div class="card">
        <h3>⑨ computed 驱动：业务数据 → 自动算出类名（8.13 会细讲）</h3>
        <p class="result" :class="scoreClass">分数：{{ score }} 分</p>
        <button @click="score = score >= 90 ? 55 : score + 10">加分 +10</button>
        <code v-pre>:class="scoreClass"</code>
      </div>

      <div class="card">
        <h3>⑩ v-bind 绑定普通属性：disabled / title / placeholder…</h3>
        <input
          :disabled="isDisabled"
          :placeholder="isDisabled ? '输入框已禁用' : '点右边按钮禁用我'"
        />
        <button :title="tip">鼠标悬停看我提示</button>
        <button @click="isDisabled = !isDisabled">禁用输入框：{{ isDisabled }}</button>
        <code v-pre>:disabled="isDisabled" :placeholder="..."</code>
      </div>
    </section>

    <p class="footer">💡 记忆口诀：<code v-pre>:class</code> 见对象（开关）见数组（拼接），<code v-pre>:style</code> 驼峰写属性，CSS 变量走 <code v-pre>--xx</code>。</p>
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
  border-left: 4px solid #42b883;
  padding-left: 0.75rem;
}

.section h2 code.inline {
  display: inline;
  background: transparent;
  padding: 0;
  margin: 0;
  color: #42b883;
}

.card {
  border: 1px solid #e5e5e5;
  border-radius: 10px;
  padding: 1rem 1.25rem;
  margin-bottom: 1rem;
  background: #fff;
}

.card h3 {
  margin: 0 0 0.75rem;
  font-size: 0.95rem;
  color: #444;
}

.result {
  font-size: 1.05rem;
  margin: 0.5rem 0;
}

/* ① ② 用到的类 */
.active {
  background: #42b883;
  color: #fff;
  border-radius: 6px;
  padding: 4px 10px;
}
.text-danger {
  background: #e05252;
  color: #fff;
  border-radius: 6px;
  padding: 4px 10px;
}
.highlight {
  background: #ffe9a8;
  border-radius: 6px;
  padding: 4px 10px;
}

/* ③ ④ 用到的类 */
.box {
  border: 2px dashed #ccc;
  border-radius: 6px;
  padding: 6px 12px;
}
.text-bold {
  font-weight: 700;
}
.shadow-box {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

/* ⑤ 用到的类 */
.pill-a,
.pill-b {
  display: inline-block;
  border-radius: 999px;
  padding: 4px 14px;
  color: #fff;
}
.pill-a {
  background: #7c4dff;
}
.pill-b {
  background: #ff7a45;
}

/* ⑧ CSS 变量 */
.ball {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  background: var(--theme-color);
  transition: background 0.3s;
  margin: 0.5rem 0;
}

/* ⑨ 用到的类 */
.score-good {
  color: #16a34a;
  font-weight: 700;
}
.score-pass {
  color: #d97706;
  font-weight: 700;
}
.score-fail {
  color: #dc2626;
  font-weight: 700;
}

button {
  margin: 0 6px 6px 0;
  padding: 4px 12px;
  border: 1px solid #42b883;
  border-radius: 6px;
  background: #fff;
  color: #42b883;
  cursor: pointer;
}
button:hover {
  background: #42b883;
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

.footer {
  margin-top: 2rem;
  color: #666;
  background: #f0f7ff;
  border-radius: 8px;
  padding: 0.75rem 1rem;
}
.footer code {
  display: inline;
  background: transparent;
  padding: 0;
  margin: 0;
  font-size: 0.9rem;
}
</style>
