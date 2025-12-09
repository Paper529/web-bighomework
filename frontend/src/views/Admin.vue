<template>
  <Layout pageTitle="管理中心">
    <div class="admin-page">
      <div class="admin-card">
        <div class="card-header">
          <h3>教师审核</h3>
          <el-button size="small" @click="loadPendingTeachers" :loading="loading">
            刷新
          </el-button>
        </div>
        <div class="card-body">
          <el-table :data="pendingTeachers" v-loading="loading" border class="admin-table">
            <el-table-column prop="system_account" label="系统账号" width="130" />
            <el-table-column prop="email" label="邮箱" width="200" />
            <el-table-column prop="real_name" label="真实姓名" width="120" />
            <el-table-column prop="application_time" label="申请时间" width="180">
              <template #default="{ row }">
                {{ formatDate(row.application_time) }}
              </template>
            </el-table-column>
            <el-table-column label="操作" width="180" fixed="right">
              <template #default="{ row }">
                <el-button type="primary" size="small" @click="handleApprove(row.approval_id, true)">
                  通过
                </el-button>
                <el-button type="danger" size="small" @click="handleApprove(row.approval_id, false)">
                  拒绝
                </el-button>
              </template>
            </el-table-column>
          </el-table>
          
          <el-empty v-if="!loading && pendingTeachers.length === 0" description="暂无待审核教师" />
        </div>
      </div>
    </div>
  </Layout>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { getPendingTeachers, approveTeacher } from '@/api/auth'
import { ElMessage, ElMessageBox } from 'element-plus'
import Layout from '@/components/Layout.vue'

const loading = ref(false)
const pendingTeachers = ref([])

const loadPendingTeachers = async () => {
  loading.value = true
  try {
    const response = await getPendingTeachers()
    if (response.success) pendingTeachers.value = response.teachers || []
  } finally {
    loading.value = false
  }
}

const handleApprove = async (approvalId, approved) => {
  const action = approved ? '通过' : '拒绝'
  ElMessageBox.prompt(`请输入${action}原因（选填）`, `${action}审核`, {
    confirmButtonText: '确定',
    cancelButtonText: '取消',
    inputPlaceholder: '请输入原因'
  }).then(async ({ value }) => {
    try {
      const response = await approveTeacher(approvalId, approved, value || '')
      if (response.success) {
        ElMessage.success(response.message || '审核完成')
        loadPendingTeachers()
      }
    } catch (error) {
      console.error('审核失败:', error)
    }
  }).catch(() => {})
}

const formatDate = (dateString) => {
  if (!dateString) return ''
  return new Date(dateString).toLocaleString('zh-CN')
}

onMounted(() => loadPendingTeachers())
</script>

<style scoped>
.admin-page {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.admin-card {
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
  color: #1f2328;
  margin: 0;
}

.card-body {
  padding: 16px;
}

.admin-table {
  border: none;
}

:deep(.el-table th.el-table__cell) {
  background: #f6f8fa;
  font-weight: 600;
}

:deep(.el-button--primary) {
  background: #2da44e;
  border-color: #2da44e;
}

:deep(.el-button--primary:hover) {
  background: #2c974b;
}
</style>
