<template>
  <!-- 骨架屏：先把 UI 框架显示出来 -->
  <div v-if="loading && !monitors?.length" class="space-y-6">
    <div class="h-12 rounded-xl bg-gray-200/70 dark:bg-gray-800/60 animate-pulse" />
    <div class="flex flex-col sm:flex-row gap-3">
      <div class="h-10 flex-1 min-w-[200px] rounded-lg bg-gray-200/70 dark:bg-gray-800/60 animate-pulse" />
      <div class="h-10 w-28 sm:w-32 rounded-lg bg-gray-200/70 dark:bg-gray-800/60 animate-pulse" />
      <div class="h-10 w-28 sm:w-32 rounded-lg bg-gray-200/70 dark:bg-gray-800/60 animate-pulse" />
    </div>
    <div class="grid gap-6 grid-cols-1 md:grid-cols-2 xl:grid-cols-3">
      <div
        v-for="i in 6"
        :key="i"
        class="card-base p-6 rounded-2xl animate-pulse"
      >
        <div class="flex items-center justify-between gap-4 mb-6">
          <div class="h-5 w-36 sm:w-44 bg-gray-200 dark:bg-gray-700 rounded" />
          <div class="h-8 w-20 bg-gray-200 dark:bg-gray-700 rounded-full" />
        </div>
        <div class="grid grid-cols-2 gap-3 mb-4">
          <div class="h-20 rounded-lg bg-gray-200 dark:bg-gray-700" />
          <div class="h-20 rounded-lg bg-gray-200 dark:bg-gray-700" />
        </div>
        <div class="h-24 rounded-lg bg-gray-200 dark:bg-gray-700" />
      </div>
    </div>
  </div>

  <!-- 加载失败 / 暂无数据 -->
  <div v-else-if="!monitors?.length" class="flex items-center justify-center p-12">
    <div v-if="!error"
         class="flex flex-col items-center gap-3 text-gray-400 dark:text-gray-500">
      <Icon
        icon="carbon:data-vis-4"
        class="w-12 h-12"
        aria-hidden="true"
      />
      <p class="text-sm">暂无监控数据</p>
    </div>
    <div v-else
         class="flex flex-col items-center gap-4 p-8 rounded-2xl
           bg-red-50/50 dark:bg-red-900/20
           border-2 border-red-100 dark:border-red-800/50
           backdrop-blur-sm animate-fade"
    >
      <div class="relative">
        <Icon
          icon="carbon:warning-filled"
          class="w-12 h-12 text-red-500/90 dark:text-red-400/90"
          aria-hidden="true"
        />
        <div class="absolute inset-0 w-12 h-12 bg-red-500/20 dark:bg-red-400/20 rounded-full motion-safe:animate-ping" />
      </div>
      <div class="text-center">
        <div class="text-red-600 dark:text-red-400 font-medium mb-1">
          {{ error }}
        </div>
      </div>
    </div>
  </div>

  <!-- 监控卡片网格布局 -->
  <div v-else>
    <!-- 整体状态摘要 -->
    <div
      class="flex items-center gap-2 px-4 py-3 mb-6 rounded-xl text-sm font-medium
        bg-emerald-50/80 dark:bg-emerald-900/20
        border border-emerald-200/70 dark:border-emerald-800/50
        text-emerald-700 dark:text-emerald-300"
      :class="summaryType === 'error' ? 'bg-red-50/80 dark:bg-red-900/20 border-red-200/70 dark:border-red-800/50 text-red-700 dark:text-red-300' : summaryType === 'warning' ? 'bg-yellow-50/80 dark:bg-yellow-900/20 border-yellow-200/70 dark:border-yellow-800/50 text-yellow-700 dark:text-yellow-300' : ''"
    >
      <Icon
        :icon="summaryType === 'ok' ? 'carbon:checkmark-filled' : summaryType === 'error' ? 'carbon:warning-filled' : 'carbon:information-filled'"
        class="w-5 h-5 shrink-0"
        aria-hidden="true"
      />
      <span>{{ summaryText }}</span>
    </div>

    <!-- 搜索 / 筛选 / 排序工具栏 -->
    <div class="flex flex-col sm:flex-row gap-3 mb-6">
      <div class="relative flex-1 min-w-[200px]">
        <Icon
          icon="carbon:search"
          class="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-gray-400 dark:text-gray-500 pointer-events-none"
          aria-hidden="true"
        />
        <input
          v-model.trim="searchQuery"
          type="search"
          placeholder="搜索站点名称或 URL"
          aria-label="搜索站点"
          class="w-full h-10 pl-9 pr-3 rounded-lg text-sm
            bg-white dark:bg-gray-800/60
            border border-gray-200 dark:border-gray-700
            text-gray-700 dark:text-gray-200
            placeholder-gray-400 dark:placeholder-gray-500
            focus:outline-none focus:ring-2 focus:ring-emerald-500/40
            transition-shadow"
        />
      </div>
      <div class="flex gap-3">
        <div class="relative flex-1 sm:flex-none sm:w-40">
          <select
            v-model="statusFilter"
            aria-label="按状态筛选"
            class="h-10 w-full cursor-pointer appearance-none rounded-lg border border-gray-200 bg-white pl-3 pr-9 text-sm text-gray-700 transition-shadow focus:outline-none focus:ring-2 focus:ring-emerald-500/40 dark:border-gray-700 dark:bg-gray-800/60 dark:text-gray-200"
          >
            <option value="all">全部状态</option>
            <option value="online">在线</option>
            <option value="abnormal">异常</option>
            <option value="pending">暂停 / 准备中</option>
          </select>
          <Icon
            icon="carbon:chevron-down"
            class="pointer-events-none absolute right-3 top-1/2 h-4 w-4 -translate-y-1/2 text-gray-400 dark:text-gray-500"
            aria-hidden="true"
          />
        </div>
        <div class="relative flex-1 sm:flex-none sm:w-40">
          <select
            v-model="sortBy"
            aria-label="排序方式"
            class="h-10 w-full cursor-pointer appearance-none rounded-lg border border-gray-200 bg-white pl-3 pr-9 text-sm text-gray-700 transition-shadow focus:outline-none focus:ring-2 focus:ring-emerald-500/40 dark:border-gray-700 dark:bg-gray-800/60 dark:text-gray-200"
          >
            <option value="default">默认排序</option>
            <option value="name">按名称</option>
            <option value="response">按响应时间</option>
            <option value="uptime">按可用率</option>
          </select>
          <Icon
            icon="carbon:chevron-down"
            class="pointer-events-none absolute right-3 top-1/2 h-4 w-4 -translate-y-1/2 text-gray-400 dark:text-gray-500"
            aria-hidden="true"
          />
        </div>
      </div>
    </div>

    <!-- 无匹配结果 -->
    <div v-if="!visibleMonitors.length"
         class="flex flex-col items-center justify-center gap-3 p-12 text-gray-400 dark:text-gray-500">
      <Icon icon="carbon:search" class="w-10 h-10" aria-hidden="true" />
      <p class="text-sm">没有符合条件的站点</p>
    </div>

    <!-- 卡片网格 -->
    <div v-else class="grid gap-6 grid-cols-1 md:grid-cols-2 xl:grid-cols-3">
      <div v-for="(monitor, index) in visibleMonitors"
           :key="monitor.id"
           class="card-base animated-border p-6 rounded-2xl backdrop-blur-sm animate-fade"
           :class="getCardBorderClass(monitor.status)"
           :style="{ animationDelay: `${Math.min(index, 12) * 60}ms` }"
           @mouseenter="$event.currentTarget.classList.add('hovered')"
      >
        <!-- 卡片头部：标题和状态指示器 -->
        <div class="flex items-start sm:items-center justify-between gap-3 sm:gap-4 mb-6">
          <div class="min-w-0">
            <div class="flex items-center gap-2">
              <h2 class="text-lg sm:text-xl font-bold truncate text-gray-800 dark:text-gray-100">
                {{ monitor.friendly_name }}
              </h2>
              <a
                v-if="monitor.url"
                :href="normalizeUrl(monitor.url)"
                target="_blank"
                rel="noopener noreferrer"
                :aria-label="`打开 ${monitor.friendly_name}`"
                :title="`打开 ${monitor.friendly_name}`"
                class="flex items-center justify-center w-7 h-7 rounded-full transition-colors duration-200
                  text-gray-400 hover:text-gray-600 hover:bg-gray-100
                  dark:text-gray-500 dark:hover:text-gray-400 dark:hover:bg-gray-700/50
                  shrink-0"
              >
                <Icon icon="bi:link-45deg" class="w-4 h-4" aria-hidden="true" />
              </a>
            </div>
          </div>
          <div class="shrink-0">
            <div
              :class="[
                'inline-flex items-center gap-1.5 px-3 sm:px-4 py-1.5 sm:py-2 rounded-full font-medium text-sm whitespace-nowrap',
                getStatusConfig(monitor.status).classes
              ]"
            >
              <div class="relative flex">
                <div :class="[
                  'w-3 h-3 rounded-full',
                  getStatusClasses(monitor.status).dot
                ]"></div>
                <div :class="[
                  'absolute inset-0 w-3 h-3 rounded-full motion-safe:animate-ping opacity-75',
                  getStatusClasses(monitor.status).dotPing
                ]"></div>
              </div>
              <span>{{ getStatusConfig(monitor.status).text }}</span>
            </div>
          </div>
        </div>

        <!-- 卡片主体：统计数据和图表 -->
        <div class="space-y-4">
          <!-- 响应时间和运行时间统计卡片 -->
          <div class="grid grid-cols-1 sm:grid-cols-2 gap-3 sm:gap-4">
            <div class="inner-card relative p-3 sm:p-4 min-w-0">
              <button
                type="button"
                :aria-label="`查看 ${monitor.friendly_name} 的响应时间趋势`"
                :title="`查看 ${monitor.friendly_name} 的响应时间趋势`"
                :class="[
                  'absolute top-2.5 right-2.5 w-7 h-7 flex items-center justify-center rounded-full transition-colors duration-200 cursor-pointer',
                  getStatusClasses(monitor.status).text,
                  getStatusClasses(monitor.status).hover.text,
                  getStatusClasses(monitor.status).hover.bg
                ]"
                @click="openResponseTimeModal(monitor)"
              >
                <Icon icon="ri:line-chart-line" class="w-4 h-4" aria-hidden="true" />
              </button>
              <div class="text-xs text-gray-500 dark:text-gray-400 mb-1">平均响应时间</div>
              <div class="text-xl font-bold text-gray-900 dark:text-gray-100">
                {{ formatters.responseTime(monitor.stats?.avgResponseTime) }}
              </div>
              <div class="text-xs text-gray-500 dark:text-gray-400 mt-1">
                最近24小时
              </div>
            </div>
            <div class="inner-card p-3 sm:p-4 min-w-0">
              <div class="text-xs text-gray-500 dark:text-gray-400 mb-1">平均运行时间</div>
              <div class="text-xl font-bold text-gray-900 dark:text-gray-100">
                {{ formatters.uptime(monitor.stats?.uptime) }}
              </div>
              <div class="text-xs text-gray-500 dark:text-gray-400 mt-1">
                最近{{ getValidDays(monitor) }}天
              </div>
            </div>
          </div>

          <!-- 状态时间线图表 -->
          <div class="inner-card">
            <!-- 监控类型和状态指示器 -->
            <div class="flex items-center gap-2 text-xs text-gray-500 dark:text-gray-400 mb-4">
              <div class="flex items-center gap-1">
                <div class="relative flex">
                  <div :class="[
                    'w-2 h-2 rounded-full',
                    getStatusClasses(monitor.status).dot
                  ]"></div>
                  <div :class="[
                    'absolute inset-0 w-2 h-2 rounded-full motion-safe:animate-ping opacity-75',
                    getStatusClasses(monitor.status).dotPing
                  ]"></div>
                </div>
                <span class="text-xs">{{ getMonitorType(monitor) }} / {{ formatInterval(monitor.interval) }}</span>
                <div class="w-1 h-1 rounded-full bg-gray-300 dark:bg-gray-600" />
                <span :class="[
                  'text-xs font-medium',
                  getStatusClasses(monitor.status).text
                ]">
                  {{ getStatusConfig(monitor.status).text }}
                </span>
              </div>
            </div>

            <!-- 30 天可用率方块 -->
            <div class="flex items-center gap-[2px] sm:gap-[3px] h-11 sm:h-12">
              <div
                v-for="(day, dayIndex) in getTimelineDays(monitor)"
                :key="dayIndex"
                class="relative flex-1 min-w-0 max-w-3 aspect-square max-h-full rounded-[3px] transition-transform duration-150 hover:scale-125 hover:z-10"
                :style="{ backgroundColor: getTimelineDayColor(day) }"
                @mouseenter="showTimelineTooltip($event, day, monitor)"
                @mouseleave="hideTimelineTooltip"
              ></div>
            </div>
            <div class="flex justify-between text-xs text-gray-400 mt-2">
              <span>30天前</span>
              <span class="text-gray-500">
                {{ getDowntimeStats(monitor) }}
              </span>
              <span>今日</span>
            </div>
          </div>

          <!-- 故障记录下拉列表 -->
          <div class="relative">
            <button
              type="button"
              @click="toggleDowntimeList(monitor.id)"
              :data-monitor-id="monitor.id.toString()"
              :aria-expanded="showDowntimeList === monitor.id"
              :aria-controls="`downtime-panel-${monitor.id}`"
              class="w-full px-4 py-3 flex items-center justify-between text-left
                bg-gray-50 dark:bg-gray-800/50
                rounded-lg transition-colors duration-200
                hover:bg-gray-100 dark:hover:bg-gray-700/50
                focus:outline-none focus-visible:ring-2 focus-visible:ring-emerald-500/40"
            >
              <span class="text-xs text-gray-500 dark:text-gray-400">故障记录</span>
              <Icon
                icon="bi:chevron-up"
                class="w-4 h-4 text-gray-400 transition-transform duration-200"
                :class="{ 'rotate-180': showDowntimeList === monitor.id }"
                aria-hidden="true"
              />
            </button>

            <Transition
              enter-active-class="transition-all duration-200 ease-out"
              enter-from-class="opacity-0 translate-y-[10px] scale-95"
              enter-to-class="opacity-100 translate-y-0 scale-100"
              leave-active-class="transition-all duration-200 ease-in"
              leave-from-class="opacity-100 translate-y-0 scale-100"
              leave-to-class="opacity-0 translate-y-[10px] scale-95"
            >
              <div v-if="showDowntimeList === monitor.id"
                   :id="`downtime-panel-${monitor.id}`"
                   class="absolute bottom-full left-0 right-0 mb-2
                     bg-white dark:bg-gray-800 border-[1.5px] border-gray-200 dark:border-gray-700
                     rounded-lg downtime-list shadow-lg"
              >
                <div class="p-4 max-h-[280px] overflow-y-auto">
                  <TransitionGroup
                    tag="div"
                    class="space-y-2"
                    enter-active-class="transition duration-200 ease-out"
                    enter-from-class="opacity-0 scale-95"
                    enter-to-class="opacity-100 scale-100"
                    leave-active-class="transition duration-200 ease-in"
                    leave-from-class="opacity-100 scale-100"
                    leave-to-class="opacity-0 scale-95"
                    move-class="transition duration-200"
                  >
                    <div v-if="getDowntimeLogs(monitor)?.length"
                         v-for="log in getDowntimeLogs(monitor)"
                         :key="log.id"
                         class="p-3 bg-red-50/90 dark:bg-red-900/20 rounded-lg
                           border border-red-200/80 dark:border-red-800/80"
                    >
                      <div class="flex justify-between gap-2">
                        <span class="text-red-600/90 dark:text-red-400/90 text-xs">{{ getErrorMessage(log.reason) }}</span>
                        <span class="text-red-600/80 dark:text-red-400/80 text-xs shrink-0">{{ formatters.dateTime(log.datetime) }}</span>
                      </div>
                      <div class="mt-1 text-red-600/80 dark:text-red-400/80 text-xs">
                        持续时间: {{ formatters.duration(log.duration) }}
                      </div>
                    </div>
                    <div v-else
                         key="empty"
                         class="text-center text-xs text-gray-400"
                    >
                      近期无故障记录
                    </div>
                  </TransitionGroup>
                  <button
                    v-if="(monitor.stats?.downtimeLogs || []).length > 15"
                    type="button"
                    class="mt-3 w-full text-xs font-medium text-gray-500 dark:text-gray-400
                      hover:text-emerald-600 dark:hover:text-emerald-400 transition-colors"
                    @click="toggleDowntimeExpanded(monitor.id)"
                  >
                    {{ downtimeExpanded[monitor.id] ? '收起' : `查看全部 ${(monitor.stats?.downtimeLogs || []).length} 条` }}
                  </button>
                </div>
              </div>
            </Transition>
          </div>
        </div>
      </div>
    </div>
  </div>

  <!-- 响应时间详情模态框 -->
  <Teleport to="body">
    <div v-if="modalMounted"
         class="fixed inset-0 z-50 flex items-center justify-center p-4"
         @keydown.esc="closeModal"
    >
      <!-- 背景遮罩 -->
      <Transition
        appear
        enter-from-class="opacity-0"
        enter-to-class="opacity-100"
        enter-active-class="transition-opacity duration-300"
        leave-from-class="opacity-100"
        leave-to-class="opacity-0"
        leave-active-class="transition-opacity duration-300"
      >
        <div v-show="showResponseTimeModal"
             class="absolute inset-0 bg-black/60"
             @click="closeModal"
        ></div>
      </Transition>

      <!-- 模态框内容 -->
      <Transition
        appear
        enter-from-class="opacity-0 translate-y-4 scale-95"
        enter-to-class="opacity-100 translate-y-0 scale-100"
        enter-active-class="transition-all duration-300 transform"
        leave-from-class="opacity-100 translate-y-0 scale-100"
        leave-to-class="opacity-0 translate-y-4 scale-95"
        leave-active-class="transition-all duration-300 transform"
        @after-leave="onAfterLeave"
      >
        <div v-show="showResponseTimeModal"
             id="response-modal"
             tabindex="-1"
             role="dialog"
             aria-modal="true"
             :aria-label="selectedMonitor ? `${selectedMonitor.friendly_name} 响应时间趋势` : '响应时间趋势'"
             class="relative bg-white dark:bg-gray-800 rounded-2xl p-6 w-full max-w-3xl
                    shadow-xl border border-gray-200 dark:border-gray-700
                    max-h-[90vh] overflow-y-auto outline-none"
             @click.stop
        >
          <div class="flex justify-between items-center mb-6">
            <h3 class="text-lg font-bold text-gray-900 dark:text-gray-100">
              {{ selectedMonitor?.friendly_name }} - 响应时间趋势
            </h3>
            <button type="button" @click="closeModal"
                    aria-label="关闭"
                    class="p-1 hover:bg-gray-100 dark:hover:bg-gray-700 rounded-full
                           transition-colors duration-200">
              <Icon icon="carbon:close"
                    class="w-5 h-5 text-gray-500 dark:text-gray-400"
                    aria-hidden="true" />
            </button>
          </div>

          <div class="h-[300px]">
            <!-- 无数据状态 -->
            <div v-if="!hasResponseTimeData(selectedMonitor)"
                 class="h-full flex flex-col items-center justify-center gap-4">
              <Icon
                icon="carbon:chart-line"
                class="w-12 h-12 text-gray-400 dark:text-gray-500"
                aria-hidden="true"
              />
              <div class="text-gray-500 dark:text-gray-400 text-sm">
                暂无数据
              </div>
            </div>
            <!-- 数据图表 -->
            <Line v-else
                  :data="getResponseTimeChartData(selectedMonitor)"
                  :options="responseTimeChartOptions"
            />
          </div>
        </div>
      </Transition>
    </div>
  </Teleport>

  <!-- 全局可用率浮窗：通过 Teleport 挂到 body，避免被卡片遮挡 -->
  <Teleport to="body">
    <div
      v-if="timelineTooltip.visible"
      class="fixed z-[9999] pointer-events-none -translate-x-1/2 -translate-y-full rounded-md bg-gray-900/95 px-2 py-1 text-xs whitespace-nowrap text-white shadow-lg dark:bg-gray-950/95"
      :style="{ left: timelineTooltip.x + 'px', top: timelineTooltip.y + 'px' }"
    >
      {{ timelineTooltip.text }}
    </div>
  </Teleport>

