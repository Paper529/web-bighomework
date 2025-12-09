<template>
  <Layout pageTitle="群聊">
    <div class="group-page">
      <div class="group-container">
        <!-- 群组列表 -->
        <div class="groups-panel">
          <div class="panel-header">
            <span>我的群组</span>
            <el-button v-if="userStore.hasRole('teacher') || userStore.hasRole('admin')" type="primary" size="small" @click="showCreateGroup = true">
              创建
            </el-button>
          </div>
          <div class="groups-list">
            <div 
              v-for="group in groups" 
              :key="group.id"
              :class="['group-item', { active: currentGroup?.id === group.id }]"
              @click="selectGroup(group)"
            >
              <div class="group-avatar">{{ group.name?.[0] || 'G' }}</div>
              <div class="group-info">
                <span class="group-name">{{ group.name }}</span>
                <span class="group-members">{{ group.memberCount }}人</span>
              </div>
            </div>
            <div v-if="groups.length === 0" class="empty-groups">
              暂无群组
            </div>
          </div>
        </div>

        <!-- 聊天区域 -->
        <div class="chat-panel">
          <template v-if="currentGroup">
            <div class="chat-header">
              <div class="header-left">
                <span class="chat-title">{{ currentGroup.name }}</span>
                <span class="chat-members">{{ currentGroup.memberCount }}人</span>
              </div>
              <div class="header-actions">
                <el-button size="small" @click="showGroupInfo = true">群信息</el-button>
              </div>
            </div>
            
            <div class="chat-messages" ref="messagesRef">
              <!-- 系统消息/公告 -->
              <div v-for="msg in messages" :key="msg.id" :class="['message', msg.type, msg.isMine ? 'mine' : 'other']">
                <!-- 系统消息 -->
                <div v-if="msg.type === 'system'" class="system-message">
                  {{ msg.content }}
                </div>
                
                <!-- 作业发布 -->
                <div v-else-if="msg.type === 'homework'" class="special-message homework">
                  <div class="special-header">📝 作业发布</div>
                  <div class="special-title">{{ msg.title }}</div>
                  <div class="special-content">{{ msg.content }}</div>
                  <div class="special-footer">
                    <span>截止时间：{{ msg.deadline }}</span>
                    <el-button size="small" type="primary">查看详情</el-button>
                  </div>
                </div>
                
                <!-- 签到通知 -->
                <div v-else-if="msg.type === 'checkin'" class="special-message checkin">
                  <div class="special-header">✅ 签到</div>
                  <div class="special-title">{{ msg.title }}</div>
                  <div class="special-content">{{ msg.content }}</div>
                  <div class="special-footer">
                    <span>{{ msg.checkinType }}</span>
                    <el-button size="small" type="success">立即签到</el-button>
                  </div>
                </div>
                
                <!-- 提问 -->
                <div v-else-if="msg.type === 'question'" class="special-message question">
                  <div class="special-header">❓ 课堂提问</div>
                  <div class="special-content">{{ msg.content }}</div>
                  <div class="special-footer">
                    <el-button size="small" type="warning">回答问题</el-button>
                  </div>
                </div>
                
                <!-- 普通消息 -->
                <template v-else>
                  <div class="message-avatar">{{ msg.senderName?.[0] || 'U' }}</div>
                  <div class="message-body">
                    <div class="message-sender">{{ msg.senderName }}</div>
                    <div class="message-content">{{ msg.content }}</div>
                    <div class="message-time">{{ msg.time }}</div>
                  </div>
                </template>
              </div>
            </div>
            
            <div class="chat-input">
              <!-- 教师功能按钮 -->
              <div v-if="userStore.hasRole('teacher')" class="teacher-actions">
                <el-button size="small" @click="showHomework = true">📝 发布作业</el-button>
                <el-button size="small" @click="showCheckin = true">✅ 发起签到</el-button>
                <el-button size="small" @click="showQuestion = true">❓ 课堂提问</el-button>
                <el-button size="small" @click="showNotice = true">📢 发布公告</el-button>
              </div>
              <div class="input-row">
                <el-input v-model="inputMessage" placeholder="输入消息..." @keyup.enter="sendMessage" />
                <el-button type="primary" @click="sendMessage">发送</el-button>
              </div>
            </div>
          </template>
          <div v-else class="empty-chat">
            <div class="empty-icon">👥</div>
            <p>选择一个群组开始聊天</p>
          </div>
        </div>
      </div>
    </div>

    <!-- 创建群组对话框 -->
    <el-dialog v-model="showCreateGroup" title="创建群组" width="450px">
      <el-form label-position="top">
        <el-form-item label="群组名称">
          <el-input v-model="newGroup.name" placeholder="如：2024级计算机1班" />
        </el-form-item>
        <el-form-item label="群组描述">
          <el-input v-model="newGroup.description" type="textarea" placeholder="群组简介" />
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="showCreateGroup = false">取消</el-button>
        <el-button type="primary">创建</el-button>
      </template>
    </el-dialog>

    <!-- 发布作业对话框 -->
    <el-dialog v-model="showHomework" title="发布作业" width="500px">
      <el-form label-position="top">
        <el-form-item label="作业标题">
          <el-input v-model="homework.title" placeholder="请输入作业标题" />
        </el-form-item>
        <el-form-item label="作业内容">
          <el-input v-model="homework.content" type="textarea" :rows="4" placeholder="请输入作业要求" />
        </el-form-item>
        <el-form-item label="截止时间">
          <el-date-picker v-model="homework.deadline" type="datetime" placeholder="选择截止时间" style="width: 100%" />
        </el-form-item>
        <el-form-item label="附件">
          <el-upload action="#" :auto-upload="false">
            <el-button size="small">上传附件</el-button>
          </el-upload>
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="showHomework = false">取消</el-button>
        <el-button type="primary">发布</el-button>
      </template>
    </el-dialog>

    <!-- 发起签到对话框 -->
    <el-dialog v-model="showCheckin" title="发起签到" width="450px">
      <el-form label-position="top">
        <el-form-item label="签到方式">
          <el-radio-group v-model="checkin.type">
            <el-radio label="normal">普通签到</el-radio>
            <el-radio label="location">位置签到</el-radio>
            <el-radio label="qrcode">扫码签到</el-radio>
            <el-radio label="question">问题签到</el-radio>
            <el-radio label="face">人脸签到</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item v-if="checkin.type === 'question'" label="签到问题">
          <el-input v-model="checkin.question" placeholder="请输入问题" />
        </el-form-item>
        <el-form-item v-if="checkin.type === 'question'" label="正确答案">
          <el-input v-model="checkin.answer" placeholder="请输入答案" />
        </el-form-item>
        <el-form-item label="签到时长（分钟）">
          <el-input-number v-model="checkin.duration" :min="1" :max="60" />
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="showCheckin = false">取消</el-button>
        <el-button type="primary">发起签到</el-button>
      </template>
    </el-dialog>

    <!-- 课堂提问对话框 -->
    <el-dialog v-model="showQuestion" title="课堂提问" width="450px">
      <el-form label-position="top">
        <el-form-item label="问题内容">
          <el-input v-model="question.content" type="textarea" :rows="3" placeholder="请输入问题" />
        </el-form-item>
        <el-form-item label="回答时限（秒）">
          <el-input-number v-model="question.timeLimit" :min="10" :max="300" />
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="showQuestion = false">取消</el-button>
        <el-button type="primary">发布提问</el-button>
      </template>
    </el-dialog>

    <!-- 发布公告对话框 -->
    <el-dialog v-model="showNotice" title="发布公告" width="450px">
      <el-form label-position="top">
        <el-form-item label="公告内容">
          <el-input v-model="notice.content" type="textarea" :rows="4" placeholder="请输入公告内容" />
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="showNotice = false">取消</el-button>
        <el-button type="primary">发布</el-button>
      </template>
    </el-dialog>

    <!-- 群信息对话框 -->
    <el-dialog v-model="showGroupInfo" title="群组信息" width="500px">
      <div class="group-info-content">
        <h4>群成员</h4>
        <div class="members-list">
          <!-- 成员列表 -->
        </div>
      </div>
    </el-dialog>
  </Layout>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { useUserStore } from '@/stores/user'
