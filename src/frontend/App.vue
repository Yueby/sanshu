<script setup lang="ts">
import { onMounted, onUnmounted, ref, watch } from 'vue'
import AppContent from './components/AppContent.vue'
import { useAppManager } from './composables/useAppManager'
import { useEventHandlers } from './composables/useEventHandlers'
import { useVersionCheck } from './composables/useVersionCheck'

const {
  naiveTheme,
  mcpRequest,
  showMcpPopup,
  appConfig,
  isInitializing,
  isIconMode,
  iconParams,
  actions,
} = useAppManager()

const handlers = useEventHandlers(actions)

const { updateInfo, updateDone, isUpdating, downloadProgress, checkForUpdate, downloadAndApply, confirmRestart } = useVersionCheck()
const showUpdateConfirm = ref(false)
const showRestartConfirm = ref(false)

function handleUpdateClick() {
  showUpdateConfirm.value = true
}

async function handleConfirmUpdate() {
  showUpdateConfirm.value = false
  await downloadAndApply()
  if (updateDone.value)
    showRestartConfirm.value = true
}

function setRootScrollLock(locked: boolean) {
  document.documentElement.style.overflow = locked ? 'hidden' : ''
  document.body.style.overflow = locked ? 'hidden' : ''
}

// 仅在三术弹窗模式锁定根滚动，避免双纵向滚动条
watch(showMcpPopup, (v) => {
  setRootScrollLock(!!v)
}, { immediate: true })

// 初始化
onMounted(async () => {
  try {
    await actions.app.initialize()
  }
  catch (error) {
    console.error('应用初始化失败:', error)
  }

  checkForUpdate()
})

// 清理
onUnmounted(() => {
  setRootScrollLock(false)
  actions.app.cleanup()
})
</script>

<template>
  <div :class="showMcpPopup ? 'h-screen overflow-hidden bg-surface transition-colors duration-200' : 'min-h-screen bg-surface transition-colors duration-200'">
    <n-config-provider :theme="naiveTheme">
      <n-message-provider>
        <n-notification-provider>
          <n-dialog-provider>
            <AppContent
              :mcp-request="mcpRequest" 
              :show-mcp-popup="showMcpPopup" 
              :app-config="appConfig"
              :is-initializing="isInitializing" 
              :is-icon-mode="isIconMode"
              :icon-params="iconParams"
              @mcp-response="handlers.onMcpResponse" 
              @mcp-cancel="handlers.onMcpCancel"
              @theme-change="handlers.onThemeChange" 
              @toggle-always-on-top="handlers.onToggleAlwaysOnTop"
              @toggle-audio-notification="handlers.onToggleAudioNotification"
              @update-audio-url="handlers.onUpdateAudioUrl" 
              @test-audio="handlers.onTestAudio"
              @stop-audio="handlers.onStopAudio" 
              @test-audio-error="handlers.onTestAudioError"
              @update-window-size="handlers.onUpdateWindowSize"
              @update-reply-config="handlers.onUpdateReplyConfig" 
              @message-ready="handlers.onMessageReady"
              @config-reloaded="handlers.onConfigReloaded"
              @update-click="handleUpdateClick"
            />
          </n-dialog-provider>
        </n-notification-provider>
      </n-message-provider>

      <!-- 确认更新 -->
      <n-modal
        v-model:show="showUpdateConfirm"
        preset="dialog"
        title="发现新版本"
        :content="`v${updateInfo?.latest_version} 可用，是否立即更新？`"
        positive-text="立即更新"
        negative-text="跳过"
        type="info"
        @positive-click="handleConfirmUpdate"
        @negative-click="showUpdateConfirm = false"
      />

      <!-- 下载进度 -->
      <n-modal :show="isUpdating" :mask-closable="false" :closable="false" preset="dialog" title="正在更新" type="info">
        <div class="py-2">
          <div class="text-sm mb-2 opacity-70">
            正在下载 v{{ updateInfo?.latest_version }}...
          </div>
          <n-progress type="line" :percentage="downloadProgress" :show-indicator="false" status="info" :height="6" />
          <div class="text-xs mt-1 text-right opacity-50">
            {{ downloadProgress }}%
          </div>
        </div>
      </n-modal>

      <!-- 重启确认 -->
      <n-modal
        v-model:show="showRestartConfirm"
        preset="dialog"
        title="更新完成"
        content="新版本已下载完成，是否立即重启应用？"
        positive-text="立即重启"
        negative-text="稍后"
        type="success"
        @positive-click="confirmRestart"
        @negative-click="showRestartConfirm = false"
      />
    </n-config-provider>
  </div>
</template>

