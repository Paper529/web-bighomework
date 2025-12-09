<template>
  <Layout pageTitle="签到">
    <div class="student-checkin">
      <!-- 当前签到任务 -->
      <div v-if="activeCheckins.length > 0" class="active-section">
        <h3>📢 进行中的签到</h3>
        <div class="checkin-cards">
          <div v-for="item in activeCheckins" :key="item.id" class="checkin-card active">
            <div class="card-header">
              <span class="card-title">{{ item.title }}</span>
              <span class="card-type">{{ getTypeName(item.type) }}</span>
            </div>
            <div class="card-info">
              <div class="info-item">
                <span class="info-label">群组</span>
                <span class="info-value">{{ item.groupName }}</span>
              </div>
              <div class="info-item">
                <span class="info-label">剩余时间</span>
                <span class="info-value countdown">{{ item.remainingTime }}</span>
              </div>
            </div>
            <div class="card-action">
              <el-button type="primary" @click="doCheckin(item)">立即签到</el-button>
            </div>
          </div>
        </div>
      </div>

      <div v-else class="empty-active">
        <div class="empty-icon">✅</div>
        <p>暂无进行中的签到</p>
      </div>

      <!-- 签到历史 -->
      <div class="history-section">
        <div class="section-header">
          <h3>签到历史</h3>
        </div>
        <div class="history-card">
          <el-table :data="checkinHistory" border>
            <el-table-column prop="title" label="签到标题" width="200" />
            <el-table-column prop="groupName" label="群组" width="150" />
            <el-table-column prop="type" label="类型" width="100">
              <template #default="{ row }">{{ getTypeName(row.type) }}</template>
            </el-table-column>
            <el-table-column prop="checkinTime" label="签到时间" width="180" />
            <el-table-column prop="status" label="状态" width="100">
              <template #default="{ row }">
                <span :class="['status-tag', row.status]">{{ getStatusName(row.status) }}</span>
              </template>
            </el-table-column>
          </el-table>
          <el-empty v-if="checkinHistory.length === 0" description="暂无签到记录" />
        </div>
      </div>
    </div>

    <!-- 签到对话框 -->
    <el-dialog v-model="showCheckinDialog" :title="currentCheckin?.title" width="450px">
      <div class="checkin-dialog">
        <!-- 普通签到 -->
        <div v-if="currentCheckin?.type === 'normal'" class="checkin-content">
          <div class="checkin-icon">✅</div>
          <p>点击下方按钮完成签到</p>
        </div>

        <!-- 位置签到 -->
        <div v-else-if="currentCheckin?.type === 'location'" class="checkin-content">
          <div class="checkin-icon">📍</div>
          <p>需要获取您的位置信息</p>
          <el-button @click="getLocation" :loading="gettingLocation">
            {{ locationInfo ? '位置已获取' : '获取位置' }}
          </el-button>
          <div v-if="locationInfo" class="location-status success">
            ✓ 位置验证通过
          </div>
        </div>

        <!-- 扫码签到 -->
        <div v-else-if="currentCheckin?.type === 'qrcode'" class="checkin-content">
          <div class="checkin-icon">📱</div>
          <p>请扫描教师展示的二维码</p>
          <el-input v-model="qrcodeInput" placeholder="或输入签到码" />
        </div>

        <!-- 问题签到 -->
        <div v-else-if="currentCheckin?.type === 'question'" class="checkin-content">
          <div class="checkin-icon">❓</div>
          <p class="question-text">{{ currentCheckin?.question }}</p>
          <el-input v-model="answerInput" placeholder="请输入答案" />
        </div>

        <!-- 人脸签到 -->
        <div v-else-if="currentCheckin?.type === 'face'" class="checkin-content">
          <div class="checkin-icon">👤</div>
          <p>请上传您的人脸照片进行验证</p>
          <el-upload
            :auto-upload="false"
            :limit="1"
            accept="image/*"
            :on-change="handleFaceUpload"
            list-type="picture-card"
          >
            <span>📷 拍照/上传</span>
          </el-upload>
          <div v-if="faceVerified" class="location-status success">
            ✓ 人脸验证通过
          </div>
        </div>
      </div>

      <template #footer>
        <el-button @click="showCheckinDialog = false">取消</el-button>
        <el-button type="primary" @click="submitCheckin" :loading="submitting">确认签到</el-button>
      </template>
    </el-dialog>
  </Layout>
