<template>
  <!-- 加载骨架屏：先渲染 UI 框架，数据到了再显示真实统计 -->
  <div v-if="loading && !monitors.length" class="grid grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 2xl:grid-cols-5 gap-4 lg:gap-5 mb-6">
    <div
      v-for="i in 5"
      :key="i"
      class="card-base p-4 sm:p-5 animate-pulse"
    >
      <div class="flex items-center justify-between gap-3">
        <div class="flex-1 space-y-2.5">
          <div class="h-3 w-14 bg-gray-200 dark:bg-gray-700 rounded" />
          <div class="h-6 w-20 bg-gray-200 dark:bg-gray-700 rounded" />
          <div class="h-3 w-24 bg-gray-200 dark:bg-gray-700 rounded" />
        </div>
        <div class="w-9 h-9 sm:w-10 sm:h-10 rounded-lg bg-gray-200 dark:bg-gray-700 shrink-0" />
      </div>
    </div>
  </div>

  <!-- 统计卡片 -->
  <div v-else class="grid grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 2xl:grid-cols-5 gap-4 lg:gap-5 mb-6">
    <div
      v-for="(item, index) in overviewItems"
      :key="index"
      class="card-base animate-fade p-4 sm:p-5"
      :class="[item.containerClass]"
    >
      <div class="flex items-center justify-between gap-3 relative">
        <div class="min-w-0">
          <div class="text-xs sm:text-sm font-medium text-gray-500 dark:text-gray-400">
            {{ item.label }}
          </div>
          <div class="mt-1.5 sm:mt-2 text-xl sm:text-2xl font-bold text-gray-900 dark:text-gray-100 leading-tight">
            <span>{{ displayValues[index] }}</span><span v-if="item.unit" class="text-sm sm:text-base ml-0.5">{{ item.unit }}</span>
          </div>
          <div class="mt-1 text-xs text-gray-500 dark:text-gray-400 truncate">
            {{ item.desc }}
          </div>
        </div>
        <div class="p-2 sm:p-2.5 bg-gray-50 dark:bg-gray-800 rounded-lg shrink-0 self-center">
          <Icon
            :icon="item.icon"
            class="w-5 h-5 sm:w-6 sm:h-6 transition-colors duration-200"
            :class="item.iconColor"
            aria-hidden="true"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, ref, watch } from 'vue'
import { Icon } from '@iconify/vue'

const props = defineProps({
  monitors: {
    type: Array,
    default: () => []
  },
  loading: {
    type: Boolean,
    default: false
  }
})

/**
 * 状态统计口径：
 * - 在线：status 2
 * - 异常：status 9（离线）及其他未知异常状态
 * - 暂停/准备中：status 0 / 1
 */
const total = computed(() => props.monitors.length)
const online = computed(() => props.monitors.filter(m => m.status === 2).length)
const abnormal = computed(() => props.monitors.filter(m => m.status === 9 || m.status === undefined).length)
const pending = computed(() => props.monitors.filter(m => m.status === 0 || m.status === 1).length)

const avgResponse = computed(() => {
  if (!props.monitors?.length) return 0
  const onlineMonitors = props.monitors.filter(m =>
    m.status === 2 && Number.isFinite(m.stats?.avgResponseTime) && m.stats.avgResponseTime > 0
  )
  if (!onlineMonitors.length) return 0
  return Math.round(
    onlineMonitors.reduce((acc, m) => acc + m.stats.avgResponseTime, 0) / onlineMonitors.length
  )
})

const displayValues = ref([0, 0, 0, 0, 0])

const animateValue = (start, end, duration, index) => {
  const startTime = performance.now()
  const updateValue = (currentTime) => {
    const elapsed = currentTime - startTime
    const progress = Math.min(elapsed / duration, 1)
    displayValues.value[index] = Math.floor(start + (end - start) * progress)
    if (progress < 1) {
      requestAnimationFrame(updateValue)
    }
  }
  requestAnimationFrame(updateValue)
}

const overviewItems = computed(() => [
  {
    label: '监控网站',
    value: total.value,
    desc: '全部网站',
    icon: 'bi:check-circle',
    iconColor: 'text-emerald-500',
    containerClass: 'hover:border-emerald-500/70 hover:ring-2 hover:ring-emerald-500/40 dark:hover:border-emerald-400/70 dark:hover:ring-emerald-400/40'
  },
  {
    label: '正常网站',
    value: online.value,
    desc: '访问正常',
    icon: 'bi:check-circle-fill',
    iconColor: 'text-green-500',
    containerClass: 'hover:border-green-500/70 hover:ring-2 hover:ring-green-500/40 dark:hover:border-green-400/70 dark:hover:ring-green-400/40'
  },
  {
    label: '异常网站',
    value: abnormal.value,
    desc: '离线或未知状态',
    icon: 'bi:x-circle-fill',
    iconColor: 'text-red-500',
    containerClass: 'hover:border-red-500/70 hover:ring-2 hover:ring-red-500/40 dark:hover:border-red-400/70 dark:hover:ring-red-400/40'
  },
  {
    label: '暂停/准备中',
    value: pending.value,
    desc: '未在监控',
    icon: 'bi:pause-circle-fill',
    iconColor: 'text-yellow-500',
    containerClass: 'hover:border-yellow-500/70 hover:ring-2 hover:ring-yellow-500/40 dark:hover:border-yellow-400/70 dark:hover:ring-yellow-400/40'
  },
  {
    label: '平均响应',
    value: avgResponse.value,
    unit: 'ms',
    desc: '在线站点网络延迟',
    icon: 'bi:clock',
    iconColor: 'text-blue-500',
    containerClass: 'hover:border-blue-500/70 hover:ring-2 hover:ring-blue-500/40 dark:hover:border-blue-400/70 dark:hover:ring-blue-400/40'
  }
])

watch(
  () => overviewItems.value.map(item => item.value),
  (newValues, oldValues) => {
    newValues.forEach((newVal, index) => {
      const oldVal = oldValues?.[index] ?? 0
      if (newVal !== oldVal) {
        animateValue(oldVal, newVal, 800, index)
      }
    })
  },
  { immediate: true }
)
</script>
