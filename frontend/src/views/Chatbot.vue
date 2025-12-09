<template>
  <Layout pageTitle="AI助教">
    <div class="chatbot-page">
      <!-- 左侧会话列表 -->
      <div class="sidebar">
        <div class="sidebar-header">
          <span>对话列表</span>
          <el-button type="primary" size="small" @click="handleCreateSession">
            新建
          </el-button>
        </div>
        <div class="session-list">
          <div 
            v-for="session in sessions" 
            :key="session.session_id"
            :class="['session-item', { active: currentSessionId === session.session_id }]"
            @click="handleSelectSession(session.session_id)"
          >
            <div class="session-info">
              <span class="session-name">{{ session.session_name }}</span>
              <span class="session-time">{{ formatTime(session.updated_at) }}</span>
            </div>
            <button class="delete-btn" @click.stop="handleDeleteSession(session.session_id)">×</button>
          </div>
          <div v-if="sessions.length === 0" class="empty-sessions">
            暂无对话
          </div>
        </div>
      </div>

      <!-- 右侧聊天区域 -->
      <div class="chat-area">
        <div class="chat-header">
          <span>🤖 AI智能助教</span>
          <el-switch v-model="useKnowledgeBase" active-text="知识库" inactive-text="纯聊天" size="small" />
        </div>

        <div class="messages" ref="messagesContainer">
          <div v-if="messages.length === 0" class="empty-messages">
            <div class="empty-icon">💬</div>
            <h3>开始新的对话</h3>
            <p>你可以问我关于学习资料的问题</p>
          </div>

          <div v-for="msg in messages" :key="msg.message_id" :class="['message', msg.role]">
            <div class="avatar">{{ msg.role === 'user' ? (userStore.userInfo?.realName?.[0] || '我') : '🤖' }}</div>
            <div class="content">
              <div class="text" v-html="formatMessage(msg.content)"></div>
              <div class="time">{{ formatTime(msg.created_at) }}</div>
            </div>
          </div>

          <div v-if="isLoading" class="message assistant">
            <div class="avatar">🤖</div>
            <div class="content">
              <div class="loading-dots">
                <span></span><span></span><span></span>
              </div>
            </div>
          </div>
        </div>

        <div class="input-area">
          <el-input
            v-model="inputMessage"
            type="textarea"
            :rows="2"
            placeholder="输入消息... (Ctrl+Enter 发送)"
            @keydown.enter.ctrl="handleSendMessage"
            resize="none"
          />
          <el-button 
            type="primary" 
            @click="handleSendMessage"
            :loading="isLoading"
            :disabled="!inputMessage.trim() || !currentSessionId"
          >
            发送
          </el-button>
        </div>
      </div>
    </div>
  </Layout>
</template>

<script setup>
import { ref, onMounted, nextTick } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
import { useUserStore } from '@/stores/user'
import { getSessions, createSession, deleteSession, getMessages, sendMessage } from '@/api/chatbot'
import Layout from '@/components/Layout.vue'

const userStore = useUserStore()
const sessions = ref([])
const currentSessionId = ref(null)
const messages = ref([])
const inputMessage = ref('')
const isLoading = ref(false)
const useKnowledgeBase = ref(true)
const messagesContainer = ref(null)

const loadSessions = async () => {
  try {
    const res = await getSessions()
    if (res.success) {
      sessions.value = res.sessions
      if (!currentSessionId.value && sessions.value.length > 0) {
        currentSessionId.value = sessions.value[0].session_id
        await loadMessages()
      }
    }
  } catch (error) { console.error('加载会话失败:', error) }
}

const handleCreateSession = async () => {
  try {
    const res = await createSession('新对话')
    if (res.success) {
      ElMessage.success('创建成功')
      await loadSessions()
      currentSessionId.value = res.sessionId
      messages.value = []
    }
  } catch (error) { ElMessage.error('创建失败') }
}

const handleSelectSession = async (sessionId) => {
  currentSessionId.value = sessionId
  await loadMessages()
}

const handleDeleteSession = async (sessionId) => {
  try {
    await ElMessageBox.confirm('确定删除这个对话吗？', '提示', { type: 'warning' })
    const res = await deleteSession(sessionId)
    if (res.success) {
      ElMessage.success('删除成功')
      if (currentSessionId.value === sessionId) {
        currentSessionId.value = null
        messages.value = []
      }
      await loadSessions()
    }
  } catch (error) { if (error !== 'cancel') ElMessage.error('删除失败') }
}

const loadMessages = async () => {
  if (!currentSessionId.value) return
  try {
    const res = await getMessages(currentSessionId.value)
    if (res.success) {
      messages.value = res.messages
      await scrollToBottom()
    }
  } catch (error) { console.error('加载消息失败:', error) }
}

const handleSendMessage = async () => {
  if (!inputMessage.value.trim() || !currentSessionId.value || isLoading.value) return
  const userMessage = inputMessage.value.trim()
  inputMessage.value = ''
  messages.value.push({ message_id: Date.now(), role: 'user', content: userMessage, created_at: new Date().toISOString() })
  await scrollToBottom()
  isLoading.value = true
  try {
    const res = await sendMessage(currentSessionId.value, userMessage, useKnowledgeBase.value)
    if (res.success) {
      messages.value.push({ message_id: Date.now() + 1, role: 'assistant', content: res.message, created_at: new Date().toISOString() })
      await scrollToBottom()
      await loadSessions()
    } else ElMessage.error(res.message || 'AI回复失败')
  } catch (error) { ElMessage.error('发送失败') }
  finally { isLoading.value = false }
}

