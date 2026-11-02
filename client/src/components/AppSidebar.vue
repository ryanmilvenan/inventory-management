<template>
  <aside
    :class="[
      'fixed left-0 top-0 h-screen bg-slate-800 text-white transition-all duration-300 z-50 flex flex-col',
      collapsed ? 'w-16' : 'w-64'
    ]"
  >
    <!-- Logo -->
    <div class="h-16 flex items-center px-4 border-b border-white/10">
      <div class="h-8 w-8 bg-blue-500 rounded-lg flex items-center justify-center text-white font-bold text-sm flex-shrink-0">
        CC
      </div>
      <div v-if="!collapsed" class="ml-3 min-w-0">
        <div class="font-semibold text-sm truncate">{{ t('nav.companyName') }}</div>
        <div class="text-xs text-slate-400 truncate">{{ t('nav.subtitle') }}</div>
      </div>
    </div>

    <!-- Navigation -->
    <nav class="flex-1 overflow-y-auto py-4">
      <!-- Main Navigation -->
      <div class="mb-6">
        <div v-if="!collapsed" class="px-4 mb-2 text-xs font-semibold text-slate-500 uppercase tracking-wider">
          Main
        </div>
        <router-link
          v-for="item in mainNavItems"
          :key="item.path"
          :to="item.path"
          :class="[
            'flex items-center px-4 py-2.5 text-sm transition-colors mx-2 rounded-lg',
            'hover:bg-slate-700',
            $route.path === item.path ? 'bg-blue-600 text-white' : 'text-slate-300'
          ]"
        >
          <component :is="item.icon" class="h-5 w-5 flex-shrink-0" />
          <span v-if="!collapsed" class="ml-3">{{ item.label }}</span>
        </router-link>
      </div>

      <!-- Operations -->
      <div class="mb-6">
        <div v-if="!collapsed" class="px-4 mb-2 text-xs font-semibold text-slate-500 uppercase tracking-wider">
          Operations
        </div>
        <router-link
          v-for="item in operationsNavItems"
          :key="item.path"
          :to="item.path"
          :class="[
            'flex items-center px-4 py-2.5 text-sm transition-colors mx-2 rounded-lg',
            'hover:bg-slate-700',
            $route.path === item.path ? 'bg-blue-600 text-white' : 'text-slate-300'
          ]"
        >
          <component :is="item.icon" class="h-5 w-5 flex-shrink-0" />
          <span v-if="!collapsed" class="ml-3">{{ item.label }}</span>
        </router-link>
      </div>

      <!-- Analytics -->
      <div>
        <div v-if="!collapsed" class="px-4 mb-2 text-xs font-semibold text-slate-500 uppercase tracking-wider">
          Analytics
        </div>
        <router-link
          v-for="item in analyticsNavItems"
          :key="item.path"
          :to="item.path"
          :class="[
            'flex items-center px-4 py-2.5 text-sm transition-colors mx-2 rounded-lg',
            'hover:bg-slate-700',
            $route.path === item.path ? 'bg-blue-600 text-white' : 'text-slate-300'
          ]"
        >
          <component :is="item.icon" class="h-5 w-5 flex-shrink-0" />
          <span v-if="!collapsed" class="ml-3">{{ item.label }}</span>
        </router-link>
      </div>
    </nav>

    <!-- Collapse Toggle -->
    <button
      @click="collapsed = !collapsed"
      class="absolute -right-3 top-20 bg-slate-800 border-2 border-slate-600 rounded-full p-1 hover:bg-slate-700 transition-colors"
    >
      <svg :class="['h-4 w-4 text-slate-300 transition-transform', collapsed ? '' : 'rotate-180']" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7" />
      </svg>
    </button>

    <!-- User Profile -->
    <div class="border-t border-white/10 p-3">
      <button
        @click="$emit('show-profile')"
        :class="['flex items-center w-full rounded-lg p-2 hover:bg-slate-700 transition-colors', collapsed ? 'justify-center' : '']"
      >
        <div class="h-9 w-9 rounded-full bg-blue-500 flex items-center justify-center text-sm font-medium flex-shrink-0">
          {{ userInitials }}
        </div>
        <div v-if="!collapsed" class="ml-3 flex-1 min-w-0 text-left">
          <div class="text-sm font-medium truncate">{{ userName }}</div>
          <div class="text-xs text-slate-400 truncate">{{ userRole }}</div>
        </div>
        <svg v-if="!collapsed" class="h-4 w-4 text-slate-400 flex-shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7" />
        </svg>
      </button>
    </div>
  </aside>
