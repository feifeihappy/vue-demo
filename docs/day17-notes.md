# 第7天 · 综合小练习：购物车（学习笔记）

> 日期：2026-08-28（补 8.17 第7天内容）· 学习计划第 2 周周一

## ✅ 今日完成

- 创建演示组件 `src/components/CartDemo.vue`，替换进 App.vue
- 完成了计划任务：**购物车界面**（商品列表、数量加减、总价）
- 亲手复现了 3 个购物车场景的响应式陷阱，验证了前 5 天的知识

## 一、购物车核心实现（前 5 天知识的汇总考）

```ts
interface Goods {
  id: number    // 唯一标识 —— 这是 key 的来源
  name: string
  price: number // 单价（元）
  count: number // 数量
}

const cart = ref<Goods[]>([...])  // 8.13：ref 包数组

function changeCount(id: number, delta: number) {
  const item = cart.value.find((g) => g.id === id)
  if (!item) return
  item.count = Math.max(1, item.count + delta)  // 边界：数量最少 1
}

const totalPrice = computed(() =>
  cart.value.reduce((sum, g) => sum + g.price * g.count, 0)  // 8.13：computed 派生
)
```

| 用到哪天知识 | 用法 |
|---|---|
| 8.13 | `ref` 包数组、`computed` 算总价 |
| 8.14 | `v-for` + `:key="g.id"` 渲染列表、`v-if` 空购物车状态 |
| 8.15 | `@click="changeCount(g.id, 1)"` 传参事件 |
| 8.13+8.15 | `Math.max(1, ...)` 防数量减成负数（边界处理） |

## 二、陷阱 1：index 当 key，删除后数量串位 ⚠️（8.14 知识实战复现）

两列商品，左列 `:key="idx"`、右列 `:key="g.id"`。操作：蓝箱子加数量 → 删除红箱子：

- ❌ 左列：删除后索引集体前移，**蓝箱子显示的数量变成红箱子的数量**——Vue 按位置复用节点，"继承"了旧位置的 DOM 状态
- ✅ 右列：key 认准身份，状态跟着数据走

**结论**：只要列表会增删/乱序，key 就必须是稳定唯一的 id。购物车这种频繁增删的场景，用 index 必出 bug。

## 三、陷阱 2：浮点精度，总价 0.30000000000000004 ⚠️

```js
0.1 * 3  // → 0.30000000000000004
```

JS 的 Number 是 IEEE 754 浮点，二进制存不下 0.1 的精确值。电商金额展示**一律 `toFixed(2)` 兜底**：

```ts
totalPrice.value.toFixed(2)  // "3.50"
```

真实项目的金额精确计算（优惠分摊等）要用整数"分"或 decimal 库，展示层 toFixed 只是最低要求。

## 四、陷阱 3：reactive 数组"清空"不生效 ⚠️（8.13 知识实战复现）

```ts
let trap3Cart = reactive([...])

trap3Cart = []                    // ❌ 换引用：新数组是普通数组，Vue 不代理它，视图不动
trap3Cart.splice(0, trap3Cart.length)  // ✅ 原地清空：代理还在，视图更新
```

**为什么购物车用 `ref` 包数组更省心**：`cart.value = []` 换的是 .value 里层，外壳（ref）还是同一个，响应性不丢。reactive 只能"原地改"，清空要用 splice。

## 📌 小作业（20 分钟，巩固今天内容）

1. 页面上实际操作一遍陷阱 1：蓝箱子加数量 → 删红箱子 → 对比左右列，把现象用自己的话说一遍。
2. 把主购物车的 `:key="g.id"` 改成 `:key="index"`（注意 v-for 要加 index 参数），重复"加数量 → 删非末位商品"，验证 bug 复现。
3. 给购物车加个功能：商品单价改成可编辑的输入框（v-model.number + .trim），看看总价是否实时更新。
4. 思考题：为什么 `toFixed(2)` 要放在**展示层**而不是把数据本身改成字符串？（提示：后续还要做加减、优惠计算）
