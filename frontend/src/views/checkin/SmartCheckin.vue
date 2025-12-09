<template>
  <Layout pageTitle="智能点到">
    <div class="smart-checkin">
      <div class="info-banner">
        <div class="banner-icon">🤖</div>
        <div class="banner-text">
          <h3>智能点到系统</h3>
          <p>上传课堂照片，通过人脸识别自动生成考勤结果</p>
        </div>
      </div>

      <div class="upload-card">
        <div class="card-header">
          <h3>上传课堂照片</h3>
        </div>
        <div class="card-body">
          <el-form label-position="top">
            <el-form-item label="选择群组">
              <el-select v-model="form.groupId" placeholder="请选择群组" style="width: 100%">
                <el-option v-for="g in groups" :key="g.id" :label="g.name" :value="g.id" />
              </el-select>
            </el-form-item>

            <el-form-item label="上传照片">
              <el-upload
                v-model:file-list="fileList"
                action="#"
                :auto-upload="false"
                list-type="picture-card"
                :limit="5"
                accept="image/*"
              >
                <div class="upload-trigger">
                  <span class="upload-icon">📷</span>
                  <span>上传照片</span>
                </div>
              </el-upload>
              <p class="upload-tip">支持上传多张课堂照片，系统将自动识别照片中的学生</p>
            </el-form-item>

            <el-form-item label="点到说明">
              <el-input v-model="form.description" type="textarea" :rows="2" placeholder="选填，如：第5周课堂点到" />
            </el-form-item>

            <div class="form-actions">
              <el-button type="primary" :loading="analyzing" @click="startAnalyze" :disabled="fileList.length === 0">
                {{ analyzing ? '识别中...' : '开始识别' }}
              </el-button>
            </div>
          </el-form>
        </div>
      </div>

      <!-- 识别结果 -->
      <div v-if="results.length > 0" class="results-card">
        <div class="card-header">
          <h3>识别结果</h3>
          <div class="header-actions">
            <el-button size="small" @click="confirmResults">确认结果</el-button>
            <el-button size="small" @click="exportResults">导出</el-button>
          </div>
        </div>

        <div class="results-stats">
          <div class="stat-item success">
            <span class="stat-value">{{ stats.recognized }}</span>
            <span class="stat-label">已识别</span>
          </div>
          <div class="stat-item warning">
            <span class="stat-value">{{ stats.uncertain }}</span>
            <span class="stat-label">待确认</span>
          </div>
          <div class="stat-item danger">
            <span class="stat-value">{{ stats.absent }}</span>
            <span class="stat-label">未识别</span>
          </div>
        </div>

        <el-table :data="results" border>
          <el-table-column prop="studentName" label="学生姓名" width="120" />
          <el-table-column prop="studentNumber" label="学号" width="140" />
          <el-table-column prop="confidence" label="置信度" width="120">
            <template #default="{ row }">
              <el-progress :percentage="row.confidence" :color="getConfidenceColor(row.confidence)" :stroke-width="6" />
            </template>
          </el-table-column>
          <el-table-column prop="status" label="状态" width="100">
            <template #default="{ row }">
              <span :class="['status-tag', row.status]">{{ getStatusName(row.status) }}</span>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="150">
            <template #default="{ row }">
              <el-button v-if="row.status === 'uncertain'" size="small" text type="success" @click="confirmStudent(row)">确认</el-button>
              <el-button v-if="row.status === 'absent'" size="small" text @click="manualCheckin(row)">手动签到</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
    </div>
  </Layout>
</template>

<script setup>
import { ref, reactive, computed } from 'vue'
import { ElMessage } from 'element-plus'
import Layout from '@/components/Layout.vue'

const groups = ref([])
const fileList = ref([])
const analyzing = ref(false)
const results = ref([])

const form = reactive({
  groupId: '',
  description: ''
})

