<template>
  <Layout pageTitle="通知公告">
    <div class="notifications-page">
      <div class="notifications-card">
        <div class="card-header">
          <div class="header-tabs">
            <button :class="['tab', { active: activeTab === 'all' }]" @click="activeTab = 'all'">全部</button>
            <button :class="['tab', { active: activeTab === 'unread' }]" @click="activeTab = 'unread'">未读</button>
            <button :class="['tab', { active: activeTab === 'system' }]" @click="activeTab = 'system'">系统</button>
          </div>
          <el-button size="small" @click="markAllRead">全部已读</el-button>
        </div>
        
        <div class="notifications-list">
          <div v-for="item in notifications" :key="item.id" :class="['notification-item', { unread: !item.isRead }]">
            <div class="notification-icon" :class="item.type">
              {{ getIcon(item.type) }}
            </div>
            <div class="notification-content">
              <div class="notification-title">{{ item.title }}</div>
              <div class="notification-desc">{{ item.content }}</div>
              <div class="notification-time">{{ item.time }}</div>
            </div>
            <el-button v-if="!item.isRead" size="small" text @click="markRead(item)">标记已读</el-button>
          </div>
          
          <el-empty v-if="notifications.length === 0" description="暂无通知" />
        </div>
      </div>
    </div>
  </Layout>
</template>

<script setup>
import { ref } from 'vue'
import Layout from '@/components/Layout.vue'

const activeTab = ref('all')
const notifications = ref([])

const getIcon = (type) => {
  const icons = {
    system: '🔔',
    homework: '📝',
    checkin: '✅',
    message: '💬',
    approval: '👤'
  }
  return icons[type] || '📢'
}

const markRead = (item) => {
  item.isRead = true
  // TODO: API
}

const markAllRead = () => {
  notifications.value.forEach(n => n.isRead = true)
  // TODO: API
}
</script>

<style scoped>
.notifications-page {
  max-width: 800px;
}

.notifications-card {
  background: #ffffff;
  border: 1px solid #d0d7de;
  border-radius: 6px;
  overflow: hidden;
}

.card-header {
  padding: 12px 16px;
  border-bottom: 1px solid #d0d7de;
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #f6f8fa;
}

.header-tabs {
  display: flex;
  gap: 4px;
}

.tab {
  padding: 6px 12px;
  border: none;
  background: transparent;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  color: #656d76;
}

.tab:hover {
  background: #ffffff;
}

.tab.active {
  background: #ffffff;
  color: #0969da;
  font-weight: 500;
}

.notifications-list {
  max-height: calc(100vh - 280px);
  overflow-y: auto;
}

.notification-item {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  padding: 16px;
  border-bottom: 1px solid #f0f0f0;
}

.notification-item.unread {
  background: #f6f8fa;
}

.notification-icon {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 18px;
  background: #f6f8fa;
}

.notification-icon.system {
  background: #ddf4ff;
}

.notification-icon.homework {
  background: #fff8c5;
}

.notification-icon.checkin {
  background: #dafbe1;
}

.notification-content {
  flex: 1;
}

.notification-title {
  font-weight: 600;
  color: #1f2328;
  margin-bottom: 4px;
}

.notification-desc {
  font-size: 14px;
  color: #656d76;
  margin-bottom: 4px;
}

.notification-time {
  font-size: 12px;
  color: #8c959f;
}
</style>
