<!--
  8.13 学习主题：响应式核心 ref / reactive
  演示内容：
    一、ref vs reactive 基础对比
    二、ref 包对象（深层响应式）
    三、computed 派生值（自动缓存）
    四、watch 监听变化
    五、陷阱剧场：解构丢响应性 / 整体替换丢响应性
-->
<script setup lang="ts">
import { ref, reactive, computed, watch, toRefs } from 'vue'

/* ============ 一、ref vs reactive 基础对比 ============ */
const count = ref(0) // ref：包基本类型，script 里要 .value
const user = reactive({ name: 'Vue', age: 3 }) // reactive：包对象，直接 .属性

function addCount() {
  count.value += 1 // script 里访问 ref 必须 .value
}

function addAge() {
  user.age += 1 // reactive 直接点属性
}

/* ============ 二、ref 包对象：也是深层响应式的 ============ */
// 注意：ref 里放对象时，内部其实用的就是 reactive 的代理
const cart = ref({ items: 2, total: 100 })

function addItem() {
  cart.value.items += 1 // 两层：先 .value 拿到对象，再 .items
  cart.value.total += 20
}

/* ============ 三、computed：派生值，依赖变化自动重算 ============ */
const price = ref(9.9)
const quantity = ref(3)

// computed 里必须通过 .value 读依赖；它自身在模板里自动解包
const totalPrice = computed(() => price.value * quantity.value)

/* ============ 四、watch：监听变化，执行副作用（记日志） ============ */
const history = ref<string[]>([])

watch(totalPrice, (newVal, oldVal) => {
  history.value.unshift(`总价：¥${oldVal} → ¥${newVal}`)
})

/* ============ 五、陷阱剧场 ============ */
const settings = reactive({ theme: 'light' })

// 陷阱 1：直接解构 —— theme 只是普通字符串副本，改了视图不更新
let { theme } = settings
function changeByDestructure() {
  theme = 'dark'
}

// 正确做法：toRefs 解构 —— 解出来的是 ref，改 .value 仍响应
const { theme: themeRef } = toRefs(settings)
function changeByToRefs() {
  themeRef.value = 'dark'
}

// 陷阱 2：整体替换 —— 新对象不是代理，视图不再更新
let settings2 = reactive({ theme: 'light' })
function replaceSettings() {
  settings2 = { theme: 'dark' }
}

// 正确做法：不整体替换，只改属性
function changeProperty() {
  settings2.theme = 'dark'
}
</script>

