<template>
  <div class="min-h-screen flex flex-col bg-gradient-to-br from-gray-50 via-gray-50 to-gray-100
    dark:from-gray-900 dark:via-gray-900 dark:to-gray-800 transition-colors duration-300">
    <div class="flex-1 p-3 sm:p-8">
      <main class="max-w-7xl mx-auto space-y-8">
        <Header
          :title="title"
          :is-refreshing="isRefreshing"
          :is-dark="isDark"
          :theme-mode="themeMode"
          :last-updated="lastUpdated"
          @refresh="refreshData"
          @toggle-theme="toggleTheme"
        />

        <Stats :monitors="monitors" :loading="initialLoading" />

        <!-- 后台刷新失败但仍有旧数据时的提示 -->
        <Transition
          enter-active-class="transition-all duration-200 ease-out"
          enter-from-class="opacity-0 -translate-y-2"
          enter-to-class="opacity-100 translate-y-0"
          leave-active-class="transition-all duration-200 ease-in"
          leave-from-class="opacity-100 translate-y-0"
          leave-to-class="opacity-0 -translate-y-2"
        >
          <div v-if="noticeMessage"
               class="flex items-center justify-between gap-4 px-4 py-3 rounded-xl
                 bg-amber-50 dark:bg-amber-900/20
                 border border-amber-200 dark:border-amber-800/50
                 text-amber-700 dark:text-amber-300 text-sm"
          >
            <div class="flex items-center gap-2 min-w-0">
              <Icon icon="carbon:warning-alt" class="w-5 h-5 shrink-0" aria-hidden="true" />
              <span class="truncate">{{ noticeMessage }}</span>
            </div>
            <button
              type="button"
              class="shrink-0 p-1 rounded-full hover:bg-amber-100 dark:hover:bg-amber-900/40 transition-colors"
              aria-label="关闭提示"
              @click="dismissNotice"
            >
              <Icon icon="carbon:close" class="w-4 h-4" aria-hidden="true" />
            </button>
          </div>
        </Transition>

        <Card
          :monitors="monitors"
          :error="errorMessage"
          :loading="initialLoading"
        />
      </main>
    </div>
    <Footer />
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { Icon } from '@iconify/vue'
import { fetchMonitorData } from './utils/api'
import Header from './components/Header.vue'
import Stats from './components/Stats.vue'
import Card from './components/Card.vue'
import Footer from './components/Footer.vue'

const title = ref(import.meta.env.VITE_APP_TITLE || '状态监控')
const monitors = ref([])
const isRefreshing = ref(false)
const errorMessage = ref('')
const noticeMessage = ref('')
const lastUpdated = ref(null)
const initialLoading = ref(true)

/**
 * 主题模式：light / dark / system
 * 兼容旧的 localStorage 'theme' 键
 */
let storedMode = 'system'
try {
  storedMode = localStorage.getItem('theme-mode') || localStorage.getItem('theme') || 'system'
} catch (e) {
  storedMode = 'system'
}
const themeMode = ref(['light', 'dark', 'system'].includes(storedMode) ? storedMode : 'system')
const systemDark = ref(false)
let themeMedia = null

const isDark = computed(() => {
  if (themeMode.value === 'dark') return true
  if (themeMode.value === 'light') return false
  return systemDark.value
})

const applyTheme = () => {
  const dark = isDark.value
  document.documentElement.classList.toggle('dark', dark)
  document.documentElement.style.colorScheme = dark ? 'dark' : 'light'
  updateThemeColor(dark)
}

const updateThemeColor = (dark) => {
  let meta = document.querySelector('meta[name="theme-color"]')
  if (!meta) {
    meta = document.createElement('meta')
    meta.name = 'theme-color'
    document.head.appendChild(meta)
  }
  meta.content = dark ? '#111827' : '#f9fafb'
}

const initTheme = () => {
  systemDark.value = window.matchMedia('(prefers-color-scheme: dark)').matches
  applyTheme()
}

const onSystemThemeChange = (event) => {
  systemDark.value = event.matches
  if (themeMode.value === 'system') {
    applyTheme()
  }
}

const toggleTheme = () => {
  if (themeMode.value === 'system') {
    themeMode.value = 'light'
  } else if (themeMode.value === 'light') {
    themeMode.value = 'dark'
  } else {
    themeMode.value = 'system'
  }
  try {
    localStorage.setItem('theme-mode', themeMode.value)
    localStorage.removeItem('theme')
  } catch (e) {
    // 忽略隐私模式等场景下的存储异常
  }
  applyTheme()
}

const dismissNotice = () => {
  noticeMessage.value = ''
}

/**
 * 刷新逻辑：保留旧数据，避免整页闪烁
 */
const refreshData = async () => {
  if (isRefreshing.value) return

  const hasOldData = monitors.value.length > 0
  isRefreshing.value = true
  if (!hasOldData) errorMessage.value = ''

  const timeoutPromise = new Promise((_, reject) =>
    setTimeout(() => reject(new Error('Timeout')), 30000)
  )

  try {
    monitors.value = await Promise.race([
      fetchMonitorData(),
      timeoutPromise
    ])
    errorMessage.value = ''
    noticeMessage.value = ''
    lastUpdated.value = new Date()
  } catch (error) {
    console.error('获取监控数据失败:', error)
    const message = error.message === 'Timeout'
      ? '请求超时，请检查网络连接后重试'
      : '获取监控数据失败，请稍后重试'
    if (hasOldData) {
      noticeMessage.value = `${message}，当前显示上次成功获取的数据`
    } else {
      errorMessage.value = message
    }
  } finally {
    isRefreshing.value = false
    initialLoading.value = false
  }
}

onMounted(() => {
  initTheme()
  themeMedia = window.matchMedia('(prefers-color-scheme: dark)')
  if (typeof themeMedia.addEventListener === 'function') {
    themeMedia.addEventListener('change', onSystemThemeChange)
  } else if (typeof themeMedia.addListener === 'function') {
    themeMedia.addListener(onSystemThemeChange)
  }
  refreshData()
})

onUnmounted(() => {
  if (themeMedia) {
    if (typeof themeMedia.removeEventListener === 'function') {
      themeMedia.removeEventListener('change', onSystemThemeChange)
    } else if (typeof themeMedia.removeListener === 'function') {
      themeMedia.removeListener(onSystemThemeChange)
    }
  }
})
</script>
