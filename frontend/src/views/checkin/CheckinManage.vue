<template>
  <Layout pageTitle="签到管理">
    <div class="checkin-manage">
      <div class="page-actions">
        <el-button type="primary" @click="$router.push('/checkin/create')">发布签到</el-button>
        <el-button @click="$router.push('/checkin/smart')">智能点到</el-button>
      </div>

      <div class="manage-card">
        <div class="card-header">
          <h3>签到任务列表</h3>
          <el-radio-group v-model="statusFilter" size="small">
            <el-radio-button label="all">全部</el-radio-button>
            <el-radio-button label="active">进行中</el-radio-button>
            <el-radio-button label="ended">已结束</el-radio-button>
          </el-radio-group>
        </div>
        
        <el-table :data="checkinList" border>
          <el-table-column prop="title" label="签到标题" width="200" />
          <el-table-column prop="type" label="签到类型" width="120">
            <template #default="{ row }">
              <span class="type-tag" :class="row.type">{{ getTypeName(row.type) }}</span>
            </template>
          </el-table-column>
          <el-table-column prop="groupName" label="所属群组" width="150" />
          <el-table-column prop="startTime" label="开始时间" width="160" />
          <el-table-column prop="duration" label="时长" width="80">
            <template #default="{ row }">{{ row.duration }}分钟</template>
          </el-table-column>
          <el-table-column prop="checkedCount" label="已签到" width="100">
            <template #default="{ row }">
              <span class="count-text">{{ row.checkedCount }}/{{ row.totalCount }}</span>
            </template>
          </el-table-column>
          <el-table-column prop="status" label="状态" width="100">
            <template #default="{ row }">
              <span :class="['status-tag', row.status]">{{ row.status === 'active' ? '进行中' : '已结束' }}</span>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="150" fixed="right">
            <template #default="{ row }">
              <el-button size="small" text @click="viewDetail(row)">详情</el-button>
              <el-button v-if="row.status === 'active'" size="small" text type="danger" @click="endCheckin(row)">结束</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
    </div>
  </Layout>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import Layout from '@/components/Layout.vue'

const router = useRouter()
const statusFilter = ref('all')
const checkinList = ref([])

const getTypeName = (type) => {
  const types = {
    normal: '普通签到',
    location: '位置签到',
    qrcode: '扫码签到',
    question: '问题签到',
    face: '人脸签到'
  }
  return types[type] || type
}

const viewDetail = (row) => {
  router.push(`/checkin/records?id=${row.id}`)
}

const endCheckin = (row) => {
  // TODO: 结束签到API
}
</script>

<style scoped>
.checkin-manage {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.page-actions {
  display: flex;
  gap: 12px;
}

.manage-card {
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

.type-tag {
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 12px;
}

.type-tag.normal { background: #f6f8fa; color: #656d76; }
.type-tag.location { background: #ddf4ff; color: #0969da; }
.type-tag.qrcode { background: #fff8c5; color: #9a6700; }
.type-tag.question { background: #ffebe9; color: #cf222e; }
.type-tag.face { background: #dafbe1; color: #1a7f37; }

.count-text {
  font-weight: 500;
}

.status-tag {
  padding: 2px 8px;
  border-radius: 4px;
  font-size: 12px;
}

.status-tag.active { background: #dafbe1; color: #1a7f37; }
.status-tag.ended { background: #f6f8fa; color: #656d76; }

:deep(.el-table th) {
  background: #f6f8fa !important;
}
</style>
