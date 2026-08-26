# 第4天 · 条件渲染与列表渲染（学习笔记）

> 日期：2026-08-26（补 8.14 第4天内容）· 学习计划第 4 天

## ✅ 今日完成

- 创建演示组件 `src/components/ConditionRenderDemo.vue`，替换进 App.vue
- 亲手实验了 v-if / v-show 的 DOM 差异（DevTools 里验证：v-show 隐藏时元素还在）
- 完成 Todo 列表雏形：添加、删除、完成切换、三种过滤、空状态提示
- 用"乱序对比实验"直观理解了 key 的作用

## 一、v-if vs v-show（两者都控制显示/隐藏，机制完全不同）

| | v-if | v-show |
|---|---|---|
| 机制 | 条件为 false 时**销毁元素**，DOM 里不存在 | 只是加 `display: none`，元素一直在 DOM |
| 切换代价 | 创建/销毁，代价高 | 只改一个 CSS 属性，代价极低 |
| 初始渲染 | false 时不渲染（惰性），省初始开销 | 一开始就会渲染 |
| 搭配 | 有 v-else-if / v-else 分支伙伴 | 没有 |
| 适用 | 不常切换的场景（登录后显示用户面板） | 高频切换（选项卡、折叠面板） |

**一句话**：v-if 是"真条件"，v-show 是"伪装隐藏"。同一个开关按钮下用 DevTools 对比，感受最直观。

## 二、v-if / v-else-if / v-else

多分支时只会命中一个，从上往下找第一个条件为 true 的：

```vue
<p v-if="state === 'user'">🎉 欢迎回来</p>
<p v-else-if="state === 'loading'">⏳ 登录中…</p>
<p v-else>👋 请先登录</p>
```

要求：v-else-if / v-else 必须**紧挨着** v-if 所在的元素，中间不能有其他内容。

## 三、v-for 的四种形态

```vue
<!-- 1. 数组：可以再拿索引 -->
<li v-for="(item, index) in items" :key="item">{{ index }}. {{ item }}</li>

<!-- 2. 对象：先 value 后 key -->
<li v-for="(value, key) in obj" :key="key">{{ key }}: {{ value }}</li>

<!-- 3. 范围：从 1 开始 -->
<span v-for="n in 5" :key="n">{{ n }}</span>

<!-- 4. 数字范围不想要 1 开头？用计算式 -->
<span v-for="n in [3, 4, 5]" :key="n">{{ n }}</span>
```

## 四、key 的作用（今天最重要的知识点）⭐

**key 是给 Vue 的"身份标识"**，让 Vue 在更新列表时知道"哪个节点是哪个"。

没有 key 时，Vue 只能按**位置**复用 DOM 节点（就地更新）；有 key 时按**身份**复用。

**乱序实验**（组件第四部分有交互演示）：

1. 两列同样的列表（Alice / Bob / Carol），每项前面一个输入框
2. 先在输入框随便打字，再点"打乱顺序"
3. 观察结果：
   - ❌ 无 key 列：输入框内容**跟着位置走**，内容串到别人名字上去了
   - ✅ 有 key 列：输入框内容**跟着名字走**，正确跟随

```vue
<div v-for="name in list">             <!-- ❌ 位置复用：有状态的内容会错位 -->
<div v-for="name in list" :key="name"> <!-- ✅ 身份复用 -->
```

**什么时候会出错**：列表项内部有"状态"时（输入框内容、勾选、动画、子组件）。纯展示的静态列表无感，但养成习惯一律加 key。

**key 用什么值**：
- ✅ 用**稳定且唯一**的数据：`todo.id`、商品 sku
- ❌ 不要用 `index`：插入/删除时 index 会变，身份就乱了（列表头部插一条，后面所有项的身份集体错位）
- index 唯一可行的场景：纯展示、不增删、不改顺序的列表

## 五、v-for 和 v-if 不要写在同一元素上

```vue
<!-- ❌ 不推荐：v-if 优先级更高，循环变量 item 根本用不到 -->
<li v-for="item in list" v-if="item.visible" :key="item.id">

<!-- ✅ 正确：先用 computed 过滤，模板里只剩 v-for -->
const visibleList = computed(() => list.filter(i => i.visible))
<li v-for="item in visibleList" :key="item.id">
```

官方理由：v-if 的优先级比 v-for 高，写在一起时 v-if **访问不到** item。今天的 Todo 组件就是这么实践的（filteredTodos）。

## 六、Todo 列表雏形（把今天知识点全用上了）

组件第三部分是一个可用的 Todo 雏形：

- 输入 + 回车添加（`@keyup.enter`）
- 点击文字切换完成状态、✕ 删除
- 未完成数量用 `computed` 派生（第 3 天学的，复习）
- 全部 / 未完成 / 已完成三个过滤 tab，也是 computed
- 空列表显示"暂无待办"：`v-if` 配 `v-else`
- key 用 `todo.id`（新增时自增 id，唯一且稳定）

## 📌 小作业（15 分钟，巩固今天内容）

1. 打开页面，用 DevTools 的 Elements 面板分别验证：v-if 隐藏时元素真的没了，v-show 隐藏时元素还在。
2. 去组件第四部分，输入框里打字 → 点"打乱顺序" → 亲眼确认无 key 列错位、有 key 列不错位。
3. 改一下 Todo：在 `addTodo` 里给新待办**固定** `id: 99`，添加两条后点"删除"，观察渲染有没有异常（用现象体会 key 重复的后果）。
4. 思考题：过滤 tab 用 computed 实现了；如果不用 computed、直接在 `v-for` 上写 `v-if`，会报什么错？动手试试再改回来。