const stats = computed(() => {
  return {
    recognized: results.value.filter(r => r.status === 'recognized').length,
    uncertain: results.value.filter(r => r.status === 'uncertain').length,
    absent: results.value.filter(r => r.status === 'absent').length
  }
})

const getStatusName = (status) => {
  const names = { recognized: '已识别', uncertain: '待确认', absent: '未识别' }
  return names[status] || status
}

const getConfidenceColor = (confidence) => {
  if (confidence >= 80) return '#1a7f37'
  if (confidence >= 60) return '#9a6700'
  return '#cf222e'
}

const startAnalyze = async () => {
  if (!form.groupId) {
    ElMessage.warning('请选择群组')
    return
  }
  analyzing.value = true
  // TODO: 调用人脸识别API
  setTimeout(() => {
    analyzing.value = false
    ElMessage.success('识别完成')
  }, 2000)
}

const confirmStudent = (row) => {
  row.status = 'recognized'
}

const manualCheckin = (row) => {
  row.status = 'recognized'
  ElMessage.success('手动签到成功')
}

const confirmResults = () => {
  // TODO: 提交结果API
  ElMessage.success('考勤结果已保存')
}

const exportResults = () => {
  // TODO: 导出功能
}
</script>

<style scoped>
.smart-checkin {
  display: flex;
  flex-direction: column;
  gap: 16px;
  max-width: 900px;
}

.info-banner {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 20px;
  background: #ddf4ff;
  border: 1px solid #54aeff;
  border-radius: 6px;
}

.banner-icon {
  font-size: 36px;
}

.banner-text h3 {
  font-size: 16px;
  font-weight: 600;
  color: #0969da;
  margin: 0 0 4px;
}

.banner-text p {
  font-size: 14px;
  color: #0969da;
  margin: 0;
  opacity: 0.8;
}

.upload-card, .results-card {
  background: #ffffff;
  border: 1px solid #d0d7de;
  border-radius: 6px;
  overflow: hidden;
}

.card-header {
  padding: 12px 16px;
  border-bottom: 1px solid #d0d7de;
  background: #f6f8fa;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.card-header h3 {
  font-size: 14px;
  font-weight: 600;
  margin: 0;
}

.header-actions {
  display: flex;
  gap: 8px;
}

.card-body {
  padding: 20px;
}

.upload-trigger {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  color: #656d76;
}

.upload-icon {
  font-size: 24px;
}

.upload-tip {
  font-size: 12px;
  color: #656d76;
  margin-top: 8px;
}

.form-actions {
  margin-top: 16px;
}

.results-stats {
  display: flex;
  gap: 16px;
  padding: 16px;
  border-bottom: 1px solid #d0d7de;
}

.stat-item {
  flex: 1;
  text-align: center;
  padding: 12px;
  border-radius: 6px;
}

.stat-item.success { background: #dafbe1; }
.stat-item.warning { background: #fff8c5; }
.stat-item.danger { background: #ffebe9; }

.stat-value {
  display: block;
  font-size: 20px;
  font-weight: 600;
}

.stat-item.success .stat-value { color: #1a7f37; }
.stat-item.warning .stat-value { color: #9a6700; }
.stat-item.danger .stat-value { color: #cf222e; }

.stat-label {
  font-size: 12px;
  color: #656d76;
}

.status-tag {
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 12px;
}

.status-tag.recognized { background: #dafbe1; color: #1a7f37; }
.status-tag.uncertain { background: #fff8c5; color: #9a6700; }
.status-tag.absent { background: #ffebe9; color: #cf222e; }

:deep(.el-form-item__label) {
  font-weight: 600;
}

:deep(.el-upload--picture-card) {
  width: 120px;
  height: 120px;
  border-radius: 6px;
  background: #f6f8fa;
}

:deep(.el-table) {
  margin: 16px;
  width: calc(100% - 32px);
}

:deep(.el-table th) {
  background: #f6f8fa !important;
}
</style>
