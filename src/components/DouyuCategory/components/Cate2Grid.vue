<template>
  <div class="cate2-container">
    <div
      class="cate2-content"
      :class="{ 'is-expanded': isExpanded }"
      ref="cate2ContentRef"
    >
      <div class="cate2-scroll-wrapper" :class="{ 'allow-scroll': isExpanded && hasMoreRows }">
        <motion.div 
          class="cate2-grid" 
          ref="cate2GridRef"
          initial="hidden"
          animate="visible"
          :variants="{
            visible: {
              transition: {
                staggerChildren: 0.02
              }
            }
          }"
        >
          <motion.div
            v-for="cate2 in cate2List"
            :key="cate2.cate2Id"
            class="cate2-card"
            :class="{ 'active': selectedCate2Id === cate2.cate2Id }"
            @click="$emit('select', cate2)"
            :variants="{
              hidden: { opacity: 0, y: 10, scale: 0.95 },
              visible: { 
                opacity: 1, 
                y: 0, 
                scale: 1,
                transition: { type: 'spring', stiffness: 300, damping: 30 }
              }
            }"
            whileHover="{ scale: 1.04, y: -2 }"
            whileTap="{ scale: 0.96 }"
          >
            <div class="cate2-icon">
              <img :src="cate2.icon" :alt="cate2.cate2Name">
            </div>
            <div class="cate2-info">
              <div class="cate2-name" :title="cate2.cate2Name">{{ formatCategoryName(cate2.cate2Name) }}</div>
            </div>
          </motion.div>
        </motion.div>
      </div>
    </div>

    <div v-if="shouldShowExpandButtonComputed" class="expand-button" @click="handleToggleExpand">
      <span>{{ isExpanded ? '收起' : '展开' }}</span>
      <span class="expand-icon" :class="{ 'is-expanded': isExpanded }">
        <svg
          width="12"
          height="12"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
        >
          <path d="M6 9l6 6 6-6" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
        </svg>
      </span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch, nextTick, computed } from 'vue'
import { motion } from 'motion-v'
import type { Category2 } from '../types'

const props = defineProps<{
  cate2List: Category2[]
  selectedCate2Id: number | null
  isExpanded: boolean
  hasMoreRows: boolean
}>()

const emit = defineEmits<{
  (e: 'select', cate2: Category2): void
  (e: 'toggle-expand'): void
  (e: 'height-changed'): void
}>()

// Constants for height calculation
const CARD_ACTUAL_HEIGHT = 36;
const GRID_VERTICAL_GAP = 12;
const CONTENT_PADDING_BOTTOM = 8;
const GRID_INTERNAL_PADDING_BOTTOM = 18;

const TARGET_CONTENT_HEIGHT_FOR_ONE_ROW = CARD_ACTUAL_HEIGHT + GRID_INTERNAL_PADDING_BOTTOM + CONTENT_PADDING_BOTTOM;
const TARGET_CONTENT_HEIGHT_FOR_TWO_ROWS = (2 * CARD_ACTUAL_HEIGHT + GRID_VERTICAL_GAP) + GRID_INTERNAL_PADDING_BOTTOM + CONTENT_PADDING_BOTTOM - 14;
const EXPANDED_CONTENT_MAX_ROWS = 7;
const TARGET_CONTENT_HEIGHT_FOR_EXPANDED_MAX_ROWS = (EXPANDED_CONTENT_MAX_ROWS * CARD_ACTUAL_HEIGHT + (EXPANDED_CONTENT_MAX_ROWS - 1) * GRID_VERTICAL_GAP) + GRID_INTERNAL_PADDING_BOTTOM + CONTENT_PADDING_BOTTOM;

const cate2ContentRef = ref<HTMLElement | null>(null)
const cate2GridRef = ref<HTMLElement | null>(null)
const actualGridScrollHeight = ref(0)

const getCurrentTargetHeight = (expandedState: boolean) => {
  const naturalContentHeight = actualGridScrollHeight.value + CONTENT_PADDING_BOTTOM;
  if (expandedState) {
    if (props.hasMoreRows) return TARGET_CONTENT_HEIGHT_FOR_EXPANDED_MAX_ROWS;
    return props.cate2List.length > 0 ? naturalContentHeight : GRID_INTERNAL_PADDING_BOTTOM + CONTENT_PADDING_BOTTOM;
  } else {
    if (naturalContentHeight <= TARGET_CONTENT_HEIGHT_FOR_ONE_ROW) return naturalContentHeight;
    return TARGET_CONTENT_HEIGHT_FOR_TWO_ROWS;
  }
};