</template>


<script setup>
import { ref, computed, nextTick, onMounted, onUnmounted } from 'vue'
import { format } from 'date-fns'
import { Icon } from '@iconify/vue'
import { Line } from 'vue-chartjs'
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend
} from 'chart.js'
import {
  getChartColor,
  getResponseTimeChartData,
  responseTimeChartOptions
} from '@/utils/chartConfig'

// 注册 Chart.js 组件
ChartJS.register(
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend
)

const props = defineProps({
  monitors: Array,
  error: String,
  loading: {
    type: Boolean,
    default: false
  }
})

/**
 * 整体状态摘要
 */
const summary = computed(() => {
  const list = props.monitors || []
  const abnormal = list.filter(m => m.status === 9 || m.status === undefined).length
  const pending = list.filter(m => m.status === 0 || m.status === 1).length

  if (abnormal > 0) {
    return {
      type: 'error',
      text: `当前有 ${abnormal} 个站点异常，请留意`
    }
  }
  if (pending > 0) {
    return {
      type: 'warning',
      text: `有 ${pending} 个站点处于暂停或准备中`
    }
  }
  return {
    type: 'ok',
    text: list.length ? '全部站点运行正常' : ''
  }
})

const summaryText = computed(() => summary.value.text)
const summaryType = computed(() => summary.value.type)