<template>
  <div class="page">
    <h1>⚡ 8.13 响应式核心：ref / reactive</h1>
    <p class="subtitle">ref · reactive · computed · watch · 数据驱动视图</p>

    <!-- ============ 一、基础对比 ============ -->
    <section class="section">
      <h2>一、ref vs reactive 基础对比</h2>

      <div class="card">
        <h3>ref(0)：script 里必须 .value，模板里自动解包</h3>
        <p class="result">count = <b>{{ count }}</b></p>
        <button @click="addCount">count.value += 1</button>
        <code v-pre>const count = ref(0)</code>
        <code v-pre>count.value += 1  // script 里</code>
        <code v-pre>{{ count }}      // 模板里不用 .value</code>
      </div>

      <div class="card">
        <h3>reactive({...})：script 里直接 .age</h3>
        <p class="result">{{ user.name }}，age = <b>{{ user.age }}</b></p>
        <button @click="addAge">user.age += 1</button>
        <code v-pre>const user = reactive({ name: 'Vue', age: 3 })</code>
        <code v-pre>user.age += 1  // 直接点属性</code>
      </div>
    </section>

    <!-- ============ 二、ref 包对象 ============ -->
    <section class="section">
      <h2>二、ref 包对象：也是深层响应式的</h2>

      <div class="card">
        <h3>cart = ref({ items, total })：对象字段变化照样触发更新</h3>
        <p class="result">商品 {{ cart.items }} 件，金额 <b>¥{{ cart.total }}</b></p>
        <button @click="addItem">添加一件商品</button>
        <code v-pre>cart.value.items += 1  // 先 .value 再 .items</code>
        <p class="tip">💡 ref 包对象时，内部其实是 reactive 代理，只是外面多一层 .value 的壳——所以官方推荐默认用 ref，它更统一。</p>
      </div>
    </section>

    <!-- ============ 三、computed ============ -->
    <section class="section">
      <h2>三、computed：派生值，依赖一变就重算</h2>

      <div class="card">
        <h3>单价 × 数量 = 总价（只读，谁也别想直接改它）</h3>
        <p class="result">
          ¥{{ price }} × {{ quantity }} 件 = <b>¥{{ totalPrice }}</b>
        </p>
        <button @click="quantity++">数量 +1（改依赖）</button>
        <button @click="price = +(price - 1).toFixed(1)">单价 -1（改依赖）</button>
        <code v-pre>const totalPrice = computed(() =&gt; price.value * quantity.value)</code>
        <p class="tip">💡 computed 有缓存：依赖没变，多次读取不会重复计算。适合"由已有数据算出来的值"。</p>
      </div>
    </section>

    <!-- ============ 四、watch ============ -->
    <section class="section">
      <h2>四、watch：监视变化，执行副作用</h2>

      <div class="card">
        <h3>watch(totalPrice, ...)：总价一变，记录日志（computed 也可以被 watch！）</h3>
        <p class="result">当前总价：<b>¥{{ totalPrice }}</b></p>
        <button @click="quantity++">数量 +1（触发 watch）</button>
        <ul class="log">
          <li v-for="(h, i) in history" :key="i">{{ h }}</li>
        </ul>
        <code v-pre>watch(totalPrice, (newVal, oldVal) =&gt; { ... })</code>
      </div>
    </section>

    <!-- ============ 五、陷阱剧场 ============ -->
    <section class="section">
      <h2>五、陷阱剧场 ⚠️（改错题考点）</h2>

      <div class="card">
        <h3>陷阱 1：reactive 直接解构 → 解出来的不是响应式的</h3>
        <p class="result">主题：<b>{{ settings.theme }}</b></p>
        <button @click="changeByDestructure">点我：改"解构出来的 theme"</button>
        <p class="warning">🔴 点上面按钮 <b>视图没变化</b> —— 因为解构出来的是普通字符串副本，改它跟 settings 无关</p>
        <button @click="changeByToRefs">正确做法：toRefs 解构后改 .value</button>
        <p class="tip">✅ 点这个，主题变 <b>dark</b> 了：toRefs 解出来的是 ref，依旧响应</p>
        <code v-pre>const { theme } = settings  // ❌ 副本</code>
        <code v-pre>const { theme } = toRefs(settings)  // ✅ 响应式</code>
      </div>

      <div class="card">
        <h3>陷阱 2：reactive 整体替换 → 新对象不是代理</h3>
        <p class="result">settings2.theme = <b>{{ settings2.theme }}</b></p>
        <button @click="replaceSettings">点我：settings2 = { theme: 'dark' }</button>
        <p class="warning">🔴 点上面按钮 <b>视图没变化</b> —— settings2 指向了新的普通对象，Vue 不再代理它</p>
        <button @click="changeProperty">正确做法：settings2.theme = 'dark'</button>
        <p class="tip">✅ 点这个就变了：只改属性，不改引用</p>
        <code v-pre>settings2 = { theme: 'dark' }  // ❌ 丢掉代理</code>
        <code v-pre>settings2.theme = 'dark'     // ✅</code>
        <p class="tip">💡 记住：需要"整个对象替换"的场景（比如重置），用 ref 更安全——因为换的是 .value，壳还是那个壳。</p>
      </div>
    </section>

    <p class="footer">📌 口诀：<b>默认用 ref</b>（基本类型全靠它，对象也包得住）；reactive 只用于"不会整体替换、不会解构"的深嵌套对象；computed 派生值、watch 做副作用。</p>
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
  border-left: 4px solid #7c4dff;
  padding-left: 0.75rem;
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

button {
  margin: 0 6px 6px 0;
  padding: 4px 12px;
  border: 1px solid #7c4dff;
  border-radius: 6px;
  background: #fff;
  color: #7c4dff;
  cursor: pointer;
}
button:hover {
  background: #7c4dff;
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

.log {
  margin: 0.5rem 0;
  padding-left: 1.25rem;
  color: #555;
  font-size: 0.9rem;
  max-height: 150px;
  overflow-y: auto;
}

.footer {
  margin-top: 2rem;
  color: #666;
  background: #f4f1ff;
  border-radius: 8px;
  padding: 0.75rem 1rem;
}
</style>
