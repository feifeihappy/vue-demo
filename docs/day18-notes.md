# 第8天 · 组件基础：组件定义、props、emits、单向数据流（学习笔记）

> 日期：2026-08-28（补 8.18 第8天内容）· 学习计划第 2 周周二

## ✅ 今日完成

- 创建演示组件：`src/components/ComponentDemo.vue`（父组件）+ `src/components/ProductCard.vue`（子组件），替换进 App.vue
- 亲手验证了组件复用（同一组件渲染 3 次，❤️ 收藏状态各自独立）
- 用 ❌ 陷阱按钮复现了「直接改 props 嵌套属性 → 真的污染了父组件数据」
- 完成了计划任务：**组件定义、props、emits、单向数据流**

## 一、组件是什么

组件 = 可复用的 UI 积木。一个 `.vue` 文件就是一个组件，由三部分组成：

| 部分 | 作用 |
|---|---|
| `<template>` | 组件的结构（HTML） |
| `<script setup>` | 组件的逻辑（数据、方法） |
| `<style scoped>` | 组件的样式（scoped = 只作用于本组件） |

```vue
<ProductCard v-for="g in goodsList" :key="g.id" :product="g" @like="onLike" />
```

同一个组件可以渲染任意多份，**每份实例都有自己的状态**——今天 3 张卡片的 ❤️ 收藏互不影响。这就是组件复用的意义：UI + 行为打包成积木，到处拼。

## 二、props：父 → 子

```ts
// 子组件声明"收什么数据"（defineProps）
const props = defineProps<{
  product: Goods
  tag?: string   // 可选 prop
}>()
```

```vue
<ProductCard :product="g" tag="⭐ 热销" />
<!--           ↑ 动态传值（变量）  ↑ 静态传值（字符串字面量） -->
```

- 父组件在标签上写 `xxx="..."`（静态）或 `:xxx="变量"`（动态）传值
- 子组件用 `defineProps` 接收：模板里直接用 `product.name`，脚本里用 `props.product.name`
- **props 是只读的**——数据的所有权在父组件

## 三、emits：子 → 父

```ts
// 子组件声明"能发什么事件"
const emit = defineEmits<{
  like: [id: number]
  add: [id: number]
}>()

emit('like', props.product.id)  // 发出事件 + 带参数
```

```vue
<!-- 父组件监听 -->
<ProductCard @like="onLike" />
```

```ts
// 父组件收到事件，自己改自己的数据（今天:记录日志 + 加购计数）
function onLike(id: number) {
  const g = goodsList.value.find((x) => x.id === id)
  if (!g) return
  events.value.unshift(`❤️ 喜欢了「${g.name}」`)
}
```

- 子组件想"报告"事情：`emit('事件名', 参数)`
- 父组件用 `@事件名="handler"` 监听，handler 收到参数自己处理
- **事件只能"上报"，不带决策权**——数据怎么改永远是父组件说了算

## 四、单向数据流（今天的核心）⭐

```
父组件（数据的所有者）
   │
   │ ① props 向下传（只读）
   ▼
子组件
   │
   │ ② emit 事件向上报（只报"发生了什么"）
   ▼
父组件 @like="onLike"：收到事件，自己改自己的数据
```

**数据永远"从上往下流"**：

| 方向 | 通道 | 职责 |
|---|---|---|
| 父 → 子 | props | 传数据（只读） |
| 子 → 父 | emit 事件 | 报事件（无决策权） |
| 子内部 | ref | 自己管自己用、父不关心的状态（如 ❤️ 收藏） |

## 五、陷阱：直接改 props，真的能改成功 ⚠️（今天亲手复现）

```ts
props.product.name += '！'   // ❌ 嵌套对象属性：改得动！
```

现象：点卡片「❌ 直接改 props.name」→ **名字真的变了**。

为什么拦不住？三层原因：

1. **运行时**：Vue 的 props 只读是"浅"的（shallowReadonly 只锁第一层）。`props.product = x` 会被拦截并警告，但 `props.product.name = x` 改的是嵌套对象属性，拦不住
2. **数据源**：`product` 对象本身就是父组件响应式数组里的元素 → 改了它，父组件所有引用这个对象的地方都跟着变，**父组件却不知道是谁改的**
3. **编译期**：TS 的 `Readonly<T>` 同样是浅的，只锁 `props.product` 这个引用，`product.name` 的类型仍是可写的——编译期放行

结论：**永远不要改 props 的任何属性**。想改数据 → emit 事件报上去，让父组件改（今天「♻️ 重置」按钮就是父组件行使所有权）。

## 📌 小作业（20 分钟，巩固今天内容）

1. 点卡片上的「❌ 直接改 props.name」，看名字真的变了；再点「♻️ 重置」恢复——体会"数据归父组件管"。
2. 在 ProductCard.vue 里写 `props.product = {...}`，看编辑器报错 + 浏览器控制台警告（第一层只读是什么效果）。
3. 给 ProductCard 加一个自己的新事件（比如 `soldout` 缺货），父组件监听后把对应卡片置灰。
4. 思考题：为什么 Vue 不让子组件直接改 props？如果允许，调试时会遇到什么噩梦？（提示：想想"谁改的"和"改了几次"）
