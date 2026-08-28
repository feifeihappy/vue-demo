<!--
  8.17 综合小练习：购物车界面
  演示内容：
    一、购物车主体：商品列表、数量加减、删除、总价 computed、空状态
    二、陷阱剧场 1：index 当 key —— 删除商品后数量串位
    三、陷阱剧场 2：浮点精度 —— 0.1×3 = 0.30000000000000004
    四、陷阱剧场 3：reactive 数组整体替换 —— 清空不生效
-->
<script setup lang="ts">
import { ref, reactive, computed } from 'vue'

/* ============ 一、购物车主体 ============ */
interface Goods {
  id: number
  name: string
  price: number // 单价（元）
  count: number // 数量
}

const cart = ref<Goods[]>([
  { id: 1, name: '🍎 苹果', price: 3.5, count: 2 },
  { id: 2, name: '🍌 香蕉', price: 5.0, count: 1 },
  { id: 3, name: '🍇 葡萄', price: 12.8, count: 1 },
])

function changeCount(id: number, delta: number) {
  const item = cart.value.find((g) => g.id === id)
  if (!item) return
  item.count = Math.max(1, item.count + delta) // 数量最低 1，不减到负数
}

function removeGoods(id: number) {
  cart.value = cart.value.filter((g) => g.id !== id)
}

function clearCart() {
  cart.value = []
}

// 总价：派生值，自动跟着数量和删除更新
const totalPrice = computed(() =>
  cart.value.reduce((sum, g) => sum + g.price * g.count, 0)
)

// 商品件数：所有数量加起来
const totalCount = computed(() => cart.value.reduce((sum, g) => sum + g.count, 0))

/* ============ 二、陷阱剧场 ============ */
// 陷阱 1：index 当 key —— 两组一样的商品，一组用 index、一组用 id
// 操作：给第二行商品加数量，再删除"第一行"商品，观察数量是否串位
const cartByIndex = ref<Goods[]>([
  { id: 1, name: '🟥 红色箱子', price: 10, count: 2 },
  { id: 2, name: '🟦 蓝色箱子', price: 10, count: 3 },
  { id: 3, name: '🟩 绿色箱子', price: 10, count: 1 },
])
const cartById = ref<Goods[]>([
  { id: 1, name: '🟥 红色箱子', price: 10, count: 2 },
  { id: 2, name: '🟦 蓝色箱子', price: 10, count: 3 },
  { id: 3, name: '🟩 绿色箱子', price: 10, count: 1 },
])

function changeCountIndex(idx: number, delta: number) {
  const item = cartByIndex.value[idx]
  if (!item) return
  item.count = Math.max(1, item.count + delta)
}
function changeCountById(id: number, delta: number) {
  const item = cartById.value.find((g) => g.id === id)
  if (!item) return
  item.count = Math.max(1, item.count + delta)
}
function removeByIndex(idx: number) {
  cartByIndex.value.splice(idx, 1)
}
function removeById(id: number) {
  cartById.value = cartById.value.filter((g) => g.id !== id)
}

// 陷阱 2：浮点精度 —— 单价 0.1 元 × 3 件
const cheapGoods = reactive({ name: '🍬 散装糖果', price: 0.1, count: 3 })

// 陷阱 3：reactive 数组整体替换 —— 清空不生效
// 注意：reactive 的"整体替换"在 const 下编译期就拦住，所以这里用 let 才能演示出真实陷阱
let trap3Cart = reactive<Goods[]>([
  { id: 1, name: '📦 旧货箱 A', price: 1, count: 1 },
  { id: 2, name: '📦 旧货箱 B', price: 2, count: 2 },
])
function trap3WrongClear() {
  trap3Cart = [] // ❌ 换引用：新数组是普通数组，视图不更新
}
function trap3CorrectClear() {
  trap3Cart.splice(0, trap3Cart.length) // ✅ 原地清空
}
</script>

