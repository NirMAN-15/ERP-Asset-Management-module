<template>
  <div class="page-container animate-fade-in">
    <div class="page-header flex justify-between items-center mb-8">
      <div>
        <h1 class="title-glow text-3xl">Maintenance Hub</h1>
        <p class="text-secondary mt-2">Schedule and monitor asset maintenance lifecycles</p>
      </div>
      <button @click="showForm = true" class="btn btn-primary">
        <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="margin-right:8px"><path d="M14.7 6.3a1 1 0 0 0 0 1.4l1.6 1.6a1 1 0 0 0 1.4 0l3.77-3.77a6 6 0 0 1-7.94 7.94l-6.91 6.91a2.12 2.12 0 0 1-3-3l6.91-6.91a6 6 0 0 1 7.94-7.94l-3.76 3.76z"></path></svg>
        Schedule Task
      </button>
    </div>

    <!-- Table -->
    <div class="glass table-container">
      <div v-if="loading" class="p-8 text-center text-secondary">
        <div class="spinner"></div>
        <p class="mt-4">Loading maintenance records...</p>
      </div>
      <table v-else class="table">
        <thead>
          <tr>
            <th>Asset ID</th>
            <th>Task Description</th>
            <th>Scheduled Date</th>
            <th class="text-right">Est. Cost (LKR)</th>
            <th>Status</th>
            <th class="text-right">Action</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="m in records" :key="m.id" class="hover:bg-white/5 transition-colors">
            <td class="font-mono text-primary font-medium">{{ m.asset_id }}</td>
            <td class="font-medium">{{ m.description }}</td>
            <td class="text-secondary">{{ formatDate(m.scheduled_date) }}</td>
            <td class="text-right font-mono">{{ m.cost ? m.cost.toLocaleString() : '—' }}</td>
            <td>
              <span class="badge" :class="badgeClass(m.status)">{{ m.status }}</span>
            </td>
            <td class="text-right">
              <button v-if="m.status === 'SCHEDULED'" @click="complete(m.id)" class="btn btn-outline border-secondary text-secondary hover:bg-secondary hover:text-white" style="padding:6px 12px; font-size:0.8rem">
                Mark Done
              </button>
            </td>
          </tr>
          <tr v-if="records.length === 0">
            <td colspan="6" class="text-center p-8 text-secondary">
              No maintenance tasks scheduled.
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Modal Form -->
    <transition name="fade">
      <div v-if="showForm" class="modal-overlay">
        <div class="modal-content glass-card p-0 overflow-hidden" style="max-width:500px">
          <div class="modal-header p-6">
            <h2 class="text-xl font-semibold">Schedule Maintenance Task</h2>
          </div>
          
          <div class="modal-body p-6">
            <div class="form-group">
              <label class="form-label">Target Asset <span class="text-red-400">*</span></label>
              <select v-model="form.asset_id" class="form-select">
                <option value="">Select asset...</option>
                <option v-for="a in assets" :key="a.id" :value="a.id">{{ a.asset_code }} — {{ a.name }}</option>
              </select>
            </div>
            
            <div class="form-group">
              <label class="form-label">Task Description <span class="text-red-400">*</span></label>
              <input v-model="form.description" class="form-input" placeholder="e.g. Annual hardware servicing">
            </div>
            
            <div class="form-group">
              <label class="form-label">Scheduled Date <span class="text-red-400">*</span></label>
              <input v-model="form.scheduled_date" type="date" class="form-input">
            </div>
            
            <div class="form-group">
              <label class="form-label">Estimated Cost (LKR)</label>
              <input v-model="form.cost" type="number" class="form-input" placeholder="0.00">
            </div>
          </div>
          
          <div class="modal-footer p-6 flex justify-end gap-3">
            <button @click="showForm = false" class="btn btn-outline">Cancel</button>
            <button @click="submit" class="btn btn-primary">Schedule Task</button>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import api from '../api/axios'

const records = ref([])
const assets = ref([])
const loading = ref(false)
const showForm = ref(false)
const form = ref({ asset_id: '', description: '', scheduled_date: '', cost: '' })

async function load() {
  loading.value = true
  try {
    const [m, a] = await Promise.all([
      api.get('/maintenance'),
      api.get('/assets')
    ])
    records.value = m.data.data || []
    assets.value = a.data.data || []
  } finally { loading.value = false }
}

async function submit() {
  if (!form.value.asset_id || !form.value.description || !form.value.scheduled_date) {
    alert('Please fill in all required fields.')
    return
  }
  try {
    const payload = {
      ...form.value,
      cost: parseFloat(form.value.cost) || 0,
      scheduled_date: form.value.scheduled_date + 'T00:00:00Z'
    }
    await api.post('/maintenance', payload)
    form.value = { asset_id: '', description: '', scheduled_date: '', cost: '' }
    showForm.value = false
    load()
  } catch (err) {
    alert(err.response?.data?.error || 'Failed to schedule maintenance')
  }
}

async function complete(id) {
  if (confirm('Mark this task as fully completed?')) {
    await api.put(`/maintenance/${id}/complete`, { cost: 0 })
    load()
  }
}

const formatDate = d => d ? new Date(d).toLocaleDateString(undefined, { year: 'numeric', month: 'short', day: 'numeric' }) : '—'

const badgeClass = (status) => {
  if (status === 'COMPLETED') return 'badge-success'
  if (status === 'SCHEDULED') return 'badge-warning'
  return 'badge-danger'
}

onMounted(load)
</script>

<style scoped>
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
  margin: 20px;
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
.text-secondary-color { color: #10B981; }
.border-secondary { border-color: #10B981; }
.hover\:bg-secondary:hover { background-color: var(--secondary-color); }
.font-mono { font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace; }
</style>