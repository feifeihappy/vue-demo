<!--
  ProductCard.vue —— 8.18 组件基础：子组件
  演示：props（父 → 子）、emits（子 → 父）、组件自己的状态
-->
<script setup lang="ts">
import { ref } from 'vue'

interface Goods {
  id: number
  name: string
  price: number
}

// ① props：父组件传进来的数据。约定只读——数据的所有权在父组件
const props = defineProps<{
  product: Goods
  tag?: string // 可选 prop，演示静态字符串传值
}>()

// ② emits：声明子组件能对外发出的事件（子 → 父 的唯一通道）
const emit = defineEmits<{
  like: [id: number]
  add: [id: number]
}>()

// ③ 组件自己的状态：只有本组件用，父组件不关心、不参与
const favorited = ref(false)
function toggleFavorite() {
  favorited.value = !favorited.value
}

// 陷阱演示：直接改 props 的嵌套属性
// TS 的 Readonly 是浅的，只锁 props.product 这个引用，product.name 的类型仍可写
// 运行时 shallowReadonly 也只拦第一层 → 这一行真的会污染父组件的数据！
function tryModifyName() {
  props.product.name += '！'
}

// 正确姿势：想改数据？emit 事件上去，让父组件决定怎么改
function like() {
  emit('like', props.product.id)
}
function addCart() {
  emit('add', props.product.id)
}
</script>

<template>
  <div class="product-card">
    <div class="head">
      <b>{{ product.name }}</b>
      <span v-if="tag" class="tag">{{ tag }}</span>
    </div>
    <p class="price">¥{{ product.price.toFixed(2) }}</p>
    <div class="row">
      <button class="mini" :class="{ on: favorited }" @click="toggleFavorite">
        {{ favorited ? '❤️ 已收藏' : '🤍 收藏' }}
      </button>
      <button class="mini" @click="like">👍 点赞</button>
      <button class="mini" @click="addCart">🛒 加购</button>
    </div>
    <button class="bad" @click="tryModifyName">❌ 直接改 props.name（陷阱）</button>
    <p class="hint">❤️ 收藏是组件自己的状态 · props 数据归父组件</p>
  </div>
</template>

<style scoped>
.product-card {
  border: 1px solid #e5e5e5;
  border-radius: 10px;
  padding: 0.9rem 1rem;
  background: #fff;
}

.head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 6px;
}

.head b {
  font-size: 1rem;
}

.tag {
  font-size: 0.75rem;
  background: #fff4e6;
  color: #e8590c;
  border: 1px solid #ffd8a8;
  border-radius: 20px;
  padding: 1px 8px;
  white-space: nowrap;
}

.price {
  color: #e8590c;
  font-weight: 700;
  font-size: 1.15rem;
  margin: 0.5rem 0;
}

.row {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-bottom: 8px;
}

button {
  padding: 3px 10px;
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
button.on {
  background: #e8590c;
  color: #fff;
}
button.bad {
  border-color: #ff6b6b;
  color: #ff6b6b;
  font-size: 0.8rem;
}
button.bad:hover {
  background: #ff6b6b;
  color: #fff;
}

.hint {
  font-size: 0.75rem;
  color: #999;
  margin: 6px 0 0;
}
</style>
