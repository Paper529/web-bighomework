<template>
  <Layout pageTitle="学生管理">
    <div class="roster-page">
      <!-- 说明卡片 -->
      <div class="info-banner">
        <div class="banner-content">
          <h3>👥 学生花名册管理</h3>
          <p>上传学生基本信息和人脸照片，用于学生注册验证</p>
        </div>
      </div>

      <!-- 学生列表 -->
      <div class="roster-card">
        <div class="card-header">
          <h3>学生列表</h3>
          <el-button type="primary" size="small" @click="showAddDialog = true">
            新增学生
          </el-button>
        </div>
        <div class="card-body">
          <el-table :data="students" border v-loading="loading" class="roster-table">
            <el-table-column type="index" label="#" width="50" />
            <el-table-column prop="student_name" label="姓名" width="100" />
            <el-table-column prop="student_id_number" label="学号" width="140" />
            <el-table-column prop="gender" label="性别" width="70" />
            <el-table-column prop="class_name" label="班级" width="120" />
            <el-table-column prop="grade" label="年级" width="100" />
            <el-table-column prop="contact_phone" label="联系电话" width="130" />
            <el-table-column prop="is_registered" label="状态" width="100">
              <template #default="{ row }">
                <span :class="['status-tag', row.is_registered ? 'success' : 'default']">
                  {{ row.is_registered ? '已注册' : '未注册' }}
                </span>
              </template>
            </el-table-column>
            <el-table-column prop="created_at" label="添加时间" width="160" />
            <el-table-column label="操作" width="80" fixed="right">
              <template #default="{ row }">
                <el-button type="danger" size="small" text @click="handleDelete(row)" :disabled="row.is_registered">
                  删除
                </el-button>
              </template>
            </el-table-column>
          </el-table>

          <el-pagination
            v-model:current-page="currentPage"
            v-model:page-size="pageSize"
            :total="total"
            :page-sizes="[10, 20, 50]"
            layout="total, sizes, prev, pager, next"
            @size-change="loadStudents"
            @current-change="loadStudents"
            class="pagination"
          />
        </div>
      </div>

      <!-- 添加学生对话框 -->
      <el-dialog v-model="showAddDialog" title="添加学生信息" width="480px" @close="resetForm">
        <el-form :model="studentForm" :rules="rules" ref="formRef" label-position="top">
          <div class="form-row">
            <el-form-item label="学生姓名" prop="studentName" class="form-col">
              <el-input v-model="studentForm.studentName" placeholder="请输入姓名" />
            </el-form-item>
            <el-form-item label="学号" prop="studentIdNumber" class="form-col">
              <el-input v-model="studentForm.studentIdNumber" placeholder="请输入学号" />
            </el-form-item>
          </div>
          
          <div class="form-row">
            <el-form-item label="性别" prop="gender" class="form-col">
              <el-radio-group v-model="studentForm.gender">
                <el-radio label="男">男</el-radio>
                <el-radio label="女">女</el-radio>
              </el-radio-group>
            </el-form-item>
            <el-form-item label="年级" class="form-col">
              <el-input v-model="studentForm.grade" placeholder="如：2024级" />
            </el-form-item>
          </div>
          
          <el-form-item label="班级">
            <el-input v-model="studentForm.className" placeholder="如：计算机1班" />
          </el-form-item>
          
          <el-form-item label="联系电话">
            <el-input v-model="studentForm.contactPhone" placeholder="选填" />
          </el-form-item>
          
          <el-form-item label="人脸照片" required>
            <el-upload
              :auto-upload="false"
              :limit="1"
              accept="image/*"
              :on-change="handleImageChange"
              :on-remove="handleImageRemove"
              list-type="picture-card"
              :file-list="fileList"
              class="photo-upload"
            >
              <span class="upload-icon">📷</span>
            </el-upload>
            <p class="form-hint">请上传学生清晰的正面照片</p>
          </el-form-item>
        </el-form>

        <template #footer>
          <el-button @click="showAddDialog = false">取消</el-button>
          <el-button type="primary" @click="submitStudent" :loading="submitting">提交</el-button>
        </template>
      </el-dialog>
    </div>
  </Layout>
</template>

<script setup>
import { ref, onMounted, reactive } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import request from '@/utils/request'
import Layout from '@/components/Layout.vue'

const students = ref([])
const loading = ref(false)
const submitting = ref(false)
const showAddDialog = ref(false)
const currentPage = ref(1)
const pageSize = ref(20)
const total = ref(0)
const formRef = ref(null)
const fileList = ref([])
const faceImage = ref(null)