const scrollToBottom = async () => {
  await nextTick()
  if (messagesContainer.value) messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight
}

const formatMessage = (text) => text.replace(/\n/g, '<br>')

const formatTime = (timestamp) => {
  if (!timestamp) return ''
  let dateStr = timestamp
  if (typeof timestamp === 'string' && !timestamp.includes('T')) dateStr = timestamp.replace(' ', 'T')
  const date = new Date(dateStr)
  const diff = Date.now() - date
  if (diff < 60000) return '刚刚'
  if (diff < 3600000) return `${Math.floor(diff / 60000)}分钟前`
  if (diff < 86400000) return `${Math.floor(diff / 3600000)}小时前`
  return date.toLocaleString('zh-CN', { month: '2-digit', day: '2-digit', hour: '2-digit', minute: '2-digit' })
}

onMounted(() => loadSessions())
</script>

<style scoped>
.chatbot-page {
  display: flex;
  height: calc(100vh - 180px);
  background: #ffffff;
  border: 1px solid #d0d7de;
  border-radius: 6px;
  overflow: hidden;
}

.sidebar {
  width: 260px;
  border-right: 1px solid #d0d7de;
  display: flex;
  flex-direction: column;
  background: #f6f8fa;
}

.sidebar-header {
  padding: 12px 16px;
  border-bottom: 1px solid #d0d7de;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-weight: 600;
  font-size: 14px;
  color: #1f2328;
}

.session-list {
  flex: 1;
  overflow-y: auto;
  padding: 8px;
}

.session-item {
  padding: 10px 12px;
  border-radius: 6px;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 4px;
  transition: background 0.15s;
}

.session-item:hover {
  background: #ffffff;
}

.session-item.active {
  background: #ffffff;
  border: 1px solid #d0d7de;
}

.session-info {
  flex: 1;
  overflow: hidden;
}

.session-name {
  display: block;
  font-size: 14px;
  color: #1f2328;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.session-time {
  font-size: 12px;
  color: #656d76;
}

.delete-btn {
  opacity: 0;
  background: none;
  border: none;
  font-size: 18px;
  color: #656d76;
  cursor: pointer;
  padding: 0 4px;
}

.session-item:hover .delete-btn {
  opacity: 1;
}

.delete-btn:hover {
  color: #cf222e;
}

.empty-sessions {
  text-align: center;
  padding: 20px;
  color: #656d76;
  font-size: 14px;
}

.chat-area {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.chat-header {
  padding: 12px 16px;
  border-bottom: 1px solid #d0d7de;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-weight: 600;
  font-size: 14px;
  color: #1f2328;
}

.messages {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
}

.empty-messages {
  height: 100%;
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

.empty-messages h3 {
  font-size: 16px;
  color: #1f2328;
  margin: 0 0 8px;
}

.empty-messages p {
  font-size: 14px;
  margin: 0;
}

.message {
  display: flex;
  margin-bottom: 16px;
}

.message.user {
  flex-direction: row-reverse;
}

.avatar {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  margin: 0 10px;
  flex-shrink: 0;
  background: #f6f8fa;
  border: 1px solid #d0d7de;
}

.message.user .avatar {
  background: #ddf4ff;
  border-color: #54aeff;
  color: #0969da;
}

.content {
  max-width: 70%;
}

.text {
  padding: 10px 14px;
  border-radius: 6px;
  line-height: 1.5;
  font-size: 14px;
  background: #f6f8fa;
  border: 1px solid #d0d7de;
  color: #1f2328;
}

.message.user .text {
  background: #ddf4ff;
  border-color: #54aeff;
}

.time {
  font-size: 11px;
  color: #656d76;
  margin-top: 4px;
  text-align: right;
}

.message.user .time {
  text-align: left;
}

.loading-dots {
  display: flex;
  gap: 4px;
  padding: 10px 14px;
  background: #f6f8fa;
  border: 1px solid #d0d7de;
  border-radius: 6px;
}

.loading-dots span {
  width: 8px;
  height: 8px;
  background: #656d76;
  border-radius: 50%;
  animation: bounce 1.4s infinite ease-in-out both;
}

.loading-dots span:nth-child(1) { animation-delay: -0.32s; }
.loading-dots span:nth-child(2) { animation-delay: -0.16s; }

@keyframes bounce {
  0%, 80%, 100% { transform: scale(0); }
  40% { transform: scale(1); }
}

.input-area {
  padding: 16px;
  border-top: 1px solid #d0d7de;
  display: flex;
  gap: 12px;
  align-items: flex-end;
}

.input-area :deep(.el-textarea__inner) {
  background: #f6f8fa;
  border-radius: 6px;
}

.input-area .el-button {
  background: #2da44e;
  border-color: #2da44e;
}

.input-area .el-button:hover {
  background: #2c974b;
}
</style>
