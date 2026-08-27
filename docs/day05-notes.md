# 第5天 · 事件处理与表单绑定（学习笔记）

> 日期：2026-08-27（补 8.15 第5天内容）· 学习计划第 5 天

## ✅ 今日完成

- 创建演示组件 `src/components/EventFormDemo.vue`，替换进 App.vue
- 亲手体验了 v-on 事件、修饰符（.stop/.prevent/.once）、按键修饰符
- 用双输入框对比理解了 v-model 的本质是语法糖
- 把 v-model 用在了所有表单控件上（text/textarea/checkbox/radio/select）
- 完成了计划任务：**即时搜索输入框**（v-model + computed 三行核心）

## 一、v-on 事件处理（缩写 @）

```vue
<button @click="count++">                <!-- 内联语句 -->
<button @click="addCount(1)">            <!-- 调用方法 + 传参 -->
<button @click="showEvent($event)">      <!-- $event = 原生事件对象 -->
<input @keyup.esc="clear" />             <!-- 按键修饰符 -->
```

- `@click` 就是 `v-on:click` 的缩写，是 Vue 里出现频率最高的指令
- 传参和事件对象可以同时要：`@click="fn($event, 'hi')"`

### 常用修饰符

| 修饰符 | 作用 | 典型场景 |
|---|---|---|
| `.stop` | 阻止事件冒泡 | 内层按钮不想触发外层点击 |
| `.prevent` | 阻止默认行为 | 表单提交不刷新页面（`@submit.prevent`） |
| `.once` | 只触发一次 | 一次性操作 |
| `.self` | 只有点击元素自身才触发 | 遮罩层点击不穿透 |
| `.capture` | 捕获阶段触发 | 少见 |
| `.passive` | 不阻止默认行为（滚动优化） | 移动端滚动 |
| `.enter` / `.esc` 等 | 按键修饰符 | 回车提交、Esc 取消 |

**记忆**：`.stop` 管冒泡、`.prevent` 管默认行为——前端两个最麻烦的事件问题，Vue 一个点解决。

## 二、v-model 原理：语法糖（今天最重要的认知）⭐

```vue
<!-- 这两行代码完全等价 -->
<input :value="name" @input="name = $event.target.value" />
<input v-model="name" />
```

**v-model = 绑定值 + 监听输入回写**，没有任何魔法：

| 控件 | 等价于 |
|---|---|
| input text / textarea | `:value` + `@input` |
| input checkbox | `:checked` + `@change` |
| select | `:value` + `@change` |
| radio | `:checked` + `@change` |

理解了语法糖之后，"双向绑定"就没有神秘感了：数据变 → 视图更新；输入 → 事件回写数据。**永远记得两条路都是 Vue 帮你接好的**。

## 三、v-model 各控件要点

```vue
<input type="checkbox" v-model="agree" />        <!-- 单选：布尔值 -->
<input type="checkbox" value="🏀" v-model="hobbies" />  <!-- 多选：绑定数组，勾选=加 value，取消=移除 -->
<input type="radio" value="男" v-model="gender" />
<select v-model="city"><option>北京</option></select>
```

关键区别：**单选 checkbox 绑布尔，多选 checkbox 绑数组**。多选时勾选/取消就是往数组里增删 value。

## 四、v-model 修饰符

```vue
<input v-model.lazy="x" />     <!-- 失焦/回车才同步（默认每次输入都同步） -->
<input v-model.trim="x" />     <!-- 自动去首尾空格 -->
<input type="number" v-model.number="x" />  <!-- 转数字（默认永远是 string） -->
<input v-model.lazy.trim.number="x" />      <!-- 可以链式叠加 -->
```

**.lazy 的本质**：把监听事件从 `input` 换成 `change`——每次敲键都同步 vs 失焦才同步，适合搜索类（及时）和表单类（省事）的不同需求。

## 五、即时搜索输入框（计划核心任务）

```ts
const search = ref('')
const fruits = ['🍎 苹果', '🍌 香蕉', ...]

const searchResult = computed(() => {
  const kw = search.value.trim().toLowerCase()
  if (!kw) return fruits
  return fruits.filter((f) => f.toLowerCase().includes(kw))
})
```

```vue
<input v-model="search" />
<ul v-if="searchResult.length">
  <li v-for="fruit in searchResult" :key="fruit">{{ fruit }}</li>
</ul>
<p v-else>🔍 没有匹配</p>
```

**数据流全貌**（今天所有知识的汇总）：
输入 → `@input` 回写 `search` → `search` 变化 → `computed` 重算 → 列表重渲染
——这条链上没有任何手动 DOM 操作，三行核心代码。这就是 Vue 声明式开发的精髓：**只管数据和派生关系，渲染是 Vue 的事**。

## 📌 小作业（15 分钟，巩固今天内容）

1. 组件第一节有冒泡演示：先点"会冒泡"按钮，再点".stop 不冒泡"按钮，看日志区别。
2. 手写版输入框改一下：去掉 `:value` 只留 `@input`，观察现象（输入不显示 → 理解绑定和回写缺一不可）。
3. 给即时搜索加需求：输入后按 Esc 清空搜索词（提示：`@keyup.esc` 今天刚学过）。
4. 思考题：`v-model.number` 在 `type="text"` 上输入 "12abc" 会得到什么？动手试试，体会 number 修饰符只做"能转就转"。
