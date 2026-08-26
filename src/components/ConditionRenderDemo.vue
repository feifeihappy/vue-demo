<!--
  8.14 学习主题：条件渲染与列表渲染
  演示内容：
    一、v-if vs v-show：创建/销毁 vs display:none，含 v-else-if 登录状态
    二、v-for 四种形态：数组 / 索引 / 对象 / 范围
    三、Todo 列表雏形：添加、删除、完成、过滤、空状态，key 用 id
    四、key 的作用：乱序对比实验（无 key 输入框错位，有 key 正确跟随）
-->
<script setup lang="ts">
import { ref, reactive, computed } from 'vue'

/* ============ 一、v-if vs v-show ============ */
const showIf = ref(true) // v-if 开关：false 时元素被销毁，DOM 里没有
const showShow = ref(true) // v-show 开关：只是 display:none，元素一直在

// 登录状态三态：v-if / v-else-if / v-else 只能有一个生效
type LoginState = 'guest' | 'loading' | 'user'
const loginState = ref<LoginState>('guest')

/* ============ 二、v-for 基础 ============ */
const fruits = ['🍎 苹果', '🍌 香蕉', '🍇 葡萄','🍊 橘子']
const person = reactive({ name: '小明', age: 12, city: '北京' })

/* ============ 三、Todo 列表雏形 ============ */
interface Todo {
  id: number
  text: string
  done: boolean
}

let nextId = 4
const todos = ref<Todo[]>([
  { id: 1, text: '学 v-if / v-show，搞清区别', done: true },
  { id: 2, text: '学 v-for 四种写法', done: false },
  { id: 3, text: '亲手验证 key 的作用', done: false },
])

const newTodo = ref('')
type Filter = 'all' | 'active' | 'done'
const filter = ref<Filter>('all')

function addTodo() {
  const text = newTodo.value.trim()
  if (!text) return // 空输入不加
  todos.value.push({ id: nextId++, text, done: false })
  newTodo.value = ''
}

function toggle(todo: Todo) {
  todo.done = !todo.done
}

function removeTodo(id: number) {
  todos.value = todos.value.filter((t) => t.id !== id)
}

// 未完成数量：由 todos 派生，自动更新
const remaining = computed(() => todos.value.filter((t) => !t.done).length)

// 过滤列表：也是派生值 —— 这样模板里不用把 v-if 和 v-for 写在一起
const filteredTodos = computed(() => {
  if (filter.value === 'active') return todos.value.filter((t) => !t.done)
  if (filter.value === 'done') return todos.value.filter((t) => t.done)
  return todos.value
})

/* ============ 四、key 的作用：乱序对比实验 ============ */
const group = ['Alice', 'Bob', 'Carol']
const noKeyList = ref([...group]) // 没有 key 的组
const withKeyList = ref([...group]) // 有 key 的组

function shuffle() {
  // Fisher-Yates 洗牌：两组用完全相同的乱序结果，方便对比
  const arr = [...group]
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    const tmp = arr[i]!
    arr[i] = arr[j]!
    arr[j] = tmp
  }
  noKeyList.value = [...arr]
  withKeyList.value = [...arr]
}
</script>

