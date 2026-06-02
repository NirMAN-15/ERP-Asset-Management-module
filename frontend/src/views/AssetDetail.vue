<template>
  <div class="page-container animate-fade-in" style="max-width:800px; margin:0 auto">
    <button @click="$router.back()" class="btn btn-outline mb-6" style="padding:6px 12px">
      <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="margin-right:6px"><line x1="19" y1="12" x2="5" y2="12"></line><polyline points="12 19 5 12 12 5"></polyline></svg>
      Back to Registry
    </button>
    
    <div v-if="asset" class="glass-card">
      <div class="flex justify-between items-start mb-8 border-b border-gray-200 pb-6">
        <div>
          <h2 class="text-3xl font-bold text-primary mb-2">{{ asset.name }}</h2>
          <p class="text-secondary font-mono">{{ asset.asset_code }}</p>
        </div>
        <span class="badge text-sm" :class="badgeClass(asset.status)" style="padding:6px 14px">{{ asset.status }}</span>
      </div>
      
      <div class="grid-2-col gap-8 mb-8">
        <div class="info-group">
          <div class="info-label">Category</div>
          <div class="info-value category-pill">{{ asset.category }}</div>
        </div>
        <div class="info-group">
          <div class="info-label">Location</div>
          <div class="info-value">{{ asset.location || 'Unassigned' }}</div>
        </div>
        <div class="info-group">
          <div class="info-label">Serial Number</div>
          <div class="info-value font-mono">{{ asset.serial_number || 'N/A' }}</div>
        </div>
        <div class="info-group">
          <div class="info-label">Purchase Order ID</div>
          <div class="info-value">{{ asset.purchase_order_id || '—' }}</div>
        </div>
        <div class="info-group">
          <div class="info-label">Initial Purchase Value</div>
          <div class="info-value">LKR {{ asset.purchase_value?.toLocaleString() }}</div>
        </div>
        <div class="info-group">
          <div class="info-label">Current Depreciated Value</div>
          <div class="info-value font-bold text-primary">LKR {{ asset.current_value?.toLocaleString() }}</div>
        </div>
      </div>
      
      <div class="flex gap-4 border-t border-white/10 pt-6 mt-4">
        <button class="btn btn-outline flex-1">
          <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="margin-right:8px"><path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"></path><path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"></path></svg>
          Modify Asset
        </button>
        <button @click="disposeAsset" class="btn btn-danger flex-1">
          <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="margin-right:8px"><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path></svg>
          Dispose Asset
        </button>
      </div>
    </div>
    
    <div v-else class="p-12 text-center">
      <div class="spinner"></div>
      <p class="mt-4 text-secondary">Fetching asset details...</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import api from '../api/axios'

const route = useRoute()
const router = useRouter()
const asset = ref(null)

onMounted(async () => {
  try {
    const res = await api.get(`/assets/${route.params.id}`)
    asset.value = res.data.data
  } catch (err) {
    alert('Asset not found.')
    router.push('/assets')
  }
})

async function disposeAsset() {
  if(confirm('Are you absolutely sure you want to permanently dispose this asset?')) {
    await api.delete(`/assets/${asset.value.id}`)
    router.push('/assets')
  }
}

const badgeClass = (status) => {
  if (status === 'ACTIVE') return 'badge-success'
  if (status === 'MAINTENANCE') return 'badge-warning'
  return 'badge-danger'
}
</script>

<style scoped>
.grid-2-col {
  display: grid;
  grid-template-columns: 1fr 1fr;
}
.info-group {
  display: flex;
  flex-direction: column;
  gap: 4px;
}
.info-label {
  font-size: 0.85rem;
  color: var(--text-secondary);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  font-weight: 600;
}
.info-value {
  font-size: 1.1rem;
  color: var(--text-primary);
}
.category-pill {
  display: inline-block;
  background: rgba(255, 255, 255, 0.1);
  padding: 4px 10px;
  border-radius: 6px;
  font-size: 0.9rem;
  width: max-content;
}
.spinner {
  width: 40px;
  height: 40px;
  border: 3px solid rgba(79, 70, 229, 0.3);
  border-radius: 50%;
  border-top-color: var(--primary-color);
  animation: spin 1s ease-in-out infinite;
  margin: 0 auto;
}
@keyframes spin { to { transform: rotate(360deg); } }
.text-primary { color: #818CF8; }
.font-mono { font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace; }
</style>