<template>
  <div class="page">
    <h1>🛒 8.17 综合小练习：购物车</h1>
    <p class="subtitle">商品列表 · 数量加减 · 总价 —— 前 5 天知识点的汇总考</p>

    <!-- ============ 一、购物车主体 ============ -->
    <section class="section">
      <h2>一、购物车（本次练习的核心）</h2>

      <div class="card">
        <template v-if="cart.length">
          <div class="cart-row cart-head">
            <span>商品</span>
            <span>单价</span>
            <span>数量</span>
            <span>小计</span>
            <span></span>
          </div>
          <div v-for="g in cart" :key="g.id" class="cart-row">
            <span>{{ g.name }}</span>
            <span>¥{{ g.price.toFixed(2) }}</span>
            <span class="qty">
              <button class="mini" @click="changeCount(g.id, -1)">−</button>
              <b>{{ g.count }}</b>
              <button class="mini" @click="changeCount(g.id, 1)">＋</button>
            </span>
            <span class="subtotal">¥{{ (g.price * g.count).toFixed(2) }}</span>
            <button class="del" @click="removeGoods(g.id)">✕</button>
          </div>

          <div class="cart-foot">
            <span>共 {{ totalCount }} 件</span>
            <span class="total">总价：<b>¥{{ totalPrice.toFixed(2) }}</b></span>
            <button class="danger" @click="clearCart">🗑 清空购物车</button>
          </div>
        </template>

        <p v-else class="empty">🛒 购物车是空的——去别处逛逛吧</p>

        <code v-pre>const cart = ref&lt;Goods[]&gt;([...])  // 商品数组</code>
        <code v-pre>const totalPrice = computed(() =&gt; cart.value.reduce((s, g) =&gt; s + g.price * g.count, 0))</code>
        <p class="tip">💡 今天用到的全是前几天的知识：<b>ref 数组</b>（8.13）· <b>v-for + :key="g.id"</b>（8.14）· <b>@click 传参</b>（8.15）· <b>computed 总价</b>（8.13）· <b>v-if 空状态</b>（8.14）。<br />💡 数量用 `Math.max(1, ...)` 卡下限，防止减成负数——这是购物车的常规边界处理。</p>
      </div>
    </section>

    <!-- ============ 二、陷阱剧场 1：index 当 key ============ -->
    <section class="section">
      <h2>二、陷阱剧场 1：index 当 key，删除商品后数量串位 ⚠️</h2>

      <div class="card">
        <p class="result">操作步骤：先给<b>蓝箱子</b>加几个数量，然后<b>删除红箱子</b>，对比左右两组</p>
        <div class="key-cols">
          <div class="key-col">
            <p class="group-title">❌ key 用 index：</p>
            <div v-for="(g, idx) in cartByIndex" :key="idx" class="mini-row">
              <span>{{ g.name }}</span>
              <span class="qty">
                <button class="mini" @click="changeCountIndex(idx, -1)">−</button>
                <b>{{ g.count }}</b>
                <button class="mini" @click="changeCountIndex(idx, 1)">＋</button>
              </span>
              <button class="del" @click="removeByIndex(idx)">✕</button>
            </div>
          </div>
          <div class="key-col">
            <p class="group-title">✅ key 用 id：</p>
            <div v-for="g in cartById" :key="g.id" class="mini-row">
              <span>{{ g.name }}</span>
              <span class="qty">
                <button class="mini" @click="changeCountById(g.id, -1)">−</button>
                <b>{{ g.count }}</b>
                <button class="mini" @click="changeCountById(g.id, 1)">＋</button>
              </span>
              <button class="del" @click="removeById(g.id)">✕</button>
            </div>
          </div>
        </div>
        <p class="warning">🔴 现象：删掉红箱子后，左组"蓝箱子"的显示数量变成 3（红箱子的数量）——因为删除后索引集体前移，Vue 按位置复用节点，把红箱子的状态"继承"给了新位置的蓝箱子。<br />✅ 右组一切正常：key 让 Vue 认准身份，状态跟着数据走。</p>
        <code v-pre>&lt;div v-for="(g, idx) in list" :key="idx"&gt;   &lt;!-- ❌ 会串位 --&gt;</code>
        <code v-pre>&lt;div v-for="g in list" :key="g.id"&gt;         &lt;!-- ✅ 认准身份 --&gt;</code>
      </div>
    </section>

    <!-- ============ 三、陷阱剧场 2：浮点精度 ============ -->
    <section class="section">
      <h2>三、陷阱剧场 2：浮点精度，总价算出 0.30000000000000004 ⚠️</h2>

      <div class="card">
        <p class="result">{{ cheapGoods.name }}：单价 ¥{{ cheapGoods.price }} × {{ cheapGoods.count }} 件</p>
        <button @click="cheapGoods.count++">加一件</button>
        <p class="result">
          不处理：<b>¥{{ cheapGoods.price * cheapGoods.count }}</b>
          <span class="bad">（一串诡异小数）</span>
        </p>
        <p class="result">用 toFixed(2)：<b>¥{{ (cheapGoods.price * cheapGoods.count).toFixed(2) }}</b> ✅</p>
        <code v-pre>0.1 * 3   // 0.30000000000000004 —— JS 浮点存储精度问题</code>
        <code v-pre>price.toFixed(2)  // 显示层兜底：保留两位小数</code>
        <p class="tip">💡 这是所有电商前端都会踩的坑：金额计算在 JS 里是浮点，展示时一律 `toFixed(2)` 兜底（组件一里的总价就是这么处理的）。真正的金额精确计算（如优惠分摊）需要整数分或 decimal 库。</p>
      </div>
    </section>

    <!-- ============ 四、陷阱剧场 3：reactive 整体替换 ============ -->
    <section class="section">
      <h2>四、陷阱剧场 3：reactive 数组"清空"，视图一动不动 ⚠️</h2>

      <div class="card">
        <p class="result">trap3Cart 当前 {{ trap3Cart.length }} 个商品</p>
        <button @click="trap3WrongClear">❌ trap3Cart = []</button>
        <button @click="trap3CorrectClear">✅ splice(0, length) 清空</button>
        <p class="warning">🔴 点第一个按钮视图没变化：`trap3Cart = []` 把变量指向了新数组，但 Vue 代理的仍然是旧数组——引用换了，代理没换。<br />✅ 点第二个按钮正常清空：`splice` 是在<b>原数组上原地操作</b>，代理还在。</p>
        <code v-pre>trap3Cart = []                    // ❌ 换引用，丢代理</code>
        <code v-pre>trap3Cart.splice(0, trap3Cart.length)  // ✅ 原地清空</code>
        <p class="tip">💡 这正是 8.13 讲过的"reactive 不能整体替换"——在购物车清空场景的复现。所以购物车建议用 <b>ref 包数组</b>：清空写 `cart.value = []` 安全（换的是 .value 里层，壳还是那个壳）。上面主购物车就是这么写的。</p>
      </div>
    </section>

    <p class="footer">📌 练习总结：<b>key 用 id 不用 index</b>（增删不串位）、<b>金额展示 toFixed(2)</b>（浮点兜底）、<b>购物车用 ref 包数组</b>（清空可整体替换）；总价永远是 computed 派生值，别手写同步。</p>
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
button.mini {
  padding: 0 8px;
  margin: 0 4px;
  border-color: #ccc;
  color: #333;
}
button.mini:hover {
  background: #eee;
  color: #333;
}
button.del {
  border-color: #ff6b6b;
  color: #ff6b6b;
  padding: 0 8px;
}
button.del:hover {
  background: #ff6b6b;
  color: #fff;
}
button.danger {
  border-color: #ff6b6b;
  color: #ff6b6b;
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

.bad {
  color: #e8590c;
  font-size: 0.85rem;
}

.empty {
  text-align: center;
  color: #999;
  padding: 1.5rem;
}

.cart-row {
  display: grid;
  grid-template-columns: 2fr 1fr 1.2fr 1fr 0.5fr;
  align-items: center;
  gap: 8px;
  padding: 8px 0;
  border-bottom: 1px dashed #eee;
}

.cart-head {
  font-size: 0.8rem;
  color: #999;
  border-bottom: 1px solid #e5e5e5;
}

.qty {
  display: flex;
  align-items: center;
}

.subtotal {
  color: #e8590c;
  font-weight: 700;
}

.cart-foot {
  display: flex;
  align-items: center;
  gap: 16px;
  padding-top: 10px;
}

.cart-foot .total {
  flex: 1;
  text-align: right;
  font-size: 1.1rem;
}

.cart-foot .total b {
  color: #e8590c;
  font-size: 1.3rem;
}

.key-cols {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

.key-col {
  border: 1px dashed #ccc;
  border-radius: 8px;
  padding: 8px;
}

.group-title {
  font-size: 0.85rem;
  color: #666;
  margin: 0.25rem 0;
}

.mini-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  padding: 4px 0;
}

.footer {
  margin-top: 2rem;
  color: #666;
  background: #fff4e6;
  border-radius: 8px;
  padding: 0.75rem 1rem;
}
</style>
