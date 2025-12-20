<template>
  <div class="cate1-list">
    <div 
      v-for="cate1 in cate1List" 
      :key="cate1.cate1Id"
      class="cate1-item"
      :class="{ active: selectedCate1Id === cate1.cate1Id }"
      @click="$emit('select', cate1.cate1Id)"
    >
      <span class="cate1-name">{{ cate1.cate1Name }}</span>
    </div>
  </div>
</template>

<script setup lang="ts">
import type { Category1 } from '../types'

defineProps<{
  cate1List: Category1[]
  selectedCate1Id: number | null
}>()

defineEmits<{
  (e: 'select', cate1Id: number): void
}>()
</script>

<style scoped>
.cate1-list {
  padding: 12px 0 6px;
  display: flex;
  gap: 16px;
  overflow-x: auto;
  background: transparent;
  flex-shrink: 0;
  transition: all 0.2s ease;
  padding-left: 24px;
  padding-right: 24px;
}

.cate1-item {
  height: 38px;
  padding: 0 16px;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  font-size: 13px;
  font-weight: 600;
  border-radius: 100px;
  white-space: nowrap;
  background: var(--hover-bg);
  color: var(--secondary-text);
  display: inline-flex;
  align-items: center;
  box-shadow: none;
}

.cate1-item:hover {
  background: var(--hover-bg);
  color: var(--primary-text);
  transform: translateY(-1px);
}

.cate1-item.active {
  background: var(--glass-bg);
  color: var(--primary-text);
  font-weight: 700;
}

.cate1-list::-webkit-scrollbar {
  height: 0;
}
</style>