/**
 * 搜索 / 筛选 / 排序
 */
const searchQuery = ref('')
const statusFilter = ref('all')
const sortBy = ref('default')

/**
 * 状态常量定义
 */
const STATUS = {
  ONLINE: 2,    // 在线状态
  PAUSED: 0,    // 暂停状态
  PREPARING: 1, // 准备中状态
  OFFLINE: 9    // 离线状态
}

/**
 * 状态配置映射
 */
const STATUS_CONFIG = {
  2: {
    text: '在线', color: 'green',
    classes: 'bg-green-50 dark:bg-green-900/30 text-green-600 dark:text-green-400'
  },
  0: {
    text: '暂停', color: 'yellow',
    classes: 'bg-yellow-50 dark:bg-yellow-900/30 text-yellow-600 dark:text-yellow-400'
  },
  1: {
    text: '准备中', color: 'yellow',
    classes: 'bg-yellow-50 dark:bg-yellow-900/30 text-yellow-600 dark:text-yellow-400'
  },
  9: {
    text: '离线', color: 'red',
    classes: 'bg-red-50 dark:bg-red-900/30 text-red-600 dark:text-red-400'
  }
}

const FALLBACK_STATUS = {
  text: '未知', color: 'gray',
  classes: 'bg-gray-100 dark:bg-gray-800 text-gray-500 dark:text-gray-400'
}

