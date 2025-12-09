<template>
  <Layout pageTitle="个人设置">
    <div class="profile-page">
      <div class="settings-card">
        <div class="card-header">
          <h3>修改密码</h3>
        </div>
        <div class="card-body">
          <el-form :model="passwordForm" :rules="passwordRules" ref="passwordFormRef" label-position="top" class="password-form">
            <el-form-item label="原密码" prop="oldPassword">
              <el-input v-model="passwordForm.oldPassword" type="password" placeholder="请输入原密码" show-password />
            </el-form-item>
            
            <el-form-item label="新密码" prop="newPassword">
              <el-input v-model="passwordForm.newPassword" type="password" placeholder="请输入新密码（至少6位）" show-password />
            </el-form-item>
            
            <el-form-item label="确认密码" prop="confirmPassword">
              <el-input v-model="passwordForm.confirmPassword" type="password" placeholder="请再次输入新密码" show-password />
            </el-form-item>
            
            <div class="form-actions">
              <el-button type="primary" :loading="loading" @click="handleChangePassword">
                更新密码
              </el-button>
              <el-button @click="resetPasswordForm">
                重置
              </el-button>
            </div>
          </el-form>
        </div>
      </div>
    </div>
  </Layout>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { changePassword } from '@/api/auth'
import { ElMessage } from 'element-plus'
import Layout from '@/components/Layout.vue'

const passwordFormRef = ref(null)
const loading = ref(false)

const passwordForm = reactive({
  oldPassword: '',
  newPassword: '',
  confirmPassword: ''
})

const validateConfirmPassword = (rule, value, callback) => {
  if (value === '') callback(new Error('请再次输入新密码'))
  else if (value !== passwordForm.newPassword) callback(new Error('两次输入的密码不一致'))
  else callback()
}

const passwordRules = {
  oldPassword: [{ required: true, message: '请输入原密码', trigger: 'blur' }],
  newPassword: [{ required: true, message: '请输入新密码', trigger: 'blur' }, { min: 6, message: '密码长度至少6位', trigger: 'blur' }],
  confirmPassword: [{ required: true, validator: validateConfirmPassword, trigger: 'blur' }]
}

const handleChangePassword = async () => {
  if (!passwordFormRef.value) return
  await passwordFormRef.value.validate(async (valid) => {
    if (!valid) return
    loading.value = true
    try {
      const response = await changePassword(passwordForm.oldPassword, passwordForm.newPassword)
      if (response.success) {
        ElMessage.success(response.message || '密码修改成功')
        resetPasswordForm()
      }
    } finally {
      loading.value = false
    }
  })
}

const resetPasswordForm = () => {
  passwordForm.oldPassword = ''
  passwordForm.newPassword = ''
  passwordForm.confirmPassword = ''
  if (passwordFormRef.value) passwordFormRef.value.clearValidate()
}
</script>

<style scoped>
.profile-page {
  max-width: 600px;
}

.settings-card {
  background: #ffffff;
  border: 1px solid #d0d7de;
  border-radius: 6px;
  overflow: hidden;
}

.card-header {
  padding: 12px 16px;
  border-bottom: 1px solid #d0d7de;
  background: #f6f8fa;
}

.card-header h3 {
  font-size: 14px;
  font-weight: 600;
  color: #1f2328;
  margin: 0;
}

.card-body {
  padding: 20px;
}

.password-form {
  max-width: 400px;
}

:deep(.el-form-item__label) {
  font-weight: 600;
  color: #1f2328;
}

:deep(.el-input__wrapper) {
  background: #f6f8fa;
}

.form-actions {
  display: flex;
  gap: 12px;
  margin-top: 24px;
}

.form-actions .el-button--primary {
  background: #2da44e;
  border-color: #2da44e;
}

.form-actions .el-button--primary:hover {
  background: #2c974b;
}
</style>