import Layout from '@/components/Layout.vue'

const userStore = useUserStore()

const groups = ref([])
const currentGroup = ref(null)
const messages = ref([])
const inputMessage = ref('')
const messagesRef = ref(null)

// 对话框控制
const showCreateGroup = ref(false)
const showGroupInfo = ref(false)
const showHomework = ref(false)
const showCheckin = ref(false)
const showQuestion = ref(false)
const showNotice = ref(false)

// 表单数据
const newGroup = reactive({ name: '', description: '' })
const homework = reactive({ title: '', content: '', deadline: null })
const checkin = reactive({ type: 'normal', question: '', answer: '', duration: 5 })
const question = reactive({ content: '', timeLimit: 60 })
const notice = reactive({ content: '' })

const selectGroup = (group) => {
  currentGroup.value = group
  // TODO: 加载群消息
}

const sendMessage = () => {
  if (!inputMessage.value.trim()) return
  // TODO: 发送消息API
  inputMessage.value = ''
}
</script>

<style scoped>
.group-page {
  height: calc(100vh - 180px);
}

.group-container {
  display: flex;
  height: 100%;
  background: #ffffff;
  border: 1px solid #d0d7de;
  border-radius: 6px;
  overflow: hidden;
}

.groups-panel {
  width: 260px;
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

.groups-list {
  flex: 1;
  overflow-y: auto;
}

.group-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  cursor: pointer;
  border-bottom: 1px solid #f0f0f0;
}