</template>

<script setup>
import { ref } from 'vue'
import { ElMessage } from 'element-plus'
import Layout from '@/components/Layout.vue'

const activeCheckins = ref([])
const checkinHistory = ref([])
const showCheckinDialog = ref(false)
const currentCheckin = ref(null)
const submitting = ref(false)

// 签到相关状态
const locationInfo = ref(null)
const gettingLocation = ref(false)
const qrcodeInput = ref('')
const answerInput = ref('')
const faceVerified = ref(false)

const getTypeName = (type) => {
  const types = { normal: '普通', location: '位置', qrcode: '扫码', question: '问题', face: '人脸' }
  return types[type] || type
}

const getStatusName = (status) => {
  const names = { checked: '已签到', late: '迟到', absent: '缺勤' }
  return names[status] || status
}

const doCheckin = (item) => {
  currentCheckin.value = item
  showCheckinDialog.value = true
  // 重置状态
  locationInfo.value = null
  qrcodeInput.value = ''
  answerInput.value = ''
  faceVerified.value = false
}

const getLocation = () => {
  gettingLocation.value = true
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(
      (pos) => {
        locationInfo.value = { lat: pos.coords.latitude, lng: pos.coords.longitude }
        gettingLocation.value = false
        ElMessage.success('位置获取成功')
      },
      () => {
        gettingLocation.value = false
        ElMessage.error('无法获取位置')
      }
    )
  }
}

const handleFaceUpload = (file) => {
  // TODO: 人脸验证API
  faceVerified.value = true
  ElMessage.success('人脸验证通过')
}

const submitCheckin = async () => {
  submitting.value = true
  // TODO: 提交签到API
  setTimeout(() => {
    submitting.value = false
    showCheckinDialog.value = false
    ElMessage.success('签到成功')
  }, 1000)
}
</script>

<style scoped>
.student-checkin {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.active-section h3, .section-header h3 {
  font-size: 16px;
  font-weight: 600;
  color: #1f2328;
  margin: 0 0 16px;
}

.checkin-cards {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 16px;
}

.checkin-card {
  background: #ffffff;
  border: 1px solid #d0d7de;
  border-radius: 6px;
  padding: 16px;
}

.checkin-card.active {
  border-color: #2da44e;
  background: #dafbe1;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 12px;
}

.card-title {
  font-weight: 600;
  color: #1f2328;
}

.card-type {
  padding: 2px 8px;
  background: #ffffff;
  border-radius: 4px;
  font-size: 12px;
  color: #656d76;
}

.card-info {
  display: flex;
  gap: 24px;
  margin-bottom: 16px;
}

.info-item {
  display: flex;
  flex-direction: column;
  gap: 4px;
}

.info-label {
  font-size: 12px;
  color: #656d76;
}

.info-value {
  font-weight: 500;
  color: #1f2328;
}

.info-value.countdown {
  color: #cf222e;
}

.empty-active {
  text-align: center;
  padding: 60px 20px;
  background: #ffffff;
  border: 1px solid #d0d7de;
  border-radius: 6px;
}

.empty-icon {
  font-size: 48px;
  margin-bottom: 16px;
}

.empty-active p {
  color: #656d76;
}

.history-card {
  background: #ffffff;
  border: 1px solid #d0d7de;
  border-radius: 6px;
  padding: 16px;
}

.status-tag {
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 12px;
}

.status-tag.checked { background: #dafbe1; color: #1a7f37; }
.status-tag.late { background: #fff8c5; color: #9a6700; }
.status-tag.absent { background: #ffebe9; color: #cf222e; }

.checkin-dialog {
  text-align: center;
  padding: 20px 0;
}

.checkin-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 16px;
}

.checkin-icon {
  font-size: 48px;
}

.question-text {
  font-size: 16px;
  font-weight: 500;
  color: #1f2328;
  padding: 16px;
  background: #f6f8fa;
  border-radius: 6px;
  width: 100%;
}

.location-status {
  padding: 8px 16px;
  border-radius: 6px;
  font-size: 14px;
}

.location-status.success {
  background: #dafbe1;
  color: #1a7f37;
}

:deep(.el-table th) {
  background: #f6f8fa !important;
}

:deep(.el-upload--picture-card) {
  width: 120px;
  height: 120px;
}
</style>
