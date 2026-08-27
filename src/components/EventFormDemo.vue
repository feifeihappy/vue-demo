<!--
  8.15 学习主题：事件处理与表单绑定
  演示内容：
    一、v-on 事件：@click 传参、$event、.stop/.prevent/.once、按键修饰符
    二、v-model 原理：:value + @input 语法糖拆解，双输入框对比
    三、v-model 各控件：text / textarea / checkbox / radio / select
    四、修饰符：.lazy / .trim / .number
    五、即时搜索输入框（综合应用）
-->
<script setup lang="ts">
import { ref, computed } from 'vue'

/* ============ 一、v-on 事件处理 ============ */
const count = ref(0)
function addCount(n: number) {
  count.value += n
}

// $event 事件对象：能看到事件本身的信息
const eventInfo = ref('（还没触发）')
function showEvent(e: Event) {
  eventInfo.value = `在 <${(e.target as HTMLElement).tagName}> 上触发，event.type = ${e.type}`
}

// .stop：阻止冒泡 —— 内层按钮点了不冒泡到外层
const bubbleLog = ref<string[]>(['（等待点击…）'])
function onOuterClick() {
  bubbleLog.value.unshift(`外层被点了：${new Date().toLocaleTimeString()}`)
}
function onInnerClick() {
  bubbleLog.value.unshift('内层被点了（没有 .stop 的话会连着外层一起触发）')
}
function onInnerClickStop() {
  bubbleLog.value.unshift('内层被点了，.stop 挡住了冒泡，外层不会触发 ✅')
}

// .once：只触发一次
const onceMsg = ref('按钮还没被点过')
function onClickOnce() {
  onceMsg.value = `✅ 只触发了一次（${new Date().toLocaleTimeString()}）`
}

// 按键修饰符：按 Esc 清空
const escText = ref('在这个框里打字，按 Esc 清空')

/* ============ 二、v-model 原理 ============ */
// v-model 是 :value + @input 的语法糖，两个输入框行为完全一样
const manualName = ref('') // 手写版
const vmodelName = ref('') // 语法糖版

/* ============ 三、v-model 各控件 ============ */
const text = ref('')
const message = ref('')
const agree = ref(false)
const hobbies = ref<string[]>([])
const gender = ref('')
const city = ref('')

/* ============ 四、修饰符 lazy / trim / number ============ */
const lazyVal = ref('')
const trimVal = ref('')
const numVal = ref<number | string>(0)

/* ============ 五、即时搜索 ============ */
const search = ref('')
const fruits = ['🍎 苹果', '🍌 香蕉', '🍇 葡萄', '🍊 橘子', '🍓 草莓', '🥭 芒果', '🍉 西瓜', '🍑 桃子']

// 过滤是派生值：输入一变，结果自动重算 —— 这就是双向绑定驱动的即时搜索
const searchResult = computed(() => {
  const kw = search.value.trim().toLowerCase()
  if (!kw) return fruits
  return fruits.filter((f) => f.toLowerCase().includes(kw))
})
</script>

