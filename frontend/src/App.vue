<template>
  <div v-if="!isLogin" class="app-layout animate-fade-in">
    <nav class="sidebar glass">
      <div class="sidebar-header">
        <h2 class="title-glow">ERP Nexus</h2>
      </div>
      
      <div class="sidebar-links">
        <router-link v-for="l in links" :key="l.path" :to="l.path" class="nav-link" active-class="nav-active">
          {{ l.label }}
        </router-link>
      </div>
      
      <div class="sidebar-footer">
        <button @click="logout" class="nav-link logout-btn">
          <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"></path><polyline points="16 17 21 12 16 7"></polyline><line x1="21" y1="12" x2="9" y2="12"></line></svg>
          Logout
        </button>
      </div>
    </nav>
    <main class="main-content">
      <div class="top-nav glass">
        <div class="breadcrumb">{{ currentRouteName }}</div>
        <div class="user-profile">
          <div class="avatar">A</div>
          <span>Admin User</span>
        </div>
      </div>
      <div class="content-area">
        <router-view v-slot="{ Component }">
          <transition name="fade" mode="out-in">
            <component :is="Component" />
          </transition>
        </router-view>
      </div>
    </main>
  </div>
  <div v-else class="login-layout">
    <router-view v-slot="{ Component }">
      <transition name="fade" mode="out-in">
        <component :is="Component" />
      </transition>
    </router-view>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useAuthStore } from './stores/authStore'

const route = useRoute()
const router = useRouter()
const auth = useAuthStore()

const isLogin = computed(() => route.path === '/login')

const links = [
  { path: '/assets', label: 'Assets Portfolio' },
  { path: '/assignments', label: 'Active Assignments' },
  { path: '/maintenance', label: 'Maintenance Hub' }
]

const currentRouteName = computed(() => {
  const match = links.find(l => l.path === route.path)
  return match ? match.label : 'Dashboard'
})

function logout() {
  auth.logout()
  router.push('/login')
}
</script>

<style scoped>
.app-layout {
  display: flex;
  height: 100vh;
  overflow: hidden;
}

.sidebar {
  width: 260px;
  display: flex;
  flex-direction: column;
  flex-shrink: 0;
  border-right: 1px solid var(--border-color);
  z-index: 10;
}

.sidebar-header {
  padding: 32px 24px;
}

.sidebar-header h2 {
  font-size: 1.5rem;
  letter-spacing: 1px;
}

.sidebar-links {
  padding: 0 16px;
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.nav-link {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  border-radius: var(--radius-sm);
  color: var(--text-secondary);
  text-decoration: none;
  font-weight: 500;
  transition: var(--transition);
}

.nav-link:hover {
  background: rgba(255, 255, 255, 0.05);
  color: var(--text-primary);
}

.nav-active {
  background: linear-gradient(90deg, rgba(79, 70, 229, 0.2), transparent);
  color: var(--primary-color) !important;
  border-left: 3px solid var(--primary-color);
}

.sidebar-footer {
  padding: 24px 16px;
}

.logout-btn {
  width: 100%;
  border: none;
  background: transparent;
  cursor: pointer;
  color: var(--danger-color);
  font-family: inherit;
  font-size: 0.95rem;
}

.logout-btn:hover {
  background: rgba(239, 68, 68, 0.1);
  color: var(--danger-hover);
}

.main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  position: relative;
}

.top-nav {
  height: 70px;
  padding: 0 32px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid var(--border-color);
  z-index: 5;
}

.breadcrumb {
  font-weight: 600;
  color: var(--text-primary);
  font-size: 1.1rem;
}

.user-profile {
  display: flex;
  align-items: center;
  gap: 12px;
}

.avatar {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: linear-gradient(135deg, var(--primary-color), var(--secondary-color));
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  color: white;
}

.content-area {
  flex: 1;
  padding: 32px;
  overflow-y: auto;
}

.login-layout {
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}

/* Page Transitions */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease, transform 0.3s ease;
}

.fade-enter-from {
  opacity: 0;
  transform: translateY(10px);
}

.fade-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}
</style>