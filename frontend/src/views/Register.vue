<template>
  <div class="register-page">
    <div class="register-box">
      <div class="register-header">
        <svg height="48" viewBox="0 0 24 24" width="48" fill="#1f2328">
          <path d="M12 2L2 7l10 5 10-5-10-5zM2 17l10 5 10-5M2 12l10 5 10-5"/>
        </svg>
        <h1>创建账号</h1>
        <p>加入教育系统，开始学习之旅</p>
      </div>
      
      <div class="register-card">
        <el-form :model="registerForm" :rules="rules" ref="registerFormRef" label-position="top">
          <el-form-item label="选择角色" prop="role">
            <div class="role-tabs">
              <button type="button" :class="['role-tab', { active: registerForm.role === 'student' }]" @click="registerForm.role = 'student'">
                🎓 学生
              </button>
              <button type="button" :class="['role-tab', { active: registerForm.role === 'teacher' }]" @click="registerForm.role = 'teacher'">
                👨‍🏫 教师
              </button>
            </div>
          </el-form-item>
          
          <el-form-item label="邮箱地址" prop="email">
            <el-input v-model="registerForm.email" placeholder="you@example.com" />
          </el-form-item>
          
          <el-form-item label="验证码" prop="verificationCode">
            <div class="code-row">
              <el-input v-model="registerForm.verificationCode" placeholder="6位验证码" />
              <el-button @click="sendCode" :disabled="countdown > 0" :loading="sendingCode">
                {{ countdown > 0 ? `${countdown}s` : '发送' }}
              </el-button>
            </div>
          </el-form-item>
          
          <div class="form-row">
            <el-form-item label="密码" prop="password">
              <el-input v-model="registerForm.password" type="password" placeholder="至少6位" show-password />
            </el-form-item>
            <el-form-item label="确认密码" prop="confirmPassword">
              <el-input v-model="registerForm.confirmPassword" type="password" placeholder="再次输入" show-password />
            </el-form-item>
          </div>
          
          <el-form-item label="真实姓名" prop="realName">
            <el-input v-model="registerForm.realName" placeholder="请输入真实姓名" />
          </el-form-item>
          
          <template v-if="registerForm.role === 'student'">
            <el-form-item label="学号" prop="studentNumber">
              <el-input v-model="registerForm.studentNumber" placeholder="请输入学号" />
            </el-form-item>
            
            <el-form-item label="人脸照片">
              <el-upload :auto-upload="false" :on-change="handleFileChange" :limit="1" accept="image/*" list-type="picture-card" :file-list="fileList" class="photo-upload">
                <div class="upload-trigger">📷 上传</div>
              </el-upload>
              <p class="form-hint">请上传清晰的正面照片用于身份验证</p>
              
              <div v-if="faceVerified" class="verify-badge success">
                ✓ 人脸验证通过 ({{ (similarity * 100).toFixed(1) }}%)
              </div>
            </el-form-item>
            
            <el-button v-if="registerForm.photo && !faceVerified" @click="handleFaceVerification" :loading="verifying" class="verify-btn">
              🔍 验证人脸
            </el-button>
          </template>
          
          <el-button type="primary" :loading="loading" @click="handleRegister" :disabled="registerForm.role === 'student' && !faceVerified" class="submit-btn">
            {{ registerForm.role === 'teacher' ? '提交注册（需审核）' : '创建账号' }}
          </el-button>
          
          <p v-if="registerForm.role === 'student' && !faceVerified && registerForm.photo" class="form-hint warning">
            ⚠️ 请先完成人脸验证
          </p>
        </el-form>
      </div>
      
      <div class="register-footer">
        <p>已有账号？<router-link to="/login">立即登录</router-link></p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { useRouter } from 'vue-router'
import { useUserStore } from '@/stores/user'
import { sendVerificationCode } from '@/api/auth'
import { ElMessage } from 'element-plus'
import request from '@/utils/request'

const router = useRouter()
const userStore = useUserStore()
const registerFormRef = ref(null)
const loading = ref(false)
const sendingCode = ref(false)
const countdown = ref(0)
const verifying = ref(false)
const faceVerified = ref(false)
const similarity = ref(0)
const fileList = ref([])
const rosterId = ref(null)

const registerForm = reactive({
  role: 'student',
  email: '',
  verificationCode: '',
  password: '',
  confirmPassword: '',
  realName: '',
  studentNumber: '',
  photo: null
})

const validateConfirmPassword = (rule, value, callback) => {
  if (value === '') callback(new Error('请再次输入密码'))
  else if (value !== registerForm.password) callback(new Error('两次密码不一致'))
  else callback()
}

const rules = {
  role: [{ required: true, message: '请选择角色', trigger: 'change' }],
  email: [{ required: true, message: '请输入邮箱', trigger: 'blur' }, { type: 'email', message: '邮箱格式错误', trigger: 'blur' }],
  verificationCode: [{ required: true, message: '请输入验证码', trigger: 'blur' }],
  password: [{ required: true, message: '请输入密码', trigger: 'blur' }, { min: 6, message: '至少6位', trigger: 'blur' }],
  confirmPassword: [{ required: true, validator: validateConfirmPassword, trigger: 'blur' }],
  realName: [{ required: true, message: '请输入姓名', trigger: 'blur' }],
  studentNumber: [{ required: true, message: '请输入学号', trigger: 'blur' }]
}