const updateActualGridScrollHeight = () => {
  nextTick(() => {
    if (cate2GridRef.value) actualGridScrollHeight.value = cate2GridRef.value.scrollHeight;
    else actualGridScrollHeight.value = GRID_INTERNAL_PADDING_BOTTOM;
    
    if (cate2ContentRef.value) {
      cate2ContentRef.value.style.height = `${getCurrentTargetHeight(props.isExpanded)}px`;
      emit('height-changed');
    }
  });
};

watch(() => props.cate2List, updateActualGridScrollHeight, { deep: true });
onMounted(updateActualGridScrollHeight);

const shouldShowExpandButtonComputed = computed(() => {
  const requiredHeight = actualGridScrollHeight.value + CONTENT_PADDING_BOTTOM;
  return requiredHeight > TARGET_CONTENT_HEIGHT_FOR_TWO_ROWS || props.hasMoreRows;
});

watch(() => props.isExpanded, (newValue) => {
  if (!cate2ContentRef.value) return;
  const targetHeight = getCurrentTargetHeight(newValue);
  cate2ContentRef.value.style.height = `${targetHeight}px`;
  
  // Wait for transition
  setTimeout(() => emit('height-changed'), 400);
});

const handleToggleExpand = () => emit('toggle-expand');

const formatCategoryName = (name: string) => {
  if (!name) return '';
  const getStringLength = (str: string) => {
    let len = 0;
    for (let i = 0; i < str.length; i++) {
      len += str.charCodeAt(i) > 127 || str.charCodeAt(i) === 94 ? 1 : 0.5;
    }
    return Math.ceil(len);
  };
  if (getStringLength(name) <= 5) return name;
  let result = '', currentLength = 0;
  for (let i = 0; i < name.length; i++) {
    const charLength = name.charCodeAt(i) > 127 || name.charCodeAt(i) === 94 ? 1 : 0.5;
    if (currentLength + charLength <= 4.5) {
      result += name[i];
      currentLength += charLength;
    } else break;
  }
  return result + '...';
}
</script>

<style scoped>
.cate2-container {
  padding: 10px 24px 8px;
  display: flex;
  flex-direction: column;
  flex: 1;
  position: relative;
  overflow: visible;
  background: transparent;
}

.cate2-content {
  position: relative;
  height: 0;
  padding-bottom: 8px;
  overflow: hidden;
  transition: height 0.4s cubic-bezier(0.16, 1, 0.3, 1);
  will-change: height;
}

.cate2-scroll-wrapper {
  height: 100%;
  overflow: hidden;
}

.cate2-content.is-expanded .cate2-scroll-wrapper.allow-scroll {
  overflow-y: auto !important;
  scrollbar-width: thin;
}

.cate2-scroll-wrapper.allow-scroll::-webkit-scrollbar {
  width: 4px;
}

.cate2-scroll-wrapper.allow-scroll::-webkit-scrollbar-thumb {
  background: var(--border-color);
  border-radius: 10px;
}

.cate2-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
  gap: 12px;
  justify-content: flex-start;
  padding-bottom: 20px;
}

.cate2-card {
  height: 38px;
  padding: 0 16px;
  border-radius: 100px;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--hover-bg);
  border: none;
  color: var(--secondary-text);
  box-shadow: none;
}

.cate2-card:hover {
  background: var(--hover-bg);
  color: var(--primary-text);
  transform: scale(1.05);
}

.cate2-card.active { 
  background: var(--glass-bg);
  color: var(--primary-text);
  font-weight: 700;
}

.cate2-icon {
  display: none;
}

.cate2-icon img {
  display: none;
}

.cate2-card.active .cate2-icon img {
  filter: none;
}

.cate2-card.active {
  /* theme-aware selected state uses glass background */
}

.cate2-info {
  flex: 1;
  overflow: hidden;
  text-align: center;
}

.cate2-name {
  font-size: 13px;
  font-weight: 600;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  text-align: center;
}

.expand-button {
  position: relative;
  align-self: center;
  margin-top: 6px;
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 4px 14px;
  font-size: 11px;
  font-weight: 600;
  color: var(--secondary-text);
  cursor: pointer;
  border-radius: 100px;
  background: var(--secondary-bg);
  border: none;
  transition: all 0.2s ease;
  z-index: 5;
  box-shadow: none;
}

.expand-button:hover {
  background: var(--tertiary-bg);
  color: var(--primary-text);
  border-color: var(--accent-color);
}

.expand-icon {
  margin-left: 2px;
  width: 12px;
  height: 12px;
  transition: transform 0.4s cubic-bezier(0.33, 0.66, 0.66, 1);
  display: inline-flex;
  align-items: center;
  justify-content: center;
}

.expand-icon.is-expanded {
  transform: rotate(180deg);
}
</style>
