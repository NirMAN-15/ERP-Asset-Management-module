<template>
  <div class="page-container animate-fade-in">
    <div class="page-header flex justify-between items-center mb-8">
      <div>
        <h1 class="title-glow text-3xl">Asset Portfolio</h1>
        <p class="text-secondary mt-2">Manage and track company resources in real-time</p>
      </div>
      <button @click="showForm = true" class="btn btn-primary">
        <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="margin-right:8px"><line x1="12" y1="5" x2="12" y2="19"></line><line x1="5" y1="12" x2="19" y2="12"></line></svg>
        Provision Asset
      </button>
    </div>

    <!-- Stats -->
    <div class="stats-grid mb-8">
      <div v-for="(s, index) in stats" :key="s.label" class="glass-card stat-card" :class="'delay-' + (index + 1)">
        <div class="stat-value">{{ s.val }}</div>
        <div class="stat-label">{{ s.label }}</div>
      </div>
    </div>

    <!-- Controls -->
    <div class="controls-bar glass mb-4 p-4 rounded-lg flex gap-4 items-center">
      <div class="filter-group">
        <label class="text-secondary text-sm mr-3">Status Filter:</label>
        <select v-model="filterStatus" @change="load" class="form-select inline-select">
          <option value="">All Statuses</option>
          <option>ACTIVE</option>
          <option>MAINTENANCE</option>
          <option>DISPOSED</option>
        </select>
      </div>
      <div class="filter-group">
        <label class="text-secondary text-sm mr-3">Category Filter:</label>
        <select v-model="filterCat" @change="load" class="form-select inline-select">
          <option value="">All Categories</option>
          <option>IT</option>
          <option>Furniture</option>
          <option>Vehicle</option>
        </select>
      </div>
    </div>

    <!-- Table -->
    <div class="glass table-container">
      <div v-if="store.loading" class="p-8 text-center text-secondary">
        <div class="spinner"></div>
        <p class="mt-4">Synchronizing registry...</p>
      </div>
      
      <table v-else class="table">
        <thead>
          <tr>
            <th>Asset Code</th>
            <th>Asset Name</th>
            <th>Category</th>
            <th>Location</th>
            <th>Status</th>
            <th class="text-right">Valuation (LKR)</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="a in store.assets" :key="a.id" @click="$router.push('/assets/'+a.id)" class="cursor-pointer hover:bg-white/5 transition-colors">
            <td class="font-mono text-primary font-medium">{{ a.asset_code }}</td>
            <td class="font-medium text-primary">{{ a.name }}</td>
            <td>
              <span class="category-pill">{{ a.category }}</span>
            </td>
            <td class="text-secondary">{{ a.location }}</td>
            <td>
              <span class="badge" :class="badgeClass(a.status)">{{ a.status }}</span>
            </td>
            <td class="text-right font-mono">{{ a.current_value.toLocaleString() }}</td>
          </tr>
          
          <tr v-if="store.assets.length === 0">
            <td colspan="6" class="text-center p-8 text-secondary">
              No assets found matching the criteria.
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Modal -->
    <transition name="fade">
      <div v-if="showForm" class="modal-overlay">
        <div class="modal-content glass-card p-0 overflow-hidden">
          <div class="modal-header p-6">
            <h2 class="text-xl font-semibold">Provision New Asset</h2>
          </div>
          
          <div class="modal-body p-6 max-h-[70vh] overflow-y-auto">
            <div v-if="submitError" class="bg-red-500/20 border border-red-500/50 text-red-200 p-4 rounded-md mb-6">
              {{ submitError }}
            </div>

            <div class="grid-2-col gap-4">
              <div class="form-group">
                <label class="form-label">Asset Name <span class="text-red-400">*</span></label>
                <input v-model="newAsset.name" class="form-input" placeholder="e.g. MacBook Pro M3">
              </div>
              
              <div class="form-group">
                <label class="form-label">Category <span class="text-red-400">*</span></label>
                <select v-model="newAsset.category" class="form-select">
                  <option value="">Select category...</option>
                  <option>IT</option>
                  <option>Furniture</option>
                  <option>Vehicle</option>
                  <option>Equipment</option>
                  <option>Other</option>
                </select>
              </div>

              <div class="form-group">
                <label class="form-label">Serial Number</label>
                <input v-model="newAsset.serial_number" class="form-input" placeholder="SN-XXXXX">
              </div>

              <div class="form-group">
                <label class="form-label">Location</label>
                <input v-model="newAsset.location" class="form-input" placeholder="Server Room A">
              </div>

              <div class="form-group">
                <label class="form-label">Purchase Date <span class="text-red-400">*</span></label>
                <input v-model="newAsset.purchase_date" type="date" class="form-input">
              </div>

              <div class="form-group">
                <label class="form-label">Purchase Value (LKR) <span class="text-red-400">*</span></label>
                <input v-model="newAsset.purchase_value" type="number" class="form-input" placeholder="0.00">
              </div>

              <div class="form-group">
                <label class="form-label">Asset Code <span class="text-red-400">*</span></label>
                <input v-model="newAsset.asset_code" class="form-input" placeholder="AST-001">
              </div>

              <div class="form-group">
                <label class="form-label">PO ID</label>
                <input v-model="newAsset.purchase_order_id" class="form-input" placeholder="PO-0001">
              </div>
            </div>
          </div>
          
          <div class="modal-footer p-6 flex justify-end gap-3">
            <button @click="showForm = false" class="btn btn-outline">Cancel</button>
            <button @click="submitAsset" class="btn btn-primary">Commit to Registry</button>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useAssetStore } from '../stores/assetStore'

