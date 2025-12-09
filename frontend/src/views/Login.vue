<template>
  <div class="login-page">
    <div class="login-box">
      <div class="login-header">
        <svg height="48" viewBox="0 0 24 24" width="48" fill="#1f2328">
          <path d="M12 2L2 7l10 5 10-5-10-5zM2 17l10 5 10-5M2 12l10 5 10-5"/>
        </svg>
        <h1>登录到教育系统</h1>
      </div>
      
      <div class="login-card">
        <el-form :model="loginForm" :rules="rules" ref="loginFormRef">
          <el-form-item prop="email">
            <label class="form-label">邮箱地址</label>
            <el-input v-model="loginForm.email" placeholder="you@example.com" size="large" />
          </el-form-item>
          
          <el-form-item prop="password">
            <label class="form-label">密码</label>
            <el-input
              v-model="loginForm.password"
              type="password"
              placeholder="请输入密码"
              size="large"
              show-password
              @keyup.enter="handleLogin"
            />
          </el-form-item>
          
          <el-button
            type="primary"
            size="large"
            :loading="loading"
            @click="handleLogin"
            class="login-btn"
          >
            登录
          </el-button>
        </el-form>
      </div>
      
      <div class="login-footer">
        <p>还没有账号？<router-link to="/register">创建账号</router-link></p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { useUserStore } from '@/stores/user'

const router = useRouter()
const route = useRoute()
const userStore = useUserStore()

const loginFormRef = ref(null)
const loading = ref(false)

const loginForm = reactive({
  email: '',
  password: ''
})

const rules = {
  email: [
    { required: true, message: '请输入邮箱', trigger: 'blur' },
    { type: 'email', message: '请输入正确的邮箱格式', trigger: 'blur' }
  ],
  password: [
    { required: true, message: '请输入密码', trigger: 'blur' },
    { min: 6, message: '密码长度至少6位', trigger: 'blur' }
  ]
}

const handleLogin = async () => {
  if (!loginFormRef.value) return
  await loginFormRef.value.validate(async (valid) => {
    if (!valid) return
    loading.value = true
    try {
      const result = await userStore.loginAction(loginForm.email, loginForm.password)
      if (result.success) {
        router.push(route.query.redirect || '/dashboard')
      }
    } finally {
      loading.value = false
    }
  })
}
</script>

<style scoped>
.login-page {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f6f8fa;
  padding: 40px 20px;
}

.login-box {
  width: 100%;
  max-width: 340px;
}

.login-header {
  text-align: center;
  margin-bottom: 24px;
}

.login-header svg {
  margin-bottom: 16px;
}

.login-header h1 {
  font-size: 24px;
  font-weight: 300;
  color: #1f2328;
  margin: 0;
}

.login-card {
  background: #ffffff;
  border: 1px solid #d0d7de;
  border-radius: 6px;
  padding: 20px;
}

.form-label {
  display: block;
  font-size: 14px;
  font-weight: 600;
  color: #1f2328;
  margin-bottom: 8px;
}

:deep(.el-form-item) {
  margin-bottom: 16px;
}

:deep(.el-input__wrapper) {
  background: #f6f8fa;
}

.login-btn {
  width: 100%;
  background: #2da44e;
  border-color: #2da44e;
  font-weight: 500;
}

.login-btn:hover {
  background: #2c974b;
  border-color: #2c974b;
}

.login-footer {
  margin-top: 16px;
  padding: 16px 20px;
  background: #ffffff;
  border: 1px solid #d0d7de;
  border-radius: 6px;
  text-align: center;
  font-size: 14px;
  color: #1f2328;
}

.login-footer a {
  color: #0969da;
  font-weight: 500;
}
</style>
