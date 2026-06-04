<template>
  <div class="app">
    <aside class="sidebar" :class="{ collapsed: sidebarCollapsed }">
      <div class="sidebar-brand">
        <div class="brand-text">
          <h1 class="brand-name">{{ t('nav.companyName') }}</h1>
          <span class="brand-subtitle">{{ t('nav.subtitle') }}</span>
        </div>
        <button class="sidebar-toggle" @click="toggleSidebar" :title="sidebarCollapsed ? 'Expand sidebar' : 'Collapse sidebar'">
          <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
            <path v-if="!sidebarCollapsed" d="M10 3L5 8L10 13" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
            <path v-else d="M6 3L11 8L6 13" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </button>
      </div>

      <nav class="sidebar-nav">
        <router-link to="/" class="nav-item" :class="{ active: $route.path === '/' }" :title="sidebarCollapsed ? t('nav.overview') : ''">
          <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
            <rect x="2" y="2" width="6" height="6" rx="1" stroke="currentColor" stroke-width="1.5"/>
            <rect x="10" y="2" width="6" height="6" rx="1" stroke="currentColor" stroke-width="1.5"/>
            <rect x="2" y="10" width="6" height="6" rx="1" stroke="currentColor" stroke-width="1.5"/>
            <rect x="10" y="10" width="6" height="6" rx="1" stroke="currentColor" stroke-width="1.5"/>
          </svg>
          <span>{{ t('nav.overview') }}</span>
        </router-link>

        <router-link to="/inventory" class="nav-item" :class="{ active: $route.path === '/inventory' }" :title="sidebarCollapsed ? t('nav.inventory') : ''">
          <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
            <path d="M2 5L9 2L16 5V13L9 16L2 13V5Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/>
            <path d="M9 2V16" stroke="currentColor" stroke-width="1.5"/>
            <path d="M2 5L9 8L16 5" stroke="currentColor" stroke-width="1.5"/>
          </svg>
          <span>{{ t('nav.inventory') }}</span>
        </router-link>

        <router-link to="/orders" class="nav-item" :class="{ active: $route.path === '/orders' }" :title="sidebarCollapsed ? t('nav.orders') : ''">
          <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
            <rect x="3" y="2" width="12" height="14" rx="1.5" stroke="currentColor" stroke-width="1.5"/>
            <path d="M6 6H12M6 9H12M6 12H10" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          </svg>
          <span>{{ t('nav.orders') }}</span>
        </router-link>

        <router-link to="/spending" class="nav-item" :class="{ active: $route.path === '/spending' }" :title="sidebarCollapsed ? t('nav.finance') : ''">
          <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
            <path d="M3 13L7 9L10 11L15 5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
            <circle cx="15" cy="5" r="1.5" fill="currentColor"/>
          </svg>
          <span>{{ t('nav.finance') }}</span>
        </router-link>

        <router-link to="/demand" class="nav-item" :class="{ active: $route.path === '/demand' }" :title="sidebarCollapsed ? t('nav.demandForecast') : ''">
          <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
            <path d="M2 14L6 10L9 12L13 7L16 9" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
            <path d="M13 5H16V8" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
          <span>{{ t('nav.demandForecast') }}</span>
        </router-link>

        <router-link to="/reports" class="nav-item" :class="{ active: $route.path === '/reports' }" :title="sidebarCollapsed ? 'Reports' : ''">
          <svg width="18" height="18" viewBox="0 0 18 18" fill="none">
            <rect x="2" y="2" width="14" height="14" rx="1.5" stroke="currentColor" stroke-width="1.5"/>
            <path d="M5 7H13M5 10H10M5 13H8" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          </svg>
          <span>Reports</span>
        </router-link>
      </nav>

      <div class="sidebar-footer">
        <div class="sidebar-footer-divider"></div>
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>
    </aside>

    <div class="main-wrapper" :class="{ collapsed: sidebarCollapsed }">
      <FilterBar />
      <main class="main-content">
        <router-view />
      </main>
    </div>

    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />
    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script>