const getStatusConfig = (status) => STATUS_CONFIG[status] || FALLBACK_STATUS

/**
 * 卡片边框颜色静态映射（避免动态拼接 Tailwind 类）
 */
const CARD_BORDER_CLASSES = {
  2: 'after:border-green-500/50 dark:after:border-green-400/50',
  0: 'after:border-yellow-500/50 dark:after:border-yellow-400/50',
  1: 'after:border-yellow-500/50 dark:after:border-yellow-400/50',
  9: 'after:border-red-500/50 dark:after:border-red-400/50'
}

const getCardBorderClass = (status) => CARD_BORDER_CLASSES[status] || 'after:border-gray-500/50 dark:after:border-gray-400/50'

/**
 * 排序后的监控列表
 */
const visibleMonitors = computed(() => {
  if (!props.monitors?.length) return []

  const query = searchQuery.value.toLowerCase()

  const filtered = props.monitors.filter(monitor => {
    // 搜索
    if (query) {
      const name = (monitor.friendly_name || '').toLowerCase()
      const url = (monitor.url || '').toLowerCase()
      if (!name.includes(query) && !url.includes(query)) return false
    }

    // 状态筛选
    if (statusFilter.value === 'online' && monitor.status !== STATUS.ONLINE) return false
    if (statusFilter.value === 'abnormal' && monitor.status !== STATUS.OFFLINE) return false
    if (statusFilter.value === 'pending' && monitor.status !== STATUS.PAUSED && monitor.status !== STATUS.PREPARING) return false

    return true
  })

  const sorted = [...filtered]

  if (sortBy.value === 'name') {
    sorted.sort((a, b) => (a.friendly_name || '').localeCompare(b.friendly_name || '', 'zh-CN'))
  } else if (sortBy.value === 'response') {
    sorted.sort((a, b) => {
      const av = a.stats?.avgResponseTime
      const bv = b.stats?.avgResponseTime
      if (av == null) return 1
      if (bv == null) return -1
      return av - bv
    })
  } else if (sortBy.value === 'uptime') {
    sorted.sort((a, b) => {
      const av = a.stats?.uptime
      const bv = b.stats?.uptime
      if (av == null) return 1
      if (bv == null) return -1
      return bv - av
    })
  } else {
    // 默认排序：离线排最后，其余保持原有顺序
    sorted.sort((a, b) => {
      if (a.status === b.status) return 0
      if (a.status === STATUS.OFFLINE) return 1
      if (b.status === STATUS.OFFLINE) return -1
      return 0
    })
  }

  return sorted
})