<template>
  <div class="page">
    <h1>🔀 8.14 条件渲染与列表渲染</h1>
    <p class="subtitle">v-if / v-show · v-for · key —— 让数据长成界面</p>

    <!-- ============ 一、v-if vs v-show ============ -->
    <section class="section">
      <h2>一、v-if vs v-show：到底差在哪？</h2>

      <div class="card">
        <h3>v-if：条件为 false 时，元素直接从 DOM 里消失（销毁）</h3>
        <button @click="showIf = !showIf">v-if 开关（当前 {{ showIf ? '显示' : '隐藏' }}）</button>
        <p v-if="showIf" class="block">✅ v-if 为 true，我在 DOM 里。</p>
        <p v-else class="block">↪️ 我是 v-else：v-if 变 false 时我顶上。</p>
        <p class="tip">💡 v-if 是"真条件渲染"：false 时元素不存在，切换代价是<b>创建 / 销毁</b>。适合不常切换的场景（如登录后显示用户面板）。</p>
      </div>

      <div class="card">
        <h3>v-show：元素一直在 DOM 里，只是 CSS display:none</h3>
        <button @click="showShow = !showShow">v-show 开关（当前 {{ showShow ? '显示' : '隐藏' }}）</button>
        <p v-show="showShow" class="block">✅ 我其实一直在 DOM 里，只是被 display:none 藏起来了。</p>
        <p class="tip">💡 切换只改一个 CSS 属性，<b>零销毁成本</b>。适合高频切换（选项卡、折叠面板）。打开 DevTools 检查元素：v-show 隐藏时元素还在，v-if 隐藏时彻底没了。</p>
      </div>

      <div class="card">
        <h3>v-if / v-else-if / v-else：多分支只能命中一个</h3>
        <p class="result">当前状态：<b>{{ loginState }}</b></p>
        <button @click="loginState = 'guest'">guest</button>
        <button @click="loginState = 'loading'">loading</button>
        <button @click="loginState = 'user'">user</button>
        <p v-if="loginState === 'user'" class="block">🎉 欢迎回来，用户！</p>
        <p v-else-if="loginState === 'loading'" class="block">⏳ 登录中，请稍候…</p>
        <p v-else class="block">👋 你好，访客，请先登录。</p>
        <code v-pre>v-if="... " v-else-if="..." v-else</code>
      </div>
    </section>

    <!-- ============ 二、v-for 基础 ============ -->
    <section class="section">
      <h2>二、v-for 四种形态</h2>

      <div class="card">
        <h3>1. 数组 + 索引：v-for="(item, index) in array"</h3>
        <ul class="plain-list">
          <li v-for="(fruit, index) in fruits" :key="fruit">{{ index }}. {{ fruit }}</li>
        </ul>
        <code v-pre>v-for="(fruit, index) in fruits" :key="fruit"</code>
      </div>

      <div class="card">
        <h3>2. 对象：v-for="(value, key) in object"</h3>
        <ul class="plain-list">
          <li v-for="(value, key) in person" :key="key">{{ key }}：{{ value }}</li>
        </ul>
        <code v-pre>v-for="(value, key) in person"</code>
      </div>

      <div class="card">
        <h3>3. 范围：v-for="n in 5"（渲染 1~5）</h3>
        <span v-for="n in 6" :key="n" class="badge">{{ n }}</span>
        <code v-pre>v-for="n in 5"  // 从 1 开始</code>
      </div>
    </section>

    <!-- ============ 三、Todo 列表雏形 ============ -->
    <section class="section">
      <h2>三、Todo 列表雏形（v-for + v-if + key 全家桶）</h2>

      <div class="card">
        <h3>输入 + 回车添加，点击文字切换完成，✕ 删除</h3>
        <div class="add-row">
          <input
            v-model="newTodo"
            class="add-input"
            placeholder="输入新待办，回车添加"
            @keyup.enter="addTodo"
          />
          <button @click="addTodo">添加</button>
        </div>

        <p class="result">未完成 <b>{{ remaining }}</b> 项 · 共 {{ todos.length }} 项</p>

        <div class="tabs">
          <button :class="{ active: filter === 'all' }" @click="filter = 'all'">全部</button>
          <button :class="{ active: filter === 'active' }" @click="filter = 'active'">未完成</button>
          <button :class="{ active: filter === 'done' }" @click="filter = 'done'">已完成</button>
        </div>

        <!-- 空状态：v-if 管"没有列表"时的提示 -->
        <ul v-if="filteredTodos.length" class="todo-list">
          <li v-for="todo in filteredTodos" :key="todo.id">
            <input type="checkbox" :checked="todo.done" @change="toggle(todo)" />
            <span :class="{ done: todo.done }" @click="toggle(todo)">{{ todo.text }}</span>
            <button class="del" @click="removeTodo(todo.id)">✕</button>
          </li>
        </ul>
        <p v-else class="empty">🎉 这个视图下没有待办——空状态交给 v-if 的搭档 v-else 处理</p>

        <code v-pre>v-for="todo in filteredTodos" :key="todo.id"</code>
        <p class="tip">💡 key 用 <b>todo.id</b>（数据自带、唯一、稳定）而不是 index——为什么？看下面第四部分。<br />💡 过滤用 computed 先算好，模板里就<b>不用把 v-for 和 v-if 写在同一元素</b>上（官方不推荐：v-if 优先级更高，循环变量根本用不到）。</p>
      </div>
    </section>

    <!-- ============ 四、key 的作用 ============ -->
    <section class="section">
      <h2>四、key 的作用：一个乱序实验看懂它</h2>

      <div class="card">
        <h3>先在上面的输入框随便打字，再点"打乱顺序"，对比两组的差别</h3>
        <button @click="shuffle">🔀 打乱顺序（两组用同一个乱序结果）</button>

        <p class="group-title">❌ 左边：没有 key —— Vue 按<b>位置</b>复用 DOM，输入框内容"跟着位置走"，错乱了</p>
        <div class="key-cols">
          <div class="key-col">
            <div v-for="name in noKeyList" class="key-row">
              <input class="mini-input" placeholder="输入点字" />
              <span>{{ name }}</span>
            </div>
          </div>

          <div class="key-col">
            <div v-for="name in withKeyList" :key="name" class="key-row">
              <input class="mini-input" placeholder="输入点字" />
              <span>{{ name }}</span>
            </div>
          </div>
        </div>
        <p class="group-title">✅ 右边：有 :key="name" —— Vue 按<b>身份</b>复用 DOM，输入框跟着名字走，不乱</p>

        <code v-pre>&lt;div v-for="name in list"&gt;                    &lt;!-- 无 key：位置复用 --&gt;</code>
        <code v-pre>&lt;div v-for="name in list" :key="name"&gt;        &lt;!-- 有 key：身份复用 --&gt;</code>
        <p class="warning">🔴 为什么？列表重排时，Vue 用 key 判断"哪个节点是哪个"。没有 key 就只能按顺序硬套，有状态的子元素（输入框内容、勾选状态）就会串位。</p>
        <p class="tip">✅ 什么时候用 index 当 key 也行？列表<b>纯展示、不增删、不改顺序</b>时（比如静态表格）。只要会增删/乱序，就必须用稳定唯一的 id。</p>
      </div>
    </section>

    <p class="footer">📌 口诀：<b>v-if 真销毁</b>（配合 v-else-if/v-else 分支）、<b>v-show 藏起来</b>；列表必配 <b>:key</b>，用 id 不用 index；要过滤先 computed 再 v-for。</p>
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
  border-left: 4px solid #2f9e44;
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

