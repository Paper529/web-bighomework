<template>
  <Layout pageTitle="首页">
    <div class="dashboard">
      <!-- 欢迎区域 -->
      <div class="welcome-section">
        <h2>👋 欢迎回来，{{ userStore.userInfo?.realName || '用户' }}</h2>
        <p>今天是个学习的好日子</p>
      </div>
      
      <!-- 快捷功能 -->
      <div class="info-card">
        <div class="card-header">
          <h3>快捷功能</h3>
        </div>
        <div class="card-body">
          <div class="action-grid">
            <div class="action-item" @click="router.push('/messages/private')">
              <div class="action-icon">💬</div>
              <div class="action-text">
                <h4>私信</h4>
                <p>与他人私聊</p>
              </div>
            </div>
            
            <div class="action-item" @click="router.push('/messages/group')">
              <div class="action-icon">👥</div>
              <div class="action-text">
                <h4>群聊</h4>
                <p>班级群组交流</p>
              </div>
            </div>
            
            <div class="action-item" @click="router.push('/chatbot')">
              <div class="action-icon">🤖</div>
              <div class="action-text">
                <h4>AI助教</h4>
                <p>智能问答助手</p>
              </div>
            </div>
            
            <!-- 学生签到入口 -->
            <div v-if="userStore.hasRole('student')" class="action-item" @click="router.push('/checkin/student')">
              <div class="action-icon">✅</div>
              <div class="action-text">
                <h4>签到</h4>
                <p>课堂签到</p>
              </div>
            </div>
            
            <!-- 教师功能 -->
            <div v-if="userStore.hasRole('teacher')" class="action-item" @click="router.push('/checkin/manage')">
              <div class="action-icon">📋</div>
              <div class="action-text">
                <h4>签到管理</h4>
                <p>发布和管理签到</p>
              </div>
            </div>
            
            <div v-if="userStore.hasRole('teacher')" class="action-item" @click="router.push('/checkin/smart')">
              <div class="action-icon">🤖</div>
              <div class="action-text">
                <h4>智能点到</h4>
                <p>人脸识别考勤</p>
              </div>
            </div>
            
            <div v-if="userStore.hasRole('teacher')" class="action-item" @click="router.push('/student-roster')">
              <div class="action-icon">📚</div>
              <div class="action-text">
                <h4>学生管理</h4>
                <p>花名册管理</p>
              </div>
            </div>
            
            <!-- 管理员功能 -->
            <div v-if="userStore.hasRole('admin')" class="action-item" @click="router.push('/admin')">
              <div class="action-icon">🛡️</div>
              <div class="action-text">
                <h4>管理中心</h4>
                <p>教师审核</p>
              </div>
            </div>
            
            <div class="action-item" @click="router.push('/profile')">
              <div class="action-icon">⚙️</div>
              <div class="action-text">
                <h4>个人设置</h4>
                <p>修改密码</p>
              </div>
            </div>
          </div>
        </div>
      </div>
      
      <!-- 信息卡片 -->
      <div class="card-grid">
        <div class="info-card">
          <div class="card-header">
            <h3>账户信息</h3>
          </div>
          <div class="card-body">
            <div class="info-row">
              <span class="info-label">系统账号</span>
              <span class="info-value">{{ userStore.userInfo?.systemAccount }}</span>
            </div>
            <div class="info-row">
              <span class="info-label">邮箱</span>
              <span class="info-value">{{ userStore.userInfo?.email }}</span>
            </div>
            <div class="info-row">
              <span class="info-label">真实姓名</span>
              <span class="info-value">{{ userStore.userInfo?.realName || '未填写' }}</span>
            </div>
            <div class="info-row">
              <span class="info-label">角色</span>
              <span :class="['role-label', userStore.userRole]">{{ roleText }}</span>
            </div>
            <div class="info-row" v-if="userStore.userRole === 'teacher'">
              <span class="info-label">审核状态</span>
              <span :class="['status-label', userStore.userInfo?.isApproved ? 'approved' : 'pending']">
                {{ userStore.userInfo?.isApproved ? '已审核' : '待审核' }}
              </span>
            </div>
          </div>
        </div>
        
        <div class="info-card">
          <div class="card-header">
            <h3>我的权限</h3>
          </div>
          <div class="card-body">
            <div class="permission-list">
              <span v-for="permission in userStore.permissions" :key="permission" class="permission-tag">
                {{ permission }}
              </span>
              <span v-if="!userStore.permissions?.length" class="empty-text">暂无特殊权限</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </Layout>
</template>

<script setup>
import { computed } from 'vue'
import { useRouter } from 'vue-router'
import { useUserStore } from '@/stores/user'
import Layout from '@/components/Layout.vue'

const router = useRouter()
const userStore = useUserStore()

const roleText = computed(() => {
  const roleMap = { admin: '管理员', teacher: '教师', student: '学生' }
  return roleMap[userStore.userRole] || '未知'
})
</script>

<style scoped>
.dashboard {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.welcome-section {
  padding: 24px;
  background: #ffffff;
  border: 1px solid #d0d7de;
  border-radius: 6px;
}

.welcome-section h2 {
  font-size: 20px;
  font-weight: 600;
  color: #1f2328;
  margin: 0 0 4px;
}

.welcome-section p {
  color: #656d76;
  margin: 0;
}

.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 16px;
}

.info-card {
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
  padding: 16px;
}

.info-row {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
  border-bottom: 1px solid #f0f0f0;
}

.info-row:last-child {
  border-bottom: none;
}

.info-label {
  color: #656d76;
  font-size: 14px;
}

.info-value {
  color: #1f2328;
  font-weight: 500;
}

.role-label {
  padding: 2px 8px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: 500;
}

.role-label.admin { background: #ffebe9; color: #cf222e; }
.role-label.teacher { background: #fff8c5; color: #9a6700; }
.role-label.student { background: #dafbe1; color: #1a7f37; }

.status-label {
  padding: 2px 8px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: 500;
}

.status-label.approved { background: #dafbe1; color: #1a7f37; }
.status-label.pending { background: #fff8c5; color: #9a6700; }

.permission-list {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.permission-tag {
  padding: 4px 10px;
  background: #f6f8fa;
  border: 1px solid #d0d7de;
  border-radius: 20px;
  font-size: 12px;
  color: #1f2328;
}

.empty-text {
  color: #656d76;
  font-size: 14px;
}

.action-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(160px, 1fr));
  gap: 12px;
}

.action-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 16px;
  background: #f6f8fa;
  border: 1px solid #d0d7de;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.15s;
}

.action-item:hover {
  border-color: #0969da;
  background: #ffffff;
}

.action-icon {
  font-size: 24px;
}

.action-text h4 {
  font-size: 14px;
  font-weight: 600;
  color: #1f2328;
  margin: 0 0 2px;
}

.action-text p {
  font-size: 12px;
  color: #656d76;
  margin: 0;
}
</style>