/**
 * 格式化工具函数
 */
const formatters = {
  /** 格式化响应时间 */
  responseTime: time => {
    if (time == null || !Number.isFinite(Number(time))) return '—'
    return `${Math.round(Number(time))} ms`
  },
  /** 格式化运行时间 */
  uptime: uptime => {
    if (uptime == null || !Number.isFinite(Number(uptime))) return '—'
    return `${Number(uptime).toFixed(2)}%`
  },
  /** 格式化持续时间 */
  duration: seconds => {
    if (!seconds) return '0秒'
    const h = Math.floor(seconds / 3600)
    const m = Math.floor((seconds % 3600) / 60)
    const s = seconds % 60

    // 如果超过100小时，只显示小时
    if (h >= 100) {
      return `约${h}小时`
    }

    return [
      h && `${h}小时`,
      m && `${m}分钟`,
      (!h && !m && s) && `${s}秒`
    ].filter(Boolean).join('')
  },
  /** 格式化日期时间 */
  dateTime: ts => format(new Date(ts * 1000), 'MM-dd HH:mm')
}

/**
 * 获取状态对应的样式类
 */
const getStatusClasses = (status) => {
  const isOnline = status === STATUS.ONLINE
  const isPending = status === STATUS.PAUSED || status === STATUS.PREPARING
  const isOffline = status === STATUS.OFFLINE
  return {
    dot: {
      'bg-green-500 dark:bg-green-400': isOnline,
      'bg-yellow-500 dark:bg-yellow-400': isPending,
      'bg-red-500 dark:bg-red-400': isOffline,
      'bg-gray-400 dark:bg-gray-500': !isOnline && !isPending && !isOffline
    },
    dotPing: {
      'bg-green-500 dark:bg-green-400': isOnline,
      'bg-yellow-500 dark:bg-yellow-400': isPending,
      'bg-red-500 dark:bg-red-400': isOffline,
      'bg-gray-400 dark:bg-gray-500': !isOnline && !isPending && !isOffline
    },
    text: {
      'text-green-500': isOnline,
      'text-yellow-500': isPending,
      'text-red-500': isOffline,
      'text-gray-400 dark:text-gray-500': !isOnline && !isPending && !isOffline
    },
    hover: {
      text: {
        'hover:text-green-600 dark:hover:text-green-300': isOnline,
        'hover:text-yellow-600 dark:hover:text-yellow-300': isPending,
        'hover:text-red-600 dark:hover:text-red-300': isOffline,
        'hover:text-gray-500 dark:hover:text-gray-400': !isOnline && !isPending && !isOffline
      },
      bg: {
        'hover:bg-green-50 dark:hover:bg-green-900/30': isOnline,
        'hover:bg-yellow-50 dark:hover:bg-yellow-900/30': isPending,
        'hover:bg-red-50 dark:hover:bg-red-900/30': isOffline,
        'hover:bg-gray-100 dark:hover:bg-gray-700/50': !isOnline && !isPending && !isOffline
      }
    }
  }
}