.block {
  border-radius: 6px;
  padding: 8px 12px;
  background: #ebfbee;
  color: #2b8a3e;
  font-weight: 600;
}

button {
  margin: 0 6px 6px 0;
  padding: 4px 12px;
  border: 1px solid #2f9e44;
  border-radius: 6px;
  background: #fff;
  color: #2f9e44;
  cursor: pointer;
}
button:hover {
  background: #2f9e44;
  color: #fff;
}
button.active {
  background: #2f9e44;
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

.plain-list {
  margin: 0.5rem 0;
  padding-left: 1.25rem;
  color: #444;
}

.badge {
  display: inline-block;
  margin: 0.25rem;
  padding: 2px 10px;
  border-radius: 999px;
  background: #ebfbee;
  color: #2b8a3e;
  font-weight: 700;
}

.add-row {
  display: flex;
  gap: 8px;
  margin-bottom: 0.5rem;
}

.add-input {
  flex: 1;
  padding: 6px 10px;
  border: 1px solid #ccc;
  border-radius: 6px;
  font-size: 0.95rem;
}
.add-input:focus {
  outline: 2px solid #2f9e44;
  border-color: transparent;
}

.tabs {
  margin: 0.5rem 0;
}

.todo-list {
  list-style: none;
  padding: 0;
  margin: 0.5rem 0;
}

.todo-list li {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 8px 10px;
  border-bottom: 1px dashed #eee;
}

.todo-list li span {
  flex: 1;
  cursor: pointer;
}

.todo-list li .done {
  text-decoration: line-through;
  color: #aaa;
}

.del {
  border-color: #ff6b6b;
  color: #ff6b6b;
  padding: 0 8px;
}
.del:hover {
  background: #ff6b6b;
  color: #fff;
}

.empty {
  text-align: center;
  color: #999;
  padding: 1rem;
}

.group-title {
  font-size: 0.85rem;
  color: #666;
  margin: 0.75rem 0 0.25rem;
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

.key-row {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 4px 0;
}

.mini-input {
  width: 90px;
  padding: 4px 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 0.85rem;
}

.footer {
  margin-top: 2rem;
  color: #666;
  background: #ebfbee;
  border-radius: 8px;
  padding: 0.75rem 1rem;
}
</style>
