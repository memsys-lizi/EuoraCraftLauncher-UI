<template>
  <div class="ui-input-wrapper" :class="{ 'is-focused': isFocused, 'is-disabled': disabled }">
    <i v-if="leadingIcon" :class="['icon', leadingIcon, 'prefix-icon']" />
    
    <input
      ref="inputRef"
      :id="id"
      :value="modelValue"
      :type="type"
      :placeholder="placeholder"
      :disabled="disabled"
      :readonly="readonly"
      :aria-label="ariaLabel"
      class="ui-input"
      @input="handleInput"
      @focus="isFocused = true"
      @blur="isFocused = false"
      @keydown.enter="$emit('enter')"
    />
    
    <button
      v-if="clearable && modelValue" 
      type="button"
      class="clear-icon"
      aria-label="清空输入"
      @click="handleClear"
    >
      <i class="icon icon-close" />
    </button>
    
    <i v-if="suffixIcon" :class="['icon', suffixIcon, 'suffix-icon']" />
  </div>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'

const props = withDefaults(defineProps<{
  modelValue: string | number
  id?: string
  type?: string
  placeholder?: string
  disabled?: boolean
  readonly?: boolean
  clearable?: boolean
  icon?: string
  prefixIcon?: string
  suffixIcon?: string
  ariaLabel?: string
}>(), {
  type: 'text',
  modelValue: '',
  disabled: false,
  readonly: false,
  clearable: false
})

const emit = defineEmits<{
  (e: 'update:modelValue', value: string | number): void
  (e: 'enter'): void
  (e: 'clear'): void
}>()

const inputRef = ref<HTMLInputElement | null>(null)
const isFocused = ref(false)
const leadingIcon = computed(() => props.prefixIcon || props.icon)

const handleInput = (event: Event) => {
  const target = event.target as HTMLInputElement
  emit('update:modelValue', target.value)
}

const handleClear = () => {
  emit('update:modelValue', '')
  emit('clear')
  inputRef.value?.focus()
}
</script>

<style scoped>
.ui-input-wrapper {
  display: flex;
  align-items: center;
  gap: 8px;
  min-height: 40px;
  padding: 8px 12px;
  background-color: var(--bg-surface);
  border: 1px solid var(--border-color);
  border-radius: var(--radius-md);
  transition: border-color 0.2s ease, background-color 0.2s ease, box-shadow 0.2s ease;
  width: 100%;
  position: relative;
}

.ui-input-wrapper:hover:not(.is-disabled) {
  border-color: var(--text-secondary);
  background-color: var(--bg-surface);
}

.ui-input-wrapper.is-focused {
  border-color: var(--color-primary);
  background-color: var(--bg-surface-active);
  box-shadow: 0 0 0 3px var(--color-primary-light);
}

.ui-input-wrapper.is-disabled {
  opacity: 0.6;
  cursor: not-allowed;
  background-color: var(--bg-surface-hover);
}

.ui-input {
  flex: 1;
  border: none;
  background: transparent;
  color: var(--text-primary);
  font-size: 14px;
  outline: none;
  width: 100%;
  padding: 0;
}

.ui-input::placeholder {
  color: var(--text-disabled);
}

.icon {
  font-size: 16px;
  color: var(--text-secondary);
  display: flex;
  align-items: center;
  justify-content: center;
}

.clear-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border: none;
  background: transparent;
  cursor: pointer;
  font-size: 14px;
  padding: 2px;
  border-radius: 50%;
  transition: all 0.2s ease;
  color: var(--text-secondary);
}

.clear-icon:hover {
  background-color: var(--bg-surface-active);
  color: var(--text-primary);
}

.clear-icon:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

.prefix-icon {
  margin-right: 4px;
}

.suffix-icon {
  margin-left: 4px;
}
</style>