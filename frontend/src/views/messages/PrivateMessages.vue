<template>
  <Layout pageTitle="私信">
    <div class="messages-page">
      <div class="messages-container">
        <!-- 联系人列表 -->
        <div class="contacts-panel">
          <div class="panel-header">
            <span>联系人</span>
            <el-button type="primary" size="small" @click="showNewChat = true">新建</el-button>
          </div>
          <div class="contacts-list">
            <div 
              v-for="contact in contacts" 
              :key="contact.id"
              :class="['contact-item', { active: currentContact?.id === contact.id }]"
              @click="selectContact(contact)"
            >
              <div class="contact-avatar">{{ contact.name?.[0] || 'U' }}</div>
              <div class="contact-info">
                <span class="contact-name">{{ contact.name }}</span>
                <span class="contact-preview">{{ contact.lastMessage }}</span>
              </div>
              <el-badge v-if="contact.unread" :value="contact.unread" class="unread-badge" />
            </div>
            <div v-if="contacts.length === 0" class="empty-contacts">
              暂无联系人
            </div>
          </div>
        </div>

        <!-- 聊天区域 -->
        <div class="chat-panel">
          <template v-if="currentContact">
            <div class="chat-header">
              <span class="chat-title">{{ currentContact.name }}</span>
              <span class="chat-role">{{ currentContact.role }}</span>
            </div>
            <div class="chat-messages" ref="messagesRef">
              <div v-for="msg in messages" :key="msg.id" :class="['message', msg.isMine ? 'mine' : 'other']">
                <div class="message-content">{{ msg.content }}</div>
                <div class="message-time">{{ msg.time }}</div>
              </div>
            </div>
            <div class="chat-input">
              <el-input v-model="inputMessage" placeholder="输入消息..." @keyup.enter="sendMessage" />
              <el-button type="primary" @click="sendMessage">发送</el-button>
            </div>
          </template>
          <div v-else class="empty-chat">
            <div class="empty-icon">💬</div>
            <p>选择一个联系人开始聊天</p>
          </div>
        </div>
      </div>
    </div>

    <!-- 新建聊天对话框 -->
    <el-dialog v-model="showNewChat" title="新建私信" width="400px">
      <el-form label-position="top">
        <el-form-item label="搜索用户">
          <el-input v-model="searchUser" placeholder="输入用户名或账号搜索" />
        </el-form-item>
        <div class="search-results">
          <!-- 搜索结果列表 -->
        </div>
      </el-form>
    </el-dialog>
  </Layout>
</template>

<script setup>
import { ref } from 'vue'
import Layout from '@/components/Layout.vue'

const contacts = ref([])
const currentContact = ref(null)
const messages = ref([])
const inputMessage = ref('')
const showNewChat = ref(false)
const searchUser = ref('')
const messagesRef = ref(null)

const selectContact = (contact) => {
  currentContact.value = contact
  // TODO: 加载聊天记录
}

const sendMessage = () => {
  if (!inputMessage.value.trim()) return
  // TODO: 发送消息API
  inputMessage.value = ''
}
</script>

<style scoped>
.messages-page {
  height: calc(100vh - 180px);
}

.messages-container {
  display: flex;
  height: 100%;
  background: #ffffff;
  border: 1px solid #d0d7de;
  border-radius: 6px;
  overflow: hidden;
}

.contacts-panel {
  width: 280px;
  border-right: 1px solid #d0d7de;
  display: flex;
  flex-direction: column;
}

.panel-header {
  padding: 12px 16px;
  border-bottom: 1px solid #d0d7de;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-weight: 600;
  background: #f6f8fa;
}

.contacts-list {
  flex: 1;
  overflow-y: auto;
}

.contact-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  cursor: pointer;
  border-bottom: 1px solid #f0f0f0;
}

.contact-item:hover {
  background: #f6f8fa;
}

.contact-item.active {
  background: #ddf4ff;
}

.contact-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: #f6f8fa;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  color: #656d76;
}

.contact-info {
  flex: 1;
  overflow: hidden;
}

.contact-name {
  display: block;
  font-weight: 500;
  color: #1f2328;
}

.contact-preview {
  display: block;
  font-size: 12px;
  color: #656d76;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.empty-contacts {
  padding: 40px;
  text-align: center;
  color: #656d76;
}

.chat-panel {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.chat-header {
  padding: 12px 16px;
  border-bottom: 1px solid #d0d7de;
  background: #f6f8fa;
}

.chat-title {
  font-weight: 600;
  color: #1f2328;
}

.chat-role {
  margin-left: 8px;
  font-size: 12px;
  color: #656d76;
}

.chat-messages {
  flex: 1;
  overflow-y: auto;
  padding: 16px;
}

.message {
  margin-bottom: 12px;
  max-width: 70%;
}

.message.mine {
  margin-left: auto;
  text-align: right;
}

.message-content {
  display: inline-block;
  padding: 8px 12px;
  border-radius: 6px;
  background: #f6f8fa;
  color: #1f2328;
}

.message.mine .message-content {
  background: #ddf4ff;
}

.message-time {
  font-size: 11px;
  color: #656d76;
  margin-top: 4px;
}

.chat-input {
  padding: 12px 16px;
  border-top: 1px solid #d0d7de;
  display: flex;
  gap: 12px;
}

.chat-input .el-input {
  flex: 1;
}

.empty-chat {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #656d76;
}

.empty-icon {
  font-size: 48px;
  margin-bottom: 16px;
}
</style>
