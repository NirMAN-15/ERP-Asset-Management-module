<template>
  <div class="moodle-login-wrapper">
    <div class="moodle-login-card">
      <div class="moodle-login-header">
        <!-- Replicating the ITUM LMS Logo / Header style -->
        <div class="logo-circle">
          <svg xmlns="http://www.w3.org/2000/svg" width="40" height="40" viewBox="0 0 24 24" fill="none" stroke="#fff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 2L2 7l10 5 10-5-10-5z"/><path d="M2 17l10 5 10-5M2 12l10 5 10-5"/></svg>
        </div>
        <h1 class="moodle-login-title">ERP Asset Management</h1>
        <p class="moodle-login-subtitle">Log in to your account</p>
      </div>
      
      <form @submit.prevent="login" class="moodle-login-form">
        <div class="moodle-form-group">
          <div class="moodle-input-wrapper">
            <span class="moodle-input-icon">
              <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"></path><circle cx="12" cy="7" r="4"></circle></svg>
            </span>
            <input 
              v-model="username" 
              class="moodle-form-input" 
              placeholder="Username" 
              required
              autocomplete="username"
            >
          </div>
        </div>
        
        <div class="moodle-form-group">
          <div class="moodle-input-wrapper">
            <span class="moodle-input-icon">
              <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="11" width="18" height="11" rx="2" ry="2"></rect><path d="M7 11V7a5 5 0 0 1 10 0v4"></path></svg>
            </span>
            <input 
              v-model="password" 
              type="password" 
              class="moodle-form-input" 
              placeholder="Password" 
              required
              autocomplete="current-password"
            >
          </div>
        </div>
        
        <button type="submit" class="moodle-btn-primary" :disabled="loading">
          <span v-if="loading">Authenticating...</span>
          <span v-else>Log in</span>
        </button>

        <div class="moodle-login-footer">
          <a href="#" class="moodle-forgot-link">Forgotten your username or password?</a>
          <div class="moodle-guest-login">
            <p>Some courses may allow guest access</p>
            <button type="button" class="moodle-btn-secondary">Log in as a guest</button>
          </div>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { useAuthStore } from '../stores/authStore'

const username = ref('')
const password = ref('')
const loading = ref(false)
const router = useRouter()
const auth = useAuthStore()

const login = async () => {
  if (!username.value || !password.value) return
  
  loading.value = true
  try {
    await auth.login(username.value, password.value)
    router.push('/assets')
  } catch (err) {
    alert('Invalid login, please try again')
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.moodle-login-wrapper {
  width: 100%;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #f4f4f4;
  background-image: url('https://images.unsplash.com/photo-1541339907198-e08756dedf3f?ixlib=rb-4.0.3&auto=format&fit=crop&w=2070&q=80');
  background-size: cover;
  background-position: center;
  position: relative;
}

.moodle-login-wrapper::before {
  content: '';
  position: absolute;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0, 0, 0, 0.6);
}

.moodle-login-card {
  position: relative;
  width: 100%;
  max-width: 450px;
  background: #ffffff;
  border-radius: 4px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
  overflow: hidden;
  margin: 20px;
}

.moodle-login-header {
  text-align: center;
  padding: 40px 30px 20px;
}

.logo-circle {
  width: 80px;
  height: 80px;
  background: #7a1b38; /* ITUM Maroon-like color */
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto 20px;
  box-shadow: 0 4px 10px rgba(122, 27, 56, 0.3);
}

.moodle-login-title {
  font-size: 1.5rem;
  color: #333333;
  margin-bottom: 8px;
  font-weight: 400;
}

.moodle-login-subtitle {
  color: #666666;
  font-size: 0.95rem;
}

.moodle-login-form {
  padding: 0 40px 40px;
}

.moodle-form-group {
  margin-bottom: 20px;
}

.moodle-input-wrapper {
  position: relative;
  display: flex;
  align-items: center;
  border: 1px solid #cccccc;
  border-radius: 4px;
  background: #f9f9f9;
  transition: border-color 0.2s;
}

.moodle-input-wrapper:focus-within {
  border-color: #7a1b38;
  background: #ffffff;
}

.moodle-input-icon {
  padding: 12px;
  color: #888888;
  border-right: 1px solid #eeeeee;
  display: flex;
}

.moodle-form-input {
  flex: 1;
  padding: 12px 16px;
  border: none;
  background: transparent;
  font-size: 1rem;
  color: #333333;
  outline: none;
}

.moodle-btn-primary {
  width: 100%;
  padding: 12px;
  background: #0f6cb6; /* Standard Moodle Blue */
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 1.1rem;
  font-weight: 400;
  cursor: pointer;
  transition: background 0.2s;
  margin-top: 10px;
}

.moodle-btn-primary:hover {
  background: #0b5592;
}

.moodle-login-footer {
  margin-top: 24px;
  text-align: center;
  border-top: 1px solid #eeeeee;
  padding-top: 24px;
}

.moodle-forgot-link {
  color: #0f6cb6;
  text-decoration: none;
  font-size: 0.9rem;
}

.moodle-forgot-link:hover {
  text-decoration: underline;
}

.moodle-guest-login {
  margin-top: 20px;
}

.moodle-guest-login p {
  font-size: 0.85rem;
  color: #666666;
  margin-bottom: 10px;
}

.moodle-btn-secondary {
  padding: 8px 20px;
  background: #f4f4f4;
  color: #333333;
  border: 1px solid #cccccc;
  border-radius: 4px;
  font-size: 0.9rem;
  cursor: pointer;
}

.moodle-btn-secondary:hover {
  background: #e9e9e9;
}
</style>
