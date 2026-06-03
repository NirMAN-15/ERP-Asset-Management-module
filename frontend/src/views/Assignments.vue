<template>
  <div class="page-container animate-fade-in">
    <div class="page-header flex justify-between items-center mb-8">
      <div>
        <h1 class="title-glow text-3xl">Asset Assignments</h1>
        <p class="text-secondary mt-2">Track which employee holds which asset</p>
      </div>
      <button @click="showForm = true" class="btn btn-primary">
        <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" style="margin-right:8px"><path d="M16 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"></path><circle cx="8.5" cy="7" r="4"></circle><line x1="20" y1="8" x2="20" y2="14"></line><line x1="23" y1="11" x2="17" y2="11"></line></svg>
        New Assignment
      </button>
    </div>

    <!-- Table -->
    <div class="glass table-container">
      <div v-if="loading" class="p-8 text-center text-secondary">
        <div class="spinner"></div>
        <p class="mt-4">Loading assignment records...</p>
      </div>
      <table v-else class="table">
        <thead>
          <tr>
            <th>Asset ID</th>
            <th>Employee ID</th>
            <th>Assigned Date</th>
            <th>Notes</th>
            <th class="text-right">Action</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="a in assignments" :key="a.id" class="hover:bg-white/5 transition-colors">
            <td class="font-mono text-primary font-medium">{{ a.asset_id }}</td>
            <td class="font-medium">{{ a.employee_id }}</td>
            <td class="text-secondary">{{ formatDate(a.assigned_at) }}</td>
            <td class="text-secondary italic">{{ a.notes || '—' }}</td>
            <td class="text-right">
              <button @click="returnAsset(a.id)" class="btn btn-outline border-primary text-primary hover:bg-primary hover:text-white" style="padding:6px 12px; font-size:0.8rem">
                Mark Returned
              </button>
            </td>
          </tr>
          <tr v-if="assignments.length === 0">
            <td colspan="5" class="text-center p-8 text-secondary">
              No active assignments currently recorded.
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
            <h2 class="text-xl font-semibold">Assign Asset to Employee</h2>
          </div>
          
          <div class="modal-body p-6">
            <div class="form-group">
              <label class="form-label">Select Asset <span class="text-red-400">*</span></label>
              <select v-model="form.asset_id" class="form-select">
                <option value="">Select an active asset...</option>
                <option v-for="a in assets" :key="a.id" :value="a.id">{{ a.asset_code }} — {{ a.name }}</option>
              </select>
            </div>
            
            <div class="form-group">
              <label class="form-label">Employee ID <span class="text-red-400">*</span></label>
              <input v-model="form.employee_id" class="form-input" placeholder="e.g. EMP-1001">
            </div>
            
            <div class="form-group">
              <label class="form-label">Notes (Optional)</label>
              <textarea v-model="form.notes" class="form-input" placeholder="Additional details..." rows="3" style="resize:vertical"></textarea>
            </div>
          </div>
          
          <div class="modal-footer p-6 flex justify-end gap-3">
            <button @click="showForm = false" class="btn btn-outline">Cancel</button>
            <button @click="submit" class="btn btn-primary">Create Assignment</button>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import api from '../api/axios'

const assignments = ref([])
const assets = ref([])
const loading = ref(false)
const showForm = ref(false)
const form = ref({ asset_id: '', employee_id: '', notes: '' })

async function load() {
  loading.value = true
  try {
    const [a, b] = await Promise.all([
      api.get('/assignments'),
      api.get('/assets?status=ACTIVE')
    ])
    assignments.value = a.data.data || []
    assets.value = b.data.data || []
  } finally { loading.value = false }
}

async function submit() {
  if (!form.value.asset_id || !form.value.employee_id) {
    alert('Please select an asset and enter an Employee ID.')
    return
  }
  try {
    await api.post('/assignments', { ...form.value, assigned_by: 'admin' })
    form.value = { asset_id: '', employee_id: '', notes: '' }
    showForm.value = false
    load()
  } catch (err) {
    alert(err.response?.data?.error || 'Failed to create assignment')
  }
}

async function returnAsset(id) {
  if (confirm('Confirm asset return?')) {
    await api.put(`/assignments/${id}/return`, {})
    load()
  }
}

const formatDate = d => d ? new Date(d).toLocaleDateString(undefined, { year: 'numeric', month: 'short', day: 'numeric' }) : '—'

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
.border-primary { border-color: #818CF8; }
.hover\:bg-primary:hover { background-color: var(--primary-color); }
.font-mono { font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace; }
</style>