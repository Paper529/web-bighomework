<template>
  <Layout pageTitle="签到记录">
    <div class="records-page">
      <div class="records-card">
        <div class="card-header">
          <h3>签到详情</h3>
          <el-button size="small" @click="exportRecords">导出记录</el-button>
        </div>

        <!-- 签到信息 -->
        <div class="checkin-info" v-if="checkinInfo">
          <div class="info-row">
            <span class="info-label">签到标题</span>
            <span class="info-value">{{ checkinInfo.title }}</span>
          </div>
          <div class="info-row">
            <span class="info-label">签到类型</span>
            <span class="info-value">{{ getTypeName(checkinInfo.type) }}</span>
          </div>
          <div class="info-row">
            <span class="info-label">开始时间</span>
            <span class="info-value">{{ checkinInfo.startTime }}</span>
          </div>
          <div class="info-row">
            <span class="info-label">签到率</span>
            <span class="info-value">{{ checkinInfo.checkedCount }}/{{ checkinInfo.totalCount }} ({{ checkinRate }}%)</span>
          </div>
        </div>

        <!-- 签到统计 -->
        <div class="stats-row">
          <div class="stat-item success">
            <span class="stat-value">{{ stats.checked }}</span>
            <span class="stat-label">已签到</span>
          </div>
          <div class="stat-item warning">
            <span class="stat-value">{{ stats.late }}</span>
            <span class="stat-label">迟到</span>
          </div>
          <div class="stat-item danger">
            <span class="stat-value">{{ stats.absent }}</span>
            <span class="stat-label">未签到</span>
          </div>
        </div>

        <!-- 记录列表 -->
        <el-table :data="records" border>
          <el-table-column prop="studentName" label="学生姓名" width="120" />
          <el-table-column prop="studentNumber" label="学号" width="140" />
          <el-table-column prop="checkinTime" label="签到时间" width="180" />
          <el-table-column prop="status" label="状态" width="100">
            <template #default="{ row }">
              <span :class="['status-tag', row.status]">{{ getStatusName(row.status) }}</span>
            </template>
          </el-table-column>
          <el-table-column prop="remark" label="备注" />
        </el-table>
      </div>
    </div>
  </Layout>
</template>

<script setup>
import { ref, computed } from 'vue'
import Layout from '@/components/Layout.vue'

const checkinInfo = ref(null)
const records = ref([])
const stats = ref({ checked: 0, late: 0, absent: 0 })

const checkinRate = computed(() => {
  if (!checkinInfo.value) return 0
  return Math.round((checkinInfo.value.checkedCount / checkinInfo.value.totalCount) * 100)
})

const getTypeName = (type) => {
  const types = { normal: '普通签到', location: '位置签到', qrcode: '扫码签到', question: '问题签到', face: '人脸签到' }
  return types[type] || type
}

const getStatusName = (status) => {
  const names = { checked: '已签到', late: '迟到', absent: '未签到' }
  return names[status] || status
}

const exportRecords = () => {
  // TODO: 导出功能
}
</script>

<style scoped>
.records-page {
  max-width: 900px;
}

.records-card {
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

.checkin-info {
  padding: 16px;
  border-bottom: 1px solid #d0d7de;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 12px;
}

.info-row {
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

.stats-row {
  display: flex;
  gap: 16px;
  padding: 16px;
  border-bottom: 1px solid #d0d7de;
}

.stat-item {
  flex: 1;
  text-align: center;
  padding: 16px;
  border-radius: 6px;
}

.stat-item.success { background: #dafbe1; }
.stat-item.warning { background: #fff8c5; }
.stat-item.danger { background: #ffebe9; }

.stat-value {
  display: block;
  font-size: 24px;
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

.status-tag.checked { background: #dafbe1; color: #1a7f37; }
.status-tag.late { background: #fff8c5; color: #9a6700; }
.status-tag.absent { background: #ffebe9; color: #cf222e; }

:deep(.el-table) {
  margin: 16px;
  width: calc(100% - 32px);
}

:deep(.el-table th) {
  background: #f6f8fa !important;
}
</style>
