<template>
  <Layout pageTitle="发布签到">
    <div class="checkin-create">
      <div class="create-card">
        <el-form :model="form" :rules="rules" ref="formRef" label-position="top">
          <el-form-item label="选择群组" prop="groupId">
            <el-select v-model="form.groupId" placeholder="请选择群组" style="width: 100%">
              <el-option v-for="g in groups" :key="g.id" :label="g.name" :value="g.id" />
            </el-select>
          </el-form-item>

          <el-form-item label="签到标题" prop="title">
            <el-input v-model="form.title" placeholder="如：第3周课堂签到" />
          </el-form-item>

          <el-form-item label="签到方式" prop="type">
            <div class="type-options">
              <div 
                v-for="t in checkinTypes" 
                :key="t.value"
                :class="['type-option', { active: form.type === t.value }]"
                @click="form.type = t.value"
              >
                <span class="type-icon">{{ t.icon }}</span>
                <span class="type-name">{{ t.label }}</span>
                <span class="type-desc">{{ t.desc }}</span>
              </div>
            </div>
          </el-form-item>

          <!-- 位置签到设置 -->
          <template v-if="form.type === 'location'">
            <el-form-item label="签到位置">
              <el-button @click="getLocation">📍 获取当前位置</el-button>
              <div v-if="form.location" class="location-info">
                当前位置：{{ form.location.address || `${form.location.lat}, ${form.location.lng}` }}
              </div>
            </el-form-item>
            <el-form-item label="允许范围（米）">
              <el-input-number v-model="form.locationRange" :min="50" :max="1000" :step="50" />
            </el-form-item>
          </template>

          <!-- 问题签到设置 -->
          <template v-if="form.type === 'question'">
            <el-form-item label="签到问题" prop="question">
              <el-input v-model="form.question" placeholder="请输入问题" />
            </el-form-item>
            <el-form-item label="正确答案" prop="answer">
              <el-input v-model="form.answer" placeholder="请输入答案" />
            </el-form-item>
          </template>

          <!-- 人脸签到说明 -->
          <div v-if="form.type === 'face'" class="face-notice">
            <el-alert title="人脸签到说明" type="info" :closable="false">
              学生需要通过人脸识别完成签到，系统将自动与花名册中的照片进行比对验证。
            </el-alert>
          </div>

          <el-form-item label="签到时长（分钟）" prop="duration">
            <el-input-number v-model="form.duration" :min="1" :max="120" />
          </el-form-item>

          <el-form-item label="签到说明">
            <el-input v-model="form.description" type="textarea" :rows="2" placeholder="选填，签到备注信息" />
          </el-form-item>

          <div class="form-actions">
            <el-button @click="$router.back()">取消</el-button>
            <el-button type="primary" @click="submitCheckin">发布签到</el-button>
          </div>
        </el-form>
      </div>
    </div>
  </Layout>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { useRouter } from 'vue-router'
import { ElMessage } from 'element-plus'
import Layout from '@/components/Layout.vue'

const router = useRouter()
const formRef = ref(null)

const groups = ref([])

const checkinTypes = [
  { value: 'normal', label: '普通签到', icon: '✅', desc: '学生点击即可签到' },
  { value: 'location', label: '位置签到', icon: '📍', desc: '需在指定位置范围内' },
  { value: 'qrcode', label: '扫码签到', icon: '📱', desc: '扫描二维码签到' },
  { value: 'question', label: '问题签到', icon: '❓', desc: '回答问题正确后签到' },
  { value: 'face', label: '人脸签到', icon: '👤', desc: '人脸识别验证签到' }
]

const form = reactive({
  groupId: '',
  title: '',
  type: 'normal',
  duration: 5,
  description: '',
  location: null,
  locationRange: 100,
  question: '',
  answer: ''
})

const rules = {
  groupId: [{ required: true, message: '请选择群组', trigger: 'change' }],
  title: [{ required: true, message: '请输入签到标题', trigger: 'blur' }],
  type: [{ required: true, message: '请选择签到方式', trigger: 'change' }],
  duration: [{ required: true, message: '请设置签到时长', trigger: 'change' }],
  question: [{ required: true, message: '请输入问题', trigger: 'blur' }],
  answer: [{ required: true, message: '请输入答案', trigger: 'blur' }]
}

const getLocation = () => {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(
      (pos) => {
        form.location = {
          lat: pos.coords.latitude,
          lng: pos.coords.longitude
        }
        ElMessage.success('位置获取成功')
      },
      () => {
        ElMessage.error('无法获取位置')
      }
    )
  } else {
    ElMessage.error('浏览器不支持定位')
  }
}

const submitCheckin = async () => {
  if (!formRef.value) return
  await formRef.value.validate((valid) => {
    if (!valid) return
    // TODO: 提交签到API
    ElMessage.success('签到发布成功')
    router.push('/checkin/manage')
  })
}
</script>

<style scoped>
.checkin-create {
  max-width: 700px;
}

.create-card {
  background: #ffffff;
  border: 1px solid #d0d7de;
  border-radius: 6px;
  padding: 24px;
}

.type-options {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
  gap: 12px;
}

.type-option {
  padding: 16px;
  border: 1px solid #d0d7de;
  border-radius: 6px;
  cursor: pointer;
  text-align: center;
  transition: all 0.15s;
}

.type-option:hover {
  border-color: #0969da;
}

.type-option.active {
  border-color: #0969da;
  background: #ddf4ff;
}

.type-icon {
  display: block;
  font-size: 28px;
  margin-bottom: 8px;
}

.type-name {
  display: block;
  font-weight: 600;
  color: #1f2328;
  margin-bottom: 4px;
}

.type-desc {
  display: block;
  font-size: 12px;
  color: #656d76;
}

.location-info {
  margin-top: 8px;
  padding: 8px 12px;
  background: #f6f8fa;
  border-radius: 4px;
  font-size: 14px;
  color: #656d76;
}

.face-notice {
  margin-bottom: 16px;
}

.form-actions {
  display: flex;
  gap: 12px;
  margin-top: 24px;
}

:deep(.el-form-item__label) {
  font-weight: 600;
}

:deep(.el-input__wrapper) {
  background: #f6f8fa;
}
</style>
