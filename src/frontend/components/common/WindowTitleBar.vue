<script setup lang="ts">
import { invoke } from '@tauri-apps/api/core'
import { getCurrentWebviewWindow } from '@tauri-apps/api/webviewWindow'
import { computed, onMounted, ref } from 'vue'
import { useVersionCheck } from '../../composables/useVersionCheck'
import ThemeIcon from './ThemeIcon.vue'

interface Props {
  title?: string
  showMinimize?: boolean
  showClose?: boolean
  currentTheme?: string
  customClose?: boolean
  cancelCount?: number
}

interface Emits {
  themeChange: [theme: string]
  close: []
  updateClick: []
}

const props = withDefaults(defineProps<Props>(), {
  title: '三术',
  showMinimize: true,
  showClose: true,
  currentTheme: 'dark',
  customClose: false,
  cancelCount: -1,
})

const closeBtnClass = computed(() => {
  if (props.cancelCount == null || props.cancelCount < 0) return 'titlebar-close-btn'
  return props.cancelCount >= 1 ? 'titlebar-close-confirm' : 'titlebar-close-warn'
})

const closeTitle = computed(() => {
  if (props.cancelCount == null || props.cancelCount < 0) return '关闭'
  return props.cancelCount >= 1 ? '确认结束对话' : '关闭'
})

const emit = defineEmits<Emits>()

const { updateInfo } = useVersionCheck()

const appWindow = getCurrentWebviewWindow()
const alwaysOnTop = ref(false)

onMounted(async () => {
  try {
    alwaysOnTop.value = await invoke('get_always_on_top') as boolean
  }
  catch {}
})

async function handleToggleAlwaysOnTop() {
  try {
    const newValue = !alwaysOnTop.value
    await invoke('set_always_on_top', { enabled: newValue })
    alwaysOnTop.value = newValue
  }
  catch (e) {
    console.warn('置顶切换失败:', e)
  }
}

function handleThemeToggle() {
  const next = props.currentTheme === 'light' ? 'dark' : 'light'
  emit('themeChange', next)
}

async function handleMinimize() {
  await appWindow.minimize()
}

async function handleClose() {
  emit('close')
  if (!props.customClose)
    await appWindow.close()
}
</script>

<template>
  <div class="select-none" data-tauri-drag-region>
    <div class="flex items-center justify-between gap-2 px-3 py-1.5" data-tauri-drag-region>
      <!-- 左侧：标题 -->
      <div class="flex items-center gap-2 min-w-0" data-tauri-drag-region>
        <div class="w-2.5 h-2.5 rounded-full bg-primary-500 flex-shrink-0" />
        <span class="text-xs font-medium text-on-surface-secondary truncate" data-tauri-drag-region>
          {{ title }}
        </span>
      </div>

      <!-- 中间：自定义内容插槽 -->
      <div class="flex items-center gap-2 flex-1 justify-end" data-tauri-drag-region>
        <slot />

        <!-- 分隔线（仅在有插槽内容时可见） -->
        <div v-if="$slots.default" class="w-px h-3 shrink-0" style="background-color: color-mix(in srgb, var(--color-on-surface) 25%, transparent)" />

        <!-- 更新 + 置顶 + 主题切换 + 窗口控制 -->
        <n-space :size="2">
          <n-tooltip v-if="updateInfo?.available" trigger="hover">
            <template #trigger>
              <n-button
                size="tiny"
                quaternary
                circle
                class="update-badge"
                @click="emit('updateClick')"
              >
                <template #icon>
                  <div class="i-carbon-upgrade w-3.5 h-3.5" />
                </template>
              </n-button>
            </template>
            v{{ updateInfo.latest_version }} 可用
          </n-tooltip>
          <n-button
            size="tiny"
            quaternary
            circle
            :title="alwaysOnTop ? '取消置顶' : '窗口置顶'"
            @click="handleToggleAlwaysOnTop"
          >
            <template #icon>
              <div
                :class="alwaysOnTop ? 'i-carbon-pin-filled text-primary-500' : 'i-carbon-pin text-on-surface-secondary'"
                class="w-3.5 h-3.5"
              />
            </template>
          </n-button>
          <n-button
            size="tiny"
            quaternary
            circle
            :title="`切换到${currentTheme === 'light' ? '深色' : '浅色'}主题`"
            @click="handleThemeToggle"
          >
            <template #icon>
              <ThemeIcon :theme="currentTheme" class="w-3.5 h-3.5 text-on-surface-secondary" />
            </template>
          </n-button>
          <n-button
            v-if="showMinimize"
            size="tiny"
            quaternary
            circle
            title="最小化"
            @click="handleMinimize"
          >
            <template #icon>
              <div class="i-carbon-subtract w-3.5 h-3.5 text-on-surface-secondary" />
            </template>
          </n-button>
          <n-button
            v-if="showClose"
            size="tiny"
            quaternary
            circle
            :title="closeTitle"
            :class="closeBtnClass"
            @click="handleClose"
          >
            <template #icon>
              <div class="i-carbon-close w-3.5 h-3.5 text-on-surface-secondary" />
            </template>
          </n-button>
        </n-space>
      </div>
    </div>
  </div>
</template>

<style scoped>
.titlebar-close-btn:hover,
.titlebar-close-confirm:hover {
  background-color: color-mix(in srgb, var(--color-error) 80%, transparent) !important;
  color: #fff !important;
}
.titlebar-close-warn:hover {
  background-color: color-mix(in srgb, var(--color-warning) 80%, transparent) !important;
  color: #fff !important;
}
.update-badge {
  color: var(--color-success) !important;
  animation: pulse-subtle 2s ease-in-out infinite;
}
@keyframes pulse-subtle {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}
</style>