</template>

<script setup>
import { ref, computed, h } from 'vue'
import { useI18n } from '../composables/useI18n'

const props = defineProps({
  userName: { type: String, default: 'User' },
  userRole: { type: String, default: '' }
})

defineEmits(['show-profile'])

const { t } = useI18n()
const collapsed = ref(false)

const userInitials = computed(() =>
  props.userName.split(' ').map(n => n[0]).join('').toUpperCase().slice(0, 2)
)

// SVG Icon components
const HomeIcon = {
  render() {
    return h('svg', { fill: 'none', stroke: 'currentColor', viewBox: '0 0 24 24', class: 'h-5 w-5' }, [
      h('path', { 'stroke-linecap': 'round', 'stroke-linejoin': 'round', 'stroke-width': '2', d: 'M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6' })
    ])
  }
}

const BoxIcon = {
  render() {
    return h('svg', { fill: 'none', stroke: 'currentColor', viewBox: '0 0 24 24', class: 'h-5 w-5' }, [
      h('path', { 'stroke-linecap': 'round', 'stroke-linejoin': 'round', 'stroke-width': '2', d: 'M20 7l-8-4-8 4m16 0l-8 4m8-4v10l-8 4m0-10L4 7m8 4v10M4 7v10l8 4' })
    ])
  }
}

const ClipboardIcon = {
  render() {
    return h('svg', { fill: 'none', stroke: 'currentColor', viewBox: '0 0 24 24', class: 'h-5 w-5' }, [
      h('path', { 'stroke-linecap': 'round', 'stroke-linejoin': 'round', 'stroke-width': '2', d: 'M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2' })
    ])
  }
}

const RefreshIcon = {
  render() {
    return h('svg', { fill: 'none', stroke: 'currentColor', viewBox: '0 0 24 24', class: 'h-5 w-5' }, [
      h('path', { 'stroke-linecap': 'round', 'stroke-linejoin': 'round', 'stroke-width': '2', d: 'M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15' })
    ])
  }
}

const ChartIcon = {
  render() {
    return h('svg', { fill: 'none', stroke: 'currentColor', viewBox: '0 0 24 24', class: 'h-5 w-5' }, [
      h('path', { 'stroke-linecap': 'round', 'stroke-linejoin': 'round', 'stroke-width': '2', d: 'M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z' })
    ])
  }
}

const TrendingIcon = {
  render() {
    return h('svg', { fill: 'none', stroke: 'currentColor', viewBox: '0 0 24 24', class: 'h-5 w-5' }, [
      h('path', { 'stroke-linecap': 'round', 'stroke-linejoin': 'round', 'stroke-width': '2', d: 'M13 7h8m0 0v8m0-8l-8 8-4-4-6 6' })
    ])
  }
}

const CurrencyIcon = {
  render() {
    return h('svg', { fill: 'none', stroke: 'currentColor', viewBox: '0 0 24 24', class: 'h-5 w-5' }, [
      h('path', { 'stroke-linecap': 'round', 'stroke-linejoin': 'round', 'stroke-width': '2', d: 'M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2m0-8c1.11 0 2.08.402 2.599 1M12 8V7m0 1v8m0 0v1m0-1c-1.11 0-2.08-.402-2.599-1M21 12a9 9 0 11-18 0 9 9 0 0118 0z' })
    ])
  }
}

const DocumentIcon = {
  render() {
    return h('svg', { fill: 'none', stroke: 'currentColor', viewBox: '0 0 24 24', class: 'h-5 w-5' }, [
      h('path', { 'stroke-linecap': 'round', 'stroke-linejoin': 'round', 'stroke-width': '2', d: 'M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z' })
    ])
  }
}

const mainNavItems = computed(() => [
  { path: '/', label: t('nav.overview'), icon: HomeIcon },
  { path: '/inventory', label: t('nav.inventory'), icon: BoxIcon },
  { path: '/orders', label: t('nav.orders'), icon: ClipboardIcon },
])

const operationsNavItems = computed(() => [
  { path: '/restocking', label: t('nav.restocking'), icon: RefreshIcon },
  { path: '/spending', label: t('nav.finance'), icon: CurrencyIcon },
])

const analyticsNavItems = computed(() => [
  { path: '/demand', label: t('nav.demandForecast'), icon: TrendingIcon },
  { path: '/reports', label: 'Reports', icon: DocumentIcon },
])
</script>
