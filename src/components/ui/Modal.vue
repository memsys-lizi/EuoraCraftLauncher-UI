<template>
  <Teleport to=".main-content">
    <Transition name="modal">
      <div v-if="visible" class="modal-overlay" @click="handleOverlayClick">
        <div
          ref="modalRef"
          class="modal-container"
          tabindex="-1"
          role="dialog"
          aria-modal="true"
          :aria-labelledby="title ? titleId : undefined"
          @click.stop
        >
          <div class="modal-header" v-if="$slots.header || title">
            <slot name="header">
              <h3 :id="titleId" class="modal-title">{{ title }}</h3>
            </slot>
            <button v-if="closable" class="modal-close" @click="close">
              <i class="icon icon-close"></i>
            </button>
          </div>
          
          <div class="modal-body">
            <slot></slot>
          </div>
          
          <div class="modal-footer" v-if="$slots.footer">
            <slot name="footer"></slot>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup lang="ts">
import { nextTick, onBeforeUnmount, ref, watch } from 'vue'

interface Props {
  visible: boolean
  title?: string
  closable?: boolean
  maskClosable?: boolean
}

interface Emits {
  (e: 'update:visible', value: boolean): void
  (e: 'close'): void
}

const props = withDefaults(defineProps<Props>(), {
  closable: true,
  maskClosable: true
})

const emit = defineEmits<Emits>()
const modalRef = ref<HTMLElement | null>(null)
const titleId = `modal-title-${Math.random().toString(36).slice(2, 9)}`
let previousFocusedElement: HTMLElement | null = null

const close = () => {
  emit('update:visible', false)
  emit('close')
}

const handleOverlayClick = () => {
  if (props.maskClosable) {
    close()
  }
}

const focusFirstElement = async () => {
  await nextTick()
  const modal = modalRef.value
  if (!modal) return

  const focusableElements = modal.querySelectorAll<HTMLElement>(
    'button:not([disabled]), [href], input:not([disabled]), select:not([disabled]), textarea:not([disabled]), [tabindex]:not([tabindex="-1"])',
  )

  if (focusableElements.length > 0) {
    focusableElements[0].focus()
    return
  }

  modal.focus()
}

const handleDocumentKeydown = (event: KeyboardEvent) => {
  if (!props.visible) return

  if (event.key === 'Escape' && props.closable) {
    event.preventDefault()
    close()
    return
  }

  if (event.key !== 'Tab' || !modalRef.value) return

  const focusableElements = Array.from(
    modalRef.value.querySelectorAll<HTMLElement>(
      'button:not([disabled]), [href], input:not([disabled]), select:not([disabled]), textarea:not([disabled]), [tabindex]:not([tabindex="-1"])',
    ),
  )

  if (focusableElements.length === 0) {
    event.preventDefault()
    modalRef.value.focus()
    return
  }

  const firstElement = focusableElements[0]
  const lastElement = focusableElements[focusableElements.length - 1]
  const activeElement = document.activeElement as HTMLElement | null

  if (event.shiftKey && activeElement === firstElement) {
    event.preventDefault()
    lastElement.focus()
  } else if (!event.shiftKey && activeElement === lastElement) {
    event.preventDefault()
    firstElement.focus()
  }
}

watch(
  () => props.visible,
  async (visible) => {
    if (visible) {
      previousFocusedElement = document.activeElement as HTMLElement | null
      document.addEventListener('keydown', handleDocumentKeydown)
      await focusFirstElement()
    } else {
      document.removeEventListener('keydown', handleDocumentKeydown)
      previousFocusedElement?.focus?.()
    }
  },
)

onBeforeUnmount(() => {
  document.removeEventListener('keydown', handleDocumentKeydown)
})
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  -webkit-backdrop-filter: blur(4px);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  opacity: 1;
}

.modal-container {
  background: var(--bg-surface);
  border-radius: var(--radius-lg);
  border: 1px solid var(--border-color);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
  min-width: min(400px, calc(100vw - 32px));
  max-width: 90vw;
  max-height: 90vh;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  outline: none;
  opacity: 1;
  transform: translateY(0) scale(1);
  transform-origin: center center;
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20px 24px;
  border-bottom: 1px solid var(--border-color);
}

.modal-title {
  margin: 0;
  font-size: 18px;
  font-weight: 600;
  color: var(--text-primary);
}

.modal-close {
  background: none;
  border: none;
  color: var(--text-secondary);
  cursor: pointer;
  padding: 4px;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-close:hover {
  background: var(--hover-bg);
  color: var(--text-primary);
}

.modal-close:focus-visible {
  outline: 2px solid var(--color-primary);
  outline-offset: 2px;
}

.modal-body {
  flex: 1;
  padding: 24px;
  overflow-y: auto;
}

.modal-footer {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 12px;
  padding: 16px 24px;
  border-top: 1px solid var(--border-color);
}

/* 模态框动画 */
.modal-enter-active,
.modal-leave-active {
  transition:
    opacity 0.22s ease,
    background-color 0.22s ease,
    backdrop-filter 0.22s ease,
    -webkit-backdrop-filter 0.22s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
  background-color: rgba(0, 0, 0, 0);
  -webkit-backdrop-filter: blur(0);
  backdrop-filter: blur(0);
}

.modal-enter-active .modal-container,
.modal-leave-active .modal-container {
  transition:
    opacity 0.22s ease,
    transform 0.28s cubic-bezier(0.22, 1, 0.36, 1);
}

.modal-enter-from .modal-container {
  opacity: 0;
  transform: translateY(12px) scale(0.985);
}

.modal-leave-to .modal-container {
  opacity: 0;
  transform: translateY(8px) scale(0.985);
}
</style>