const sendCode = async () => {
  if (!registerForm.email) { ElMessage.warning('请先输入邮箱'); return }
  sendingCode.value = true
  try {
    const response = await sendVerificationCode(registerForm.email)
    if (response.success) {
      ElMessage.success(response.message)
      countdown.value = 60
      const timer = setInterval(() => { countdown.value--; if (countdown.value <= 0) clearInterval(timer) }, 1000)
    }
  } finally { sendingCode.value = false }
}

const handleFileChange = (file) => {
  registerForm.photo = file.raw
  fileList.value = [file]
  faceVerified.value = false
}

const handleFaceVerification = async () => {
  if (!registerForm.studentNumber) { ElMessage.warning('请先输入学号'); return }
  if (!registerForm.photo) { ElMessage.warning('请先上传照片'); return }
  verifying.value = true
  try {
    const formData = new FormData()
    formData.append('studentIdNumber', registerForm.studentNumber)
    formData.append('faceImage', registerForm.photo)
    const res = await request({ url: '/roster/verify-face', method: 'post', data: formData, headers: { 'Content-Type': 'multipart/form-data' }, skipAuth: true })
    if (res.success) {
      faceVerified.value = true
      similarity.value = res.similarity
      rosterId.value = res.roster_id
      ElMessage.success(`验证通过！`)
    } else ElMessage.error(res.message || '验证失败')
  } catch (error) { ElMessage.error('验证失败，请重试') }
  finally { verifying.value = false }
}

const handleRegister = async () => {
  if (!registerFormRef.value) return
  await registerFormRef.value.validate(async (valid) => {
    if (!valid) return
    if (registerForm.role === 'student' && (!registerForm.photo || !faceVerified.value)) {
      ElMessage.warning('请完成人脸验证'); return
    }
    loading.value = true
    try {
      const formDataToSubmit = { ...registerForm }
      if (registerForm.role === 'student') formDataToSubmit.rosterId = rosterId.value
      const result = await userStore.registerAction(formDataToSubmit)
      if (result.success) setTimeout(() => router.push('/login'), 1500)
    } finally { loading.value = false }
  })
}
</script>

<style scoped>
.register-page {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #f6f8fa;
  padding: 40px 20px;
}

.register-box {
  width: 100%;
  max-width: 440px;
}

.register-header {
  text-align: center;
  margin-bottom: 24px;
}

.register-header svg {
  margin-bottom: 16px;
}

.register-header h1 {
  font-size: 24px;
  font-weight: 300;
  color: #1f2328;
  margin: 0 0 8px;
}

.register-header p {
  color: #656d76;
  font-size: 14px;
  margin: 0;
}

.register-card {
  background: #ffffff;
  border: 1px solid #d0d7de;
  border-radius: 6px;
  padding: 20px;
}

.role-tabs {
  display: flex;
  gap: 8px;
}

.role-tab {
  flex: 1;
  padding: 12px;
  border: 1px solid #d0d7de;
  border-radius: 6px;
  background: #f6f8fa;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  color: #1f2328;
  transition: all 0.15s;
}

.role-tab:hover {
  border-color: #0969da;
}

.role-tab.active {
  border-color: #0969da;
  background: #ddf4ff;
  color: #0969da;
}

.code-row {
  display: flex;
  gap: 8px;
}

.code-row .el-input {
  flex: 1;
}

.form-row {
  display: flex;
  gap: 12px;
}

.form-row .el-form-item {
  flex: 1;
}

.form-hint {
  font-size: 12px;
  color: #656d76;
  margin: 8px 0 0;
}

.form-hint.warning {
  color: #bf8700;
}

.photo-upload :deep(.el-upload--picture-card) {
  width: 100px;
  height: 100px;
  border-radius: 6px;
  border: 1px dashed #d0d7de;
  background: #f6f8fa;
}

.upload-trigger {
  font-size: 14px;
  color: #656d76;
}

.verify-badge {
  display: inline-block;
  padding: 4px 12px;
  border-radius: 20px;
  font-size: 12px;
  margin-top: 8px;
}

.verify-badge.success {
  background: #dafbe1;
  color: #1a7f37;
}

.verify-btn {
  width: 100%;
  margin-bottom: 16px;
}

.submit-btn {
  width: 100%;
  background: #2da44e;
  border-color: #2da44e;
  font-weight: 500;
}

.submit-btn:hover {
  background: #2c974b;
}

.register-footer {
  margin-top: 16px;
  padding: 16px 20px;
  background: #ffffff;
  border: 1px solid #d0d7de;
  border-radius: 6px;
  text-align: center;
  font-size: 14px;
}

.register-footer a {
  color: #0969da;
  font-weight: 500;
}

:deep(.el-form-item__label) {
  font-weight: 600;
  color: #1f2328;
}

:deep(.el-input__wrapper) {
  background: #f6f8fa;
}
</style>