<template>
  <div class="page">
    <h1>⌨️ 8.15 事件处理与表单绑定</h1>
    <p class="subtitle">v-on 事件 · 修饰符 · v-model —— 用户输入与数据的桥</p>

    <!-- ============ 一、v-on 事件 ============ -->
    <section class="section">
      <h2>一、v-on 事件处理（@click 就是 v-on:click）</h2>

      <div class="card">
        <h3>1. 内联调用方法 + 传参</h3>
        <p class="result">点击次数：<b>{{ count }}</b></p>
        <button @click="addCount(1)">+1</button>
        <button @click="addCount(5)">+5</button>
        <code v-pre>@click="addCount(1)"  // 方法名 + 括号传参</code>
      </div>

      <div class="card">
        <h3>2. $event：把原生事件对象传进去</h3>
        <button @click="showEvent($event)">点我</button>
        <p class="result">{{ eventInfo }}</p>
        <code v-pre>@click="showEvent($event)"  // 想同时传参：showEvent($event, 'hi')</code>
      </div>

      <div class="card">
        <h3>3. 修饰符 .stop：阻止事件冒泡</h3>
        <div class="bubble-box" @click="onOuterClick">
          🟫 外层盒子（@click）
          <div class="bubble-inner">
            <button @click="onInnerClick">点我（会冒泡）</button>
            <button @click.stop="onInnerClickStop">点我（.stop 不冒泡）</button>
          </div>
        </div>
        <ul class="log">
          <li v-for="(log, i) in bubbleLog" :key="i">{{ log }}</li>
        </ul>
        <code v-pre>@click.stop="onInnerClickStop"  // 阻止冒泡，外层不会被触发</code>
      </div>

      <div class="card">
        <h3>4. 修饰符 .once 和按键修饰符</h3>
        <p class="result">{{ onceMsg }}</p>
        <button @click.once="onClickOnce">只响一次</button>
        <input class="add-input" v-model="escText" @keyup.esc="escText = ''" />
        <p class="tip">💡 <b>.once</b>：事件只触发一次；<b>@keyup.esc</b>：只监听 Esc 键（还有 .enter/.tab/.ctrl 等）。<br />💡 常用修饰符：<b>.stop</b> 阻止冒泡、<b>.prevent</b> 阻止默认行为（表单提交刷新页面）、<b>.once</b> 只一次、<b>.self</b> 只有点击自身才触发。</p>
      </div>
    </section>

    <!-- ============ 二、v-model 原理 ============ -->
    <section class="section">
      <h2>二、v-model 原理：就是 :value + @input 的语法糖</h2>

      <div class="card">
        <h3>两个输入框行为完全一样，写法差在哪？</h3>
        <div class="add-row">
          <input
            class="add-input"
            placeholder="① 手写版：:value + @input"
            :value="manualName"
            @input="manualName = ($event.target as HTMLInputElement).value"
          />
          <input class="add-input" placeholder="② 语法糖版：v-model" v-model="vmodelName" />
        </div>
        <p class="result">① 手写版的值：<b>{{ manualName }}</b> ｜ ② 语法糖的值：<b>{{ vmodelName }}</b></p>
        <code v-pre>① &lt;input :value="name" @input="name = $event.target.value"&gt;</code>
        <code v-pre>② &lt;input v-model="name"&gt;  // 编译器帮你做上面那两件事</code>
        <p class="tip">💡 原理：v-model = <b>绑定值（:value）+ 监听输入（@input）回写</b>。不同控件配对不同：text/textarea 用 value+input，checkbox 用 checked+change，select 用 value+change。<br />💡 所以"双向绑定"其实不存在魔法——数据改 → 视图更新；输入 → 事件回写数据。两行代码拼成一个语法糖。</p>
      </div>
    </section>

    <!-- ============ 三、v-model 各控件 ============ -->
    <section class="section">
      <h2>三、v-model 各种表单控件</h2>

      <div class="card">
        <h3>text + textarea</h3>
        <div class="add-row">
          <input class="add-input" v-model="text" placeholder="输入文字" />
          <textarea class="add-input" v-model="message" rows="2" placeholder="多行文本"></textarea>
        </div>
        <p class="result">text: <b>{{ text }}</b> ｜ textarea: <b>{{ message }}</b></p>
      </div>

      <div class="card">
        <h3>checkbox 单选（布尔值）+ 多选（数组）</h3>
        <label class="row"><input type="checkbox" v-model="agree" /> 我同意协议（{{ agree }}）</label>
        <p class="row">
          爱好：
          <label><input type="checkbox" value="🏀 篮球" v-model="hobbies" /> 🏀 篮球</label>
          <label><input type="checkbox" value="🎮 游戏" v-model="hobbies" /> 🎮 游戏</label>
          <label><input type="checkbox" value="📖 读书" v-model="hobbies" /> 📖 读书</label>
        </p>
        <p class="result">hobbies: <b>{{ hobbies.join('、') || '（空）' }}</b></p>
        <p class="tip">💡 多选 checkbox 的 v-model 绑定<b>数组</b>：勾选 = 往数组加 value，取消 = 移除。这就是单选（布尔）和多选（数组）的区别。</p>
      </div>

      <div class="card">
        <h3>radio + select</h3>
        <p class="row">
          性别：
          <label><input type="radio" value="男" v-model="gender" /> 男</label>
          <label><input type="radio" value="女" v-model="gender" /> 女</label>
        </p>
        <select v-model="city" class="add-input">
          <option value="">请选择城市</option>
          <option>北京</option>
          <option>上海</option>
          <option>深圳</option>
        </select>
        <p class="result">gender: <b>{{ gender || '（未选）' }}</b> ｜ city: <b>{{ city || '（未选）' }}</b></p>
      </div>
    </section>

    <!-- ============ 四、修饰符 ============ -->
    <section class="section">
      <h2>四、v-model 修饰符：lazy / trim / number</h2>

      <div class="card">
        <h3>.lazy：不再每次输入都更新，失焦（或回车）才同步</h3>
        <input class="add-input" v-model.lazy="lazyVal" placeholder="随便输入，然后点别处" />
        <p class="result">下方数值在<b>失焦后</b>才变：<b>{{ lazyVal || '（空）' }}</b></p>
        <code v-pre>v-model.lazy="lazyVal"  // 默认是 input 事件，lazy 改成 change 事件</code>
      </div>

      <div class="card">
        <h3>.trim：自动去掉首尾空格</h3>
        <input class="add-input" v-model.trim="trimVal" placeholder="输入带空格：  hi  " />
        <p class="result">显示加引号看效果：<b>"{{ trimVal }}"</b></p>
        <code v-pre>v-model.trim="trimVal"  // 存进去之前先 trim 一次</code>
      </div>

      <div class="card">
        <h3>.number：输入自动转数字（type="number" 时甚至默认生效）</h3>
        <input class="add-input" type="number" v-model.number="numVal" placeholder="输入数字" />
        <p class="result">值：<b>{{ numVal }}</b> ｜ 类型：<b>{{ typeof numVal }}</b>（改成数字类型而不是字符串）</p>
        <code v-pre>v-model.number="numVal"  // 没有它：输入框的值永远是 string</code>
        <p class="tip">💡 三个一起用：<code class="inline">v-model.lazy.trim.number="x"</code> 修饰符可以链式叠加。</p>
      </div>
    </section>

    <!-- ============ 五、即时搜索 ============ -->
    <section class="section">
      <h2>五、即时搜索输入框（v-model + computed 综合应用）</h2>

      <div class="card">
        <h3>每敲一个字，列表立刻过滤 —— 输入到结果全程没有一行"手动操作"</h3>
        <input class="add-input" v-model="search" placeholder="搜索水果，比如输入「苹」" />
        <p class="result">搜索词：<b>{{ search }}</b> ｜ 匹配 {{ searchResult.length }} / {{ fruits.length }} 个</p>

        <ul v-if="searchResult.length" class="plain-list">
          <li v-for="(fruit, i) in searchResult" :key="fruit">{{ i + 1 }}. {{ fruit }}</li>
        </ul>
        <p v-else class="empty">🔍 没有匹配「{{ search }}」的水果</p>

        <code v-pre>const searchResult = computed(() =&gt; fruits.filter(f =&gt; f.includes(search.value)))</code>
        <p class="tip">💡 数据流全貌：<b>输入</b> → @input 回写 search → search 变化 → computed 重算 → 列表重渲染。这条链上没有任何手动 DOM 操作，这就是 Vue 声明式开发的精髓。</p>
      </div>
    </section>

    <p class="footer">📌 口诀：<b>@ 缩写 v-on</b>，冒泡用 .stop、刷新用 .prevent、只响一次用 .once；表单一律 <b>v-model</b>（:value+@input 的糖），懒更新 .lazy、去空格 .trim、转数字 .number；过滤列表 = v-model + computed，即时搜索三行搞定。</p>
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
  border-left: 4px solid #1971c2;
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
  border: 1px solid #1971c2;
  border-radius: 6px;
  background: #fff;
  color: #1971c2;
  cursor: pointer;
}
button:hover {
  background: #1971c2;
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

code.inline {
  display: inline;
  padding: 2px 6px;
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
  min-width: 0;
}
.add-input:focus {
  outline: 2px solid #1971c2;
  border-color: transparent;
}

.row {
  margin: 0.5rem 0;
}

.row label {
  margin-right: 12px;
  cursor: pointer;
}

.bubble-box {
  border: 2px dashed #1971c2;
  border-radius: 8px;
  padding: 12px;
  color: #1971c2;
  font-size: 0.85rem;
}

.bubble-inner {
  margin-top: 8px;
}

.log {
  margin: 0.5rem 0;
  padding-left: 1.25rem;
  color: #555;
  font-size: 0.85rem;
  max-height: 140px;
  overflow-y: auto;
}

.plain-list {
  margin: 0.5rem 0;
  padding-left: 1.25rem;
  color: #444;
}

.empty {
  text-align: center;
  color: #999;
  padding: 1rem;
}

.footer {
  margin-top: 2rem;
  color: #666;
  background: #e7f5ff;
  border-radius: 8px;
  padding: 0.75rem 1rem;
}
</style>
