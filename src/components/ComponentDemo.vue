<!--
  8.18 组件基础：组件定义、props、emits、单向数据流
  演示：
    一、组件复用：同一张 ProductCard 渲染 3 次，各自状态独立
    二、props 传值（静态 + 动态）、emits 事件上报（父组件记录日志）
    三、陷阱剧场：直接改 props 的嵌套属性 → 真的污染了父组件数据
    四、单向数据流图解
-->
<script setup lang="ts">
import { ref } from 'vue'
import ProductCard from './ProductCard.vue'

interface Goods {
  id: number
  name: string
  price: number
}

// 每次调用都返回全新对象——"重置"时不会被之前污染过的对象影响
function createInitialGoods(): Goods[] {
  return [
    { id: 1, name: '🍎 苹果', price: 3.5 },
    { id: 2, name: '🍌 香蕉', price: 5.0 },
    { id: 3, name: '🍇 葡萄', price: 12.8 },
  ]
}

// 父组件的数据：所有商品数据都归父组件所有，通过 props 分发给子组件
const goodsList = ref<Goods[]>(createInitialGoods())

// 子组件 emit 上来的事件，由父组件记录并处理
interface LogItem {
  id: number
  text: string
}
let eventId = 0
const events = ref<LogItem[]>([])
const cartCount = ref(0)

function pushEvent(text: string) {
  events.value.unshift({ id: eventId++, text })
  if (events.value.length > 5) events.value.pop() // 只保留最近 5 条
}

function findGoods(id: number) {
  return goodsList.value.find((g) => g.id === id)
}

// 监听子组件的 like 事件
function onLike(id: number) {
  const g = findGoods(id)
  if (!g) return
  pushEvent(`❤️ 喜欢了「${g.name}」`)
}

// 监听子组件的 add 事件
function onAdd(id: number) {
  const g = findGoods(id)
  if (!g) return
  cartCount.value++
  pushEvent(`🛒 把「${g.name}」加进购物车（第 ${cartCount.value} 件）`)
}

// 陷阱演示：子组件直接改了 props 后，父组件用"重置"恢复——数据只有父组件能改
function resetList() {
  goodsList.value = createInitialGoods() // ref 数组整体替换（复习 8.13）
  pushEvent('♻️ 父组件重置了商品数据')
}
</script>

<template>
  <div class="page">
    <h1>🧩 8.18 组件基础：props / emits / 单向数据流</h1>
    <p class="subtitle">组件 = 可复用的 UI 积木 · 父传子用 props · 子报父用 emit</p>

    <!-- ============ 一、组件复用 ============ -->
    <section class="section">
      <h2>一、组件复用：一张卡片渲染 3 次，状态各自独立</h2>
      <div class="card">
        <div class="cards">
          <ProductCard
            v-for="g in goodsList"
            :key="g.id"
            :product="g"
            tag="⭐ 热销"
            @like="onLike"
            @add="onAdd"
          />
        </div>
        <p class="tip">💡 同一个组件渲染 3 份，每份的 ❤️ 收藏状态互不影响——<b>每个组件实例都有自己的状态</b>，这就是组件复用的基础。</p>
        <code v-pre>&lt;ProductCard :product="g" tag="⭐ 热销" @like="onLike" @add="onAdd" /&gt;</code>
        <p class="hint">:product 是<b>动态</b>传值（变量）· tag 是<b>静态</b>传值（字符串字面量）· @like / @add 是监听子组件事件</p>
      </div>
    </section>

    <!-- ============ 二、emits 事件上报 ============ -->
    <section class="section">
      <h2>二、子 → 父：emits 事件上报</h2>
      <div class="card">
        <p class="result">🛒 已加购 <b>{{ cartCount }}</b> 件（父组件的计数，由子组件 emit 驱动）</p>
        <ul class="log">
          <li v-for="e in events" :key="e.id">{{ e.text }}</li>
        </ul>
        <p v-if="!events.length" class="empty-log">点卡片上的「👍 点赞 / 🛒 加购」，事件会冒到父组件这里</p>
        <code v-pre>const emit = defineEmits&lt;{ like: [id: number]; add: [id: number] }&gt;()  // 子组件声明事件</code>
        <code v-pre>emit('like', props.product.id)  // 子组件发出事件 + 参数</code>
        <code v-pre>@like="onLike"                 // 父组件监听，收到后自己改自己的数据</code>
        <p class="tip">💡 <b>子组件永远不直接改父组件的数据</b>，而是发一个"事件"告诉父组件"发生了什么事"，数据怎么改由父组件决定。</p>
      </div>
    </section>

    <!-- ============ 三、陷阱剧场 ============ -->
    <section class="section">
      <h2>三、陷阱剧场：直接改 props，真的能改成功 ⚠️</h2>
      <div class="card">
        <p class="result">点任意卡片上的「❌ 直接改 props.name」按钮，再看卡片名字：</p>
        <p class="warning">🔴 名字真的变了！`props.product.name = ...` 不是"改不动"，而是<b>改得动且会污染父组件的数据</b>。props 的只读是"浅"的（shallowReadonly 只锁第一层，嵌套对象的属性拦不住），而 `product` 对象本身就是父组件响应式数组里的元素——改了它，父组件所有引用这个对象的地方都会变，父组件却不知道是谁改的。<br />✅ 正确的做法是像上面一样 emit 事件，让父组件自己改。</p>
        <button @click="resetList">♻️ 重置商品数据（只有父组件能改）</button>
        <code v-pre>props.product.name += '！'   // ❌ 嵌套属性拦不住，直接污染父数据</code>
        <code v-pre>props.product = {...}        // ❌ 第一层赋值：编译期报错 + 运行时警告</code>
        <p class="tip">💡 为什么 TS 帮不了第一个陷阱？`Readonly&lt;T&gt;` 是浅的——只锁 `props.product` 这个引用，`product.name` 的属性类型仍是可写的。这也是 JS 项目里这类 bug 特别隐蔽的原因。</p>
      </div>
    </section>

    <!-- ============ 四、单向数据流 ============ -->
    <section class="section">
      <h2>四、单向数据流</h2>
      <div class="card">
        <pre class="flow">父组件（数据的所有者）
   │
   │ ① props 向下传（只读，数据不能往回改）
   ▼
子组件
   │
   │ ② emit 事件向上报（只报"发生了什么"，不带决策权）
   ▼
父组件 @like="onLike"：收到事件，自己改自己的数据</pre>
        <p class="footer">📌 口诀：<b>数据向下传（props），事件向上报（emits）；数据归父管，子想改？报上去，让父改。</b></p>
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

.cards {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
}

.log {
  list-style: none;
  padding: 0;
  margin: 0.5rem 0;
}

.log li {
  padding: 4px 8px;
  border-bottom: 1px dashed #eee;
  font-size: 0.9rem;
}

.empty-log {
  color: #999;
  font-size: 0.85rem;
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