.group-item:hover {
  background: #f6f8fa;
}

.group-item.active {
  background: #ddf4ff;
}

.group-avatar {
  width: 40px;
  height: 40px;
  border-radius: 8px;
  background: #ddf4ff;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  color: #0969da;
}

.group-info {
  flex: 1;
}

.group-name {
  display: block;
  font-weight: 500;
  color: #1f2328;
}

.group-members {
  font-size: 12px;
  color: #656d76;
}

.empty-groups {
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
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.chat-title {
  font-weight: 600;
  color: #1f2328;
}

.chat-members {
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
  display: flex;
  gap: 10px;
  margin-bottom: 16px;
}

.message.mine {
  flex-direction: row-reverse;
}

.message-avatar {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: #f6f8fa;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  font-weight: 600;
  color: #656d76;
  flex-shrink: 0;
}

.message-body {
  max-width: 60%;
}

.message-sender {
  font-size: 12px;
  color: #656d76;
  margin-bottom: 4px;
}

.message.mine .message-sender {
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

.system-message {
  text-align: center;
  font-size: 12px;
  color: #656d76;
  padding: 8px;
}

.special-message {
  width: 100%;
  max-width: 400px;
  border: 1px solid #d0d7de;
  border-radius: 8px;
  overflow: hidden;
}

.special-header {
  padding: 8px 12px;
  font-weight: 600;
  font-size: 13px;
}

.special-message.homework .special-header {
  background: #fff8c5;
  color: #9a6700;
}

.special-message.checkin .special-header {
  background: #dafbe1;
  color: #1a7f37;
}

.special-message.question .special-header {
  background: #ddf4ff;
  color: #0969da;
}

.special-title {
  padding: 8px 12px 0;
  font-weight: 600;
  color: #1f2328;
}

.special-content {
  padding: 8px 12px;
  color: #656d76;
  font-size: 14px;
}

.special-footer {
  padding: 8px 12px;
  border-top: 1px solid #f0f0f0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 12px;
  color: #656d76;
}

.chat-input {
  border-top: 1px solid #d0d7de;
  padding: 12px 16px;
}

.teacher-actions {
  display: flex;
  gap: 8px;
  margin-bottom: 12px;
  flex-wrap: wrap;
}

.input-row {
  display: flex;
  gap: 12px;
}

.input-row .el-input {
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
