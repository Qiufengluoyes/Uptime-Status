<template>
  <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-3">
    <div class="flex items-center gap-3 min-w-0">
      <img
        src="/logo.svg"
        alt="Logo"
        class="w-8 h-8 sm:w-10 sm:h-10 shrink-0"
      />
      <div class="min-w-0">
        <h1 class="text-lg sm:text-2xl font-bold text-gray-800 dark:text-gray-100 truncate">
          {{ title }}
        </h1>
        <p v-if="lastUpdated" class="text-xs text-gray-400 dark:text-gray-500 mt-0.5">
          上次更新 {{ formatDateTime(lastUpdated) }}
        </p>
      </div>
    </div>
    <div class="flex items-center gap-3 shrink-0">
      <button
        type="button"
        @click="refreshData"
        :disabled="isRefreshing"
        :aria-label="isRefreshing ? '正在刷新' : '立即刷新'"
        :title="isRefreshing ? '正在刷新' : '立即刷新'"
        class="flex items-center gap-2 px-3 h-9 rounded-full transition-all duration-200
          bg-emerald-50 dark:bg-emerald-900/30
          text-emerald-600 dark:text-emerald-400
          shadow-sm shadow-emerald-500/10 dark:shadow-emerald-900/20
          hover:bg-emerald-100 dark:hover:bg-emerald-900/40
          disabled:opacity-75 disabled:cursor-not-allowed disabled:shadow-none"
      >
        <Icon
          icon="ph:arrows-counter-clockwise-bold"
          class="w-4 h-4 transition-all"
          :class="isRefreshing ? 'animate-spin' : ''"
          aria-hidden="true"
        />
        <span class="hidden sm:block text-sm font-medium">
          {{ isRefreshing ? '刷新中' : `${formatTime(countdown)}后刷新` }}
        </span>
      </button>
      <button
        type="button"
        @click="toggleTheme"
        :aria-label="themeButtonLabel"
        :title="themeButtonLabel"
        class="flex items-center justify-center w-9 h-9 rounded-full transition-all duration-200
          bg-white dark:bg-gray-800
          text-gray-600 dark:text-gray-300
          shadow-sm shadow-gray-200/50 dark:shadow-gray-900/30
          hover:bg-gray-50 dark:hover:bg-gray-700"
      >
        <Icon
          :icon="themeIcon"
          class="w-5 h-5"
          aria-hidden="true"
        />
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { Icon } from '@iconify/vue'
import { format } from 'date-fns'

const formatTime = (seconds) => {
  const minutes = Math.floor(seconds / 60)
  const remainingSeconds = seconds % 60
  return `${minutes}:${remainingSeconds.toString().padStart(2, '0')}`
}

const formatDateTime = (date) => format(date, 'yyyy-MM-dd HH:mm:ss')

const props = defineProps({
  title: {
    type: String,
    required: true
  },
  isRefreshing: {
    type: Boolean,
    default: false
  },
  isDark: {
    type: Boolean,
    default: false
  },
  themeMode: {
    type: String,
    default: 'system'
  },
  lastUpdated: {
    type: [Date, String],
    default: null
  }
})

const emit = defineEmits(['refresh', 'toggle-theme'])

/**
 * 刷新间隔：5 分钟
 */
const REFRESH_INTERVAL = 300
const countdown = ref(REFRESH_INTERVAL)
let timer = null
let endTime = 0

const themeIcon = computed(() => {
  if (props.themeMode === 'system') return 'carbon:contrast'
  return props.themeMode === 'dark' ? 'bi:moon' : 'bi:sun'
})

const themeButtonLabel = computed(() => {
  if (props.themeMode === 'system') return '当前为跟随系统，点击切换为浅色'
  if (props.themeMode === 'light') return '当前为浅色模式，点击切换为深色'
  return '当前为深色模式，点击切换为跟随系统'
})

/**
 * 基于绝对时间戳的倒计时，避免后台标签页节流导致漂移
 */
const scheduleTick = () => {
  clearTimeout(timer)
  timer = setTimeout(tick, 1000)
}

const tick = () => {
  const remaining = Math.max(0, Math.ceil((endTime - Date.now()) / 1000))
  countdown.value = remaining

  if (remaining === 0 && !props.isRefreshing) {
    emit('refresh')
    endTime = Date.now() + REFRESH_INTERVAL * 1000
  }

  scheduleTick()
}

const startTimer = () => {
  clearTimeout(timer)
  countdown.value = REFRESH_INTERVAL
  endTime = Date.now() + REFRESH_INTERVAL * 1000
  scheduleTick()
}

const refreshData = () => {
  emit('refresh')
  countdown.value = REFRESH_INTERVAL
  endTime = Date.now() + REFRESH_INTERVAL * 1000
}

const toggleTheme = () => {
  emit('toggle-theme')
}

const handleVisibility = () => {
  if (!document.hidden) {
    tick()
  }
}

onMounted(() => {
  startTimer()
  document.addEventListener('visibilitychange', handleVisibility)
})

onUnmounted(() => {
  clearTimeout(timer)
  document.removeEventListener('visibilitychange', handleVisibility)
})
</script>