import { ref, onMounted, onUnmounted, computed } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()

    // Sidebar collapse state — persisted to localStorage
    const sidebarCollapsed = ref(localStorage.getItem('sidebarCollapsed') === 'true')

    const toggleSidebar = () => {
      sidebarCollapsed.value = !sidebarCollapsed.value
      localStorage.setItem('sidebarCollapsed', sidebarCollapsed.value)
    }

    // Auto-collapse on small screens
    const handleResize = () => {
      if (window.innerWidth < 1024) {
        sidebarCollapsed.value = true
      }
    }
    window.addEventListener('resize', handleResize)
    // Check initial width
    if (window.innerWidth < 1024) {
      sidebarCollapsed.value = true
    }

    onUnmounted(() => {
      window.removeEventListener('resize', handleResize)
    })

    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const apiTasks = ref([])

    // Merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        // Add new task to the beginning of the array
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          // Remove from mock tasks
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          // Remove from API tasks
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          // Toggle mock task status
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          // Toggle API task
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(loadTasks)

    return {
      t,
      sidebarCollapsed,
      toggleSidebar,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  background: #f8fafc;
  color: #1e293b;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* === LAYOUT === */
.app {
  display: flex;
  flex-direction: row;
  min-height: 100vh;
}

/* === SIDEBAR === */
.sidebar {
  width: 240px;
  min-width: 240px;
  background: #1e293b;
  display: flex;
  flex-direction: column;
  position: fixed;
  top: 0;
  left: 0;
  height: 100vh;
  z-index: 100;
  overflow-y: auto;
  overflow-x: hidden;
  transition: width 0.2s ease, min-width 0.2s ease;
}

.main-wrapper {
  transition: margin-left 0.2s ease;
}

/* === COLLAPSED STATE === */
.sidebar.collapsed {
  width: 64px;
  min-width: 64px;
}

.main-wrapper.collapsed {
  margin-left: 64px;
}

/* Brand area in collapsed state */
.sidebar-toggle {
  background: transparent;
  border: none;
  color: #64748b;
  cursor: pointer;
  padding: 0.25rem;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
  transition: color 0.15s ease, background 0.15s ease;
}

.sidebar-toggle:hover {
  color: #e2e8f0;
  background: #334155;
}

.sidebar.collapsed .sidebar-toggle {
  margin: 0 auto;
}

.sidebar.collapsed .brand-text {
  display: none;
}

/* Nav items in collapsed state */
.sidebar.collapsed .nav-item {
  justify-content: center;
  padding: 0;
  width: 40px;
  margin: 0 auto;
  border-left: none;
  border-radius: 8px;
}

.sidebar.collapsed .nav-item.active {
  border-left: none;
  background: #334155;
  color: #60a5fa;
}

.sidebar.collapsed .nav-item span {
  display: none;
}

.sidebar.collapsed .nav-item svg {
  opacity: 1;
}

/* Sidebar nav padding in collapsed state */
.sidebar.collapsed .sidebar-nav {
  padding: 0.75rem 0.75rem 0;
  align-items: center;
}

/* Footer in collapsed state — hide text-heavy components, show compact icons */
.sidebar.collapsed .sidebar-footer {
  align-items: center;
}

.sidebar.collapsed .sidebar-footer :deep(.language-button),
.sidebar.collapsed .sidebar-footer :deep(.profile-button) {
  width: 40px;
  height: 40px;
  padding: 0;
  justify-content: center;
  border-radius: 8px;
}

.sidebar.collapsed .sidebar-footer :deep(.language-label),
.sidebar.collapsed .sidebar-footer :deep(.profile-name),
.sidebar.collapsed .sidebar-footer :deep(.chevron) {
  display: none;
}

/* === SIDEBAR BRAND (base) === */
.sidebar-brand {
  height: 64px;
  min-height: 64px;
  padding: 0 1.25rem;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid #334155;
}

.brand-name {
  font-size: 0.938rem;
  font-weight: 700;
  color: #ffffff;
  letter-spacing: -0.01em;
  line-height: 1.3;
  margin: 0;
}

.brand-subtitle {
  font-size: 0.75rem;
  color: #94a3b8;
  font-weight: 400;
  margin-top: 0.125rem;
}

.sidebar-nav {
  flex: 1;
  padding: 0.75rem 0.75rem 0;
  display: flex;
  flex-direction: column;
  gap: 0.125rem;
}

.nav-item {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 0 0.75rem;
  height: 40px;
  border-radius: 6px;
  color: #94a3b8;
  text-decoration: none;
  font-size: 0.875rem;
  font-weight: 500;
  transition: all 0.15s ease;
  border-left: 2px solid transparent;
}

.nav-item:hover {
  color: #e2e8f0;
  background: #334155;
}

.nav-item.active {
  color: #2563eb;
  background: #eff6ff;
  border-left-color: #2563eb;
  font-weight: 600;
}

.nav-item svg {
  flex-shrink: 0;
  opacity: 0.85;
}

.nav-item.active svg {
  opacity: 1;
}

/* === SIDEBAR FOOTER === */
.sidebar-footer {
  padding: 0.75rem;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.sidebar-footer-divider {
  height: 1px;
  background: #334155;
  margin-bottom: 0.25rem;
}

/* Dark sidebar overrides — pierce into child component styles */
.sidebar-footer :deep(.language-button),
.sidebar-footer :deep(.profile-button) {
  background: transparent;
  border-color: #334155;
  color: #94a3b8;
  width: 100%;
  justify-content: flex-start;
}

.sidebar-footer :deep(.language-button):hover,
.sidebar-footer :deep(.profile-button):hover {
  background: #334155;
  border-color: #475569;
  color: #e2e8f0;
}

.sidebar-footer :deep(.profile-name),
.sidebar-footer :deep(.language-label) {
  color: #cbd5e1;
}

.sidebar-footer :deep(.globe-icon),
.sidebar-footer :deep(.chevron) {
  color: #64748b;
}

/* Dropdowns open upward — prevent clipping at bottom of viewport */
.sidebar-footer :deep(.dropdown-menu) {
  top: auto;
  bottom: calc(100% + 0.5rem);
  right: auto;
  left: 0;
  z-index: 200;
  min-width: 200px;
}

/* === MAIN AREA === */
.main-wrapper {
  margin-left: 240px;
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  min-width: 0; /* allows table overflow-x: auto to work inside flex */
}

.main-content {
  flex: 1;
  padding: 1.5rem 2rem;
}

.page-header {
  margin-bottom: 1.5rem;
}

.page-header h2 {
  font-size: 1.875rem;
  font-weight: 700;
  color: #0f172a;
  margin-bottom: 0.375rem;
  letter-spacing: -0.025em;
}

.page-header p {
  color: #64748b;
  font-size: 0.938rem;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.25rem;
  margin-bottom: 1.5rem;
}

.stat-card {
  background: white;
  padding: 1.25rem;
  border-radius: 10px;
  border: 1px solid #e2e8f0;
  transition: all 0.2s ease;
}

.stat-card:hover {
  border-color: #cbd5e1;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
}

.stat-label {
  color: #64748b;
  font-size: 0.875rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 0.625rem;
}

.stat-value {
  font-size: 2.25rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
}

.stat-card.warning .stat-value {
  color: #ea580c;
}

.stat-card.success .stat-value {
  color: #059669;
}

.stat-card.danger .stat-value {
  color: #dc2626;
}

.stat-card.info .stat-value {
  color: #2563eb;
}

.card {
  background: white;
  border-radius: 10px;
  padding: 1.25rem;
  border: 1px solid #e2e8f0;
  margin-bottom: 1.25rem;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.875rem;
  border-bottom: 1px solid #e2e8f0;
}

.card-title {
  font-size: 1.125rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
}

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: #f8fafc;
  border-top: 1px solid #e2e8f0;
  border-bottom: 1px solid #e2e8f0;
}

th {
  text-align: left;
  padding: 0.5rem 0.75rem;
  font-weight: 600;
  color: #475569;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

td {
  padding: 0.5rem 0.75rem;
  border-top: 1px solid #f1f5f9;
  color: #334155;
  font-size: 0.875rem;
}

tbody tr {
  transition: background-color 0.15s ease;
}

tbody tr:hover {
  background: #f8fafc;
}

.badge {
  display: inline-block;
  padding: 0.313rem 0.75rem;
  border-radius: 6px;
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.025em;
}

.badge.success {
  background: #d1fae5;
  color: #065f46;
}

.badge.warning {
  background: #fed7aa;
  color: #92400e;
}

.badge.danger {
  background: #fecaca;
  color: #991b1b;
}

.badge.info {
  background: #dbeafe;
  color: #1e40af;
}

.badge.increasing {
  background: #d1fae5;
  color: #065f46;
}

.badge.decreasing {
  background: #fecaca;
  color: #991b1b;
}

.badge.stable {
  background: #e0e7ff;
  color: #3730a3;
}

.badge.high {
  background: #fecaca;
  color: #991b1b;
}

.badge.medium {
  background: #fed7aa;
  color: #92400e;
}

.badge.low {
  background: #dbeafe;
  color: #1e40af;
}

.loading {
  text-align: center;
  padding: 3rem;
  color: #64748b;
  font-size: 0.938rem;
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: 1rem;
  border-radius: 8px;
  margin: 1rem 0;
  font-size: 0.938rem;
}
</style>