const studentForm = reactive({
  studentName: '',
  studentIdNumber: '',
  gender: '',
  className: '',
  grade: '',
  contactPhone: ''
})

const rules = {
  studentName: [{ required: true, message: '请输入姓名', trigger: 'blur' }],
  studentIdNumber: [{ required: true, message: '请输入学号', trigger: 'blur' }],
  gender: [{ required: true, message: '请选择性别', trigger: 'change' }]
}

const handleImageChange = (file) => {
  faceImage.value = file.raw
  fileList.value = [file]
}

const handleImageRemove = () => {
  faceImage.value = null
  fileList.value = []
}

const loadStudents = async () => {
  loading.value = true
  try {
    const res = await request({
      url: '/roster/my-students',
      method: 'get',
      params: { page: currentPage.value, pageSize: pageSize.value }
    })
    if (res.success) {
      students.value = res.students
      total.value = res.total
    }
  } catch (error) {
    ElMessage.error('加载失败')
  } finally {
    loading.value = false
  }
}

const submitStudent = async () => {
  if (!formRef.value) return
  if (!faceImage.value) {
    ElMessage.error('请上传人脸照片')
    return
  }
  
  await formRef.value.validate(async (valid) => {
    if (!valid) return
    submitting.value = true
    const formData = new FormData()
    Object.keys(studentForm).forEach(key => {
      if (studentForm[key]) formData.append(key, studentForm[key])
    })
    formData.append('faceImage', faceImage.value)

    try {
      const res = await request({
        url: '/roster/add-student',
        method: 'post',
        data: formData,
        headers: { 'Content-Type': 'multipart/form-data' }
      })
      if (res.success) {
        ElMessage.success('添加成功')
        showAddDialog.value = false
        loadStudents()
        resetForm()
      } else {
        ElMessage.error(res.message || '添加失败')
      }
    } catch (error) {
      ElMessage.error('添加失败')
    } finally {
      submitting.value = false
    }
  })
}

const handleDelete = async (row) => {
  try {
    await ElMessageBox.confirm(`确定删除 ${row.student_name} 吗？`, '提示', { type: 'warning' })
    const res = await request({ url: `/roster/delete-student/${row.roster_id}`, method: 'delete' })
    if (res.success) {
      ElMessage.success('删除成功')
      loadStudents()
    }
  } catch (error) {
    if (error !== 'cancel') ElMessage.error('删除失败')
  }
}

const resetForm = () => {
  if (formRef.value) formRef.value.resetFields()
  Object.assign(studentForm, { studentName: '', studentIdNumber: '', gender: '', className: '', grade: '', contactPhone: '' })
  faceImage.value = null
  fileList.value = []
}

onMounted(() => loadStudents())
</script>

<style scoped>
.roster-page {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.info-banner {
  background: #ddf4ff;
  border: 1px solid #54aeff;
  border-radius: 6px;
  padding: 16px 20px;
}

.banner-content h3 {
  font-size: 14px;
  font-weight: 600;
  color: #0969da;
  margin: 0 0 4px;
}

.banner-content p {
  font-size: 14px;
  color: #0969da;
  margin: 0;
  opacity: 0.8;
}

.roster-card {
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

.roster-table {
  border: none;
}

:deep(.el-table th.el-table__cell) {
  background: #f6f8fa;
  font-weight: 600;
}

.status-tag {
  display: inline-block;
  padding: 2px 8px;
  border-radius: 20px;
  font-size: 12px;
}

.status-tag.success {
  background: #dafbe1;
  color: #1a7f37;
}

.status-tag.default {
  background: #f6f8fa;
  color: #656d76;
}

.pagination {
  margin-top: 16px;
  display: flex;
  justify-content: flex-end;
}

.form-row {
  display: flex;
  gap: 12px;
}

.form-col {
  flex: 1;
}

.photo-upload :deep(.el-upload--picture-card) {
  width: 100px;
  height: 100px;
  border-radius: 6px;
  background: #f6f8fa;
  border: 1px dashed #d0d7de;
}

.upload-icon {
  font-size: 24px;
}

.form-hint {
  font-size: 12px;
  color: #656d76;
  margin: 8px 0 0;
}

:deep(.el-form-item__label) {
  font-weight: 600;
  color: #1f2328;
}

:deep(.el-input__wrapper) {
  background: #f6f8fa;
}

:deep(.el-dialog) {
  border-radius: 12px;
}

:deep(.el-button--primary) {
  background: #2da44e;
  border-color: #2da44e;
}

:deep(.el-button--primary:hover) {
  background: #2c974b;
}
</style>