/**
 * 监控类型映射
 */
const monitorTypeMap = {
  1: 'HTTPS',
  2: 'Keyword',
  3: 'PING',
  4: 'Port',
  default: 'HTTP'
}

/**
 * 获取监控类型
 */
const getMonitorType = (monitor) => {
  return monitorTypeMap[monitor.type] || monitorTypeMap.default
}

const formatInterval = (interval) => {
  if (!Number.isFinite(Number(interval))) return '—'
  const minutes = Math.floor(Number(interval) / 60)
  return minutes > 0 ? `${minutes}m` : `${Number(interval)}s`
}

/**
 * 错误消息映射
 */
const ERROR_MESSAGES = {
  333333: '连接超时',
  444444: '无响应',
  100001: 'DNS解析失败',
  98: '离线状态',
  99: '失联状态',
  default: '连接异常'
}

/**
 * 获取错误消息
 */
const getErrorMessage = (code) => {
  const errorCode = typeof code === 'object' ? code.code : code
  return ERROR_MESSAGES[errorCode] || ERROR_MESSAGES.default
}

/**
 * 获取宕机统计信息
 */
const getDowntimeStats = (monitor) => {
  const downtimeLogs = monitor.stats?.downtimeLogs || []
  const downtimeCount = downtimeLogs.length
  const totalDowntime = formatters.duration(monitor.stats?.totalDowntime)
  const validDays = getValidDays(monitor)

  if (validDays <= 0) return '暂无数据'

  if (downtimeCount > 0 || monitor.status === STATUS.OFFLINE) {
    if (downtimeCount > 0) {
      return `最近${validDays}天 ${downtimeCount} 次故障，总计${totalDowntime}`
    }
    return '当前离线'
  }
  return `最近${validDays}天运行正常`
}