const store = useAssetStore()
const filterStatus = ref('')
const filterCat = ref('')

const load = () => store.fetchAssets({ status: filterStatus.value, category: filterCat.value })
onMounted(load)

const showForm = ref(false)

const stats = computed(() => [
  { val: store.assets.length, label: 'Total Assessed Units' },
  { val: store.assets.filter(a => a.status === 'ACTIVE').length, label: 'Active Deployment' },
  { val: store.assets.filter(a => a.status === 'MAINTENANCE').length, label: 'Under Maintenance' },
  { val: 'Rs. ' + store.assets.reduce((s, a) => s + a.current_value, 0).toLocaleString(), label: 'Total Valuation' },
])

const badgeClass = (status) => {
  if (status === 'ACTIVE') return 'badge-success'
  if (status === 'MAINTENANCE') return 'badge-warning'
  return 'badge-danger'
}

const newAsset = ref({
  name: '', category: '', serial_number: '', location: '', purchase_date: '', purchase_value: 0, asset_code: '', purchase_order_id: ''
})
const submitError = ref('')

const submitAsset = async () => {
  submitError.value = ''
  if (!newAsset.value.name || !newAsset.value.category || !newAsset.value.purchase_date || !newAsset.value.purchase_value || !newAsset.value.asset_code) {
    submitError.value = 'Please fill in all mandatory fields denoted by (*)'
    return
  }
  try {
    const payload = {
      ...newAsset.value,
      purchase_value: parseFloat(newAsset.value.purchase_value) || 0,
      purchase_date: newAsset.value.purchase_date + 'T00:00:00Z'
    }
    await store.createAsset(payload)
    showForm.value = false
    newAsset.value = { name: '', category: '', serial_number: '', location: '', purchase_date: '', purchase_value: 0, asset_code: '', purchase_order_id: '' }
    load()
  } catch (err) {
    submitError.value = err.response?.data?.error || err.message || 'Registry creation failed due to system error'
  }
}
</script>

<style scoped>
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 20px;
}

.stat-card {
  padding: 24px;
  text-align: left;
  border-left: 4px solid var(--primary-color);
}

.stat-card:nth-child(2) { border-left-color: var(--secondary-color); }
.stat-card:nth-child(3) { border-left-color: #FBBF24; }
.stat-card:nth-child(4) { border-left-color: #A855F7; }

.stat-value {
  font-size: 2rem;
  font-weight: 700;
  color: white;
  line-height: 1.2;
  margin-bottom: 8px;
}

.stat-label {
  font-size: 0.875rem;
  color: var(--text-secondary);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  font-weight: 600;
}

.inline-select {
  width: auto;
  min-width: 160px;
  display: inline-block;
  padding-top: 8px;
  padding-bottom: 8px;
}

.category-pill {
  background: rgba(255, 255, 255, 0.1);
  padding: 4px 10px;
  border-radius: 6px;
  font-size: 0.8rem;
  font-weight: 500;
}

.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(15, 23, 42, 0.85);
  backdrop-filter: blur(8px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  width: 100%;
  max-width: 700px;
  margin: 20px;
}

.grid-2-col {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
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

@keyframes spin {
  to { transform: rotate(360deg); }
}

.text-primary { color: #818CF8; }
.font-mono { font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace; }
</style>