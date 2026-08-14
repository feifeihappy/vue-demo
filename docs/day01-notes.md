# 第1天 · 环境与项目初始化（学习笔记）

> 日期：2026-08-14（补 8.11 第1天内容）· 学习计划第 1 天

## ✅ 今日完成

- 环境确认：Node v24.15.0 / npm 11.12.1 / git 2.54.0
- 用 `create-vue`（Vue 官方脚手架）初始化 **Vite + Vue 3** 项目 `vue-demo`
- `npm install` 安装依赖（151 个包）
- `npm run dev` 启动开发服务器：**http://localhost:5174/**（5173 被其他项目占用，显式指定了 5174）
- 初始化 Git 仓库并提交 `f180be0 initial commit`

## 🖥 如何自己搭建（以后换电脑可照着做）

```bash
# 1. 安装 Node.js（建议 LTS 版本），验证：
node -v   # v24.15.0
npm -v    # 11.12.1

# 2. 用官方脚手架创建项目（交互式选择功能）：
npm create vue@latest
# 提示项说明：
#   - TypeScript：勾选（现在是官方默认，第8.27天会用到）
#   - Router / Pinia / Vitest / ESLint：第1天可以先不选，后续按计划再加

# 3. 进入目录装依赖：
cd vue-demo
npm install

# 4. 启动开发服务器：
npm run dev   # 默认 http://localhost:5173，端口被占用会自动 +1

# 5. （可选）初始化 Git：
git init && git add -A && git commit -m "initial commit"
```

## 🗂 项目结构

```
vue-demo/
├── index.html          # 唯一的 HTML 入口，Vite 加载它的地方
├── package.json        # 依赖清单 + 脚本命令（dev/build/preview）
├── vite.config.ts      # Vite 配置（插件、@ 路径别名等）
├── tsconfig*.json      # TypeScript 编译配置
├── env.d.ts            # TS 环境声明（让 .vue 文件可被 TS 识别）
├── public/             # 静态资源（favicon.ico 等，原样拷贝）
└── src/                # 源码目录（重点）
    ├── main.ts         # 应用入口：创建并挂载 Vue 应用
    ├── App.vue         # 根组件：整个应用的"首页外壳"
    ├── assets/         # 静态资源（CSS、图片）
    └── components/     # 组件（HelloWorld、TheWelcome 等示例）
```

## 🔍 main.ts 逐行讲解

```ts
import './assets/main.css'        // ① 引入全局样式（所有组件都能用到）

import { createApp } from 'vue'   // ② 从 vue 包导入 createApp 函数（工厂函数）
import App from './App.vue'       // ③ 导入根组件 App.vue

createApp(App).mount('#app')      // ④ 创建应用实例，挂载到 <div id="app">
```

| 行 | 作用 | 理解要点 |
|---|---|---|
| ① | 引入全局 CSS | 副作用导入：浏览器加载样式，不需要变量名 |
| ② | 导入 `createApp` | Vue 3 通过"创建应用实例"的方式启动，不再是 Vue 2 的 `new Vue()` |
| ③ | 导入根组件 | `.vue` 是单文件组件（SFC），浏览器不认识，由 Vite 编译成 JS |
| ④ | 创建 + 挂载 | `createApp(App)` 生成应用对象 → `.mount('#app')` 找到 `index.html` 里的 `#app` 元素，把组件树渲染进去 |

**一句话总结**：`main.ts` 是发动机的点火开关——创建一个 Vue 应用，把根组件 `App.vue` 塞进页面上的 `#app` 容器里。

## 🔍 App.vue 逐行讲解

`.vue` 单文件组件 = 一个组件一个文件，分成三个块：

```vue
<!-- ① 逻辑部分（script setup = 组合式 API 的简写） -->
<script setup lang="ts">
import HelloWorld from './components/HelloWorld.vue'  // 导入子组件
import TheWelcome from './components/TheWelcome.vue'  // 导入子组件
</script>

<!-- ② 模板部分：写 HTML，组件可以像标签一样使用 -->
<template>
  <header>
    <img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />
    <div class="wrapper">
      <HelloWorld msg="You did it!" />   <!-- 使用组件 + 传 prop（msg） -->
    </div>
  </header>

  <main>
    <TheWelcome />
  </main>
</template>

<!-- ③ 样式部分：scoped = 样式只作用于本组件，不会污染全局 -->
<style scoped>
header { line-height: 1.5; }
/* ... */
</style>
```

| 块 | 作用 | 关键点 |
|---|---|---|
| `<script setup lang="ts">` | 组件的 JS/TS 逻辑 | 这里 `import` 的组件、声明的变量，**模板里直接用**，无需注册 |
| `<template>` | 组件的 HTML 结构 | 组件名当作自定义标签；`msg="..."` 是往子组件传数据（props） |
| `<style scoped>` | 组件的私有样式 | 加 `scoped` 后，Vue 会给元素自动加 `data-v-xxx` 属性来隔离样式 |

**一句话总结**：`App.vue` 是"首页外壳"，负责摆放子组件（HelloWorld、TheWelcome），并给它们传参、配样式。

## 🧩 index.html 的作用（经常被忽略，但很重要）

```html
<div id="app"></div>                        <!-- Vue 挂载的"插座" -->
<script type="module" src="/src/main.ts"></script>  <!-- 入口脚本 -->
```

- 浏览器加载页面 → 执行 `/src/main.ts` → Vue 应用启动 → 把内容渲染进 `<div id="app">`。
- 开发模式下 Vite 拦截 `/src/main.ts` 并**实时编译** `.vue` 文件，这就是热更新（HMR）的起点。

## 🚀 常用命令

| 命令 | 作用 |
|---|---|
| `npm run dev` | 启动开发服务器（带热更新） |
| `npm run build` | 先类型检查再打包，产物在 `dist/`（生产环境用） |
| `npm run preview` | 本地预览打包产物 |
| `npm run type-check` | 只做 TS 类型检查 |

## ⚠️ 今天踩的坑（经验值 +3）

1. **端口占用**：5173 被另一个项目占用了，Vite 默认会自动换端口；想固定端口用
   `npm run dev -- --port 5174 --strictPort`。
2. **`create-vue --force` 会清空目标目录**：如果 npm 缓存放在目标目录里，会连脚手架自身模板一起删掉。解决：缓存放目录外，或先让目录保持空。
3. **TypeScript 现在是 create-vue 默认**：`--default` 也会带 TS。所以 8.27 那天的计划从"给旧代码加 TS"调整为"类型增强 + 严格模式检查"。

## 📌 小作业（10 分钟，巩固今天内容）

1. 打开 http://localhost:5174/ ，把 `src/App.vue` 里的 `<h1>` 文字改成你的名字，**不要刷新页面**，观察热更新（HMR）效果。
2. 把 `<HelloWorld msg="You did it!" />` 的 msg 改成别的字符串，看页面变化，体会"数据从父组件流入子组件（props）"。
3. 用浏览器开发者工具看 `index.html` 里的 `#app` 元素，验证 Vue 把内容渲染进去了。