/**
 * 响应式状态
 */
const showDowntimeList = ref(null)
const showResponseTimeModal = ref(false)
const selectedMonitor = ref(null)
const downtimeExpanded = ref({})
const timelineTooltip = ref({ visible: false, text: '', x: 0, y: 0 })

/**
 * URL 处理函数
 */
const normalizeUrl = (url) => {
  if (!url) return '#'
  return !url.startsWith('http://') && !url.startsWith('https://')
    ? 'http://' + url
    : url
}

/**
 * 图表相关配置
 */
const dateRange = computed(() => {
  const now = new Date()
  now.setHours(0, 0, 0, 0)
  const dates = Array.from({ length: 30 }, (_, i) => {
    const date = new Date(now)
    date.setDate(date.getDate() - (29 - i))
    return date
  })
  return { startDate: dates[0], dates }
})

/**
 * 宕机日志获取
 */
const getDowntimeLogs = (monitor) => {
  const logs = monitor.stats?.downtimeLogs || []
  return downtimeExpanded.value[monitor.id] ? logs : logs.slice(0, 15)
}

const toggleDowntimeExpanded = (id) => {
  downtimeExpanded.value[id] = !downtimeExpanded.value[id]
}

/**
 * 计算有效监控天数
 */
const getValidDays = (monitor) => {
  if (!monitor.stats?.dailyUptimes) return 0

  // 添加时间验证逻辑
  const now = Date.now()
  const createTime = monitor.create_datetime ? monitor.create_datetime * 1000 : now
  const effectiveCreateTime = createTime > now ? now : createTime

  const daysSinceStart = Math.max(0, Math.floor(
    (new Date(effectiveCreateTime) - dateRange.value.startDate) / 86400000
  ))

  return monitor.stats.dailyUptimes
    .slice(daysSinceStart)
    .filter(v => v != null && !isNaN(v))
    .length
}

/**
 * 30 天可用率方块数据（固定 30 个，方便对齐和间距控制）
 */
const getTimelineDays = (monitor) => {
  const raw = monitor.stats?.dailyUptimes || []
  const data = Array.from({ length: 30 }, (_, i) => raw[i] ?? null)

  const now = Date.now()
  const createTime = monitor.create_datetime ? monitor.create_datetime * 1000 : now
  const effectiveCreateTime = createTime > now ? now : createTime
  const daysSinceStart = Math.max(0, Math.floor(
    (new Date(effectiveCreateTime) - dateRange.value.startDate) / 86400000
  ))

  return data.map((value, i) => ({
    value,
    date: dateRange.value.dates[i],
    status: monitor.status,
    isBeforeCreation: i < daysSinceStart
  }))
}

const getTimelineDayColor = (day) => getChartColor(day.value, day.isBeforeCreation, day.status)

const getTimelineDayTitle = (day, monitor) => {
  const dateText = format(day.date, 'yyyy-MM-dd')
  if (day.isBeforeCreation) return `${dateText}：无数据`
  if (day.status === 0) return `${dateText}：已暂停`
  if (day.value == null || isNaN(day.value)) return `${dateText}：无数据`
  return `${dateText}：可用率 ${Number(day.value).toFixed(2)}%`
}

const clamp = (value, min, max) => Math.min(Math.max(value, min), max)

const showTimelineTooltip = (event, day, monitor) => {
  const rect = event.currentTarget.getBoundingClientRect()
  timelineTooltip.value = {
    visible: true,
    text: getTimelineDayTitle(day, monitor),
    x: clamp(rect.left + rect.width / 2, 90, window.innerWidth - 90),
    y: rect.top - 8
  }
}

const hideTimelineTooltip = () => {
  timelineTooltip.value.visible = false
}

const hasResponseTimeData = (monitor) => {
  const value = monitor?.stats?.avgResponseTime
  return value != null && Number.isFinite(Number(value))
}

/**
 * 事件监听
 */
const closeOnClickOutside = (e) => {
  if (showDowntimeList.value) {
    const path = e.composedPath()
    const isClickInside = path.some(element => {
      return element.classList?.contains('downtime-list') ||
              element.dataset?.monitorId === showDowntimeList.value.toString()
    })
    if (!isClickInside) {
      showDowntimeList.value = null
    }
  }
}

const toggleDowntimeList = (id) => {
  showDowntimeList.value = showDowntimeList.value === id ? null : id
}

/**
 * 控制模态框挂载状态
 */
const modalMounted = ref(false)
let lastFocusedElement = null

// 打开模态框
const openResponseTimeModal = (monitor) => {
  selectedMonitor.value = monitor
  lastFocusedElement = document.activeElement
  modalMounted.value = true
  showResponseTimeModal.value = true
  document.body.style.overflow = 'hidden'
  nextTick(() => {
    document.getElementById('response-modal')?.focus()
  })
}

// 关闭模态框
const closeModal = () => {
  showResponseTimeModal.value = false
  document.body.style.overflow = ''
  lastFocusedElement?.focus?.()
}

// 在动画结束后卸载组件
const onAfterLeave = () => {
  modalMounted.value = false
}

const handleKeydown = (e) => {
  if (e.key === 'Escape' && showResponseTimeModal.value) {
    closeModal()
  }
}

/**
 * 生命周期钩子
 */
onMounted(() => {
  document.addEventListener('click', closeOnClickOutside)
  window.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  document.removeEventListener('click', closeOnClickOutside)
  window.removeEventListener('keydown', handleKeydown)
  document.body.style.overflow = ''
  timelineTooltip.value.visible = false
})
</script>
