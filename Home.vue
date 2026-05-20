<template>
  <div class="home">
    <!-- 顶部横幅 -->
    <div class="banner">
      🎉 春季体检8折活动 · 点击查看 &gt;
    </div>
    
    <!-- 快捷入口（根据角色动态显示） -->
    <div class="shortcuts">
      <div class="shortcut-item" v-for="item in shortcuts" :key="item.name" @click="handleShortcut(item)">
        <div class="shortcut-icon">{{ item.icon }}</div>
        <div class="shortcut-name">{{ item.name }}</div>
      </div>
    </div>
    
    <!-- 新手教程专区（所有角色都可见） -->
    <div class="section">
      <div class="section-header">
        <h3>📹 新手教程专区</h3>
        <a href="#" class="more" @click.prevent="router.push('/tutorial')">查看更多 &gt;</a>
      </div>
      <div class="video-grid">
        <div class="video-card" v-for="video in videos" :key="video.title" @click="router.push('/tutorial')">
          <div class="video-cover">{{ video.cover }}</div>
          <div class="video-title">{{ video.title }}</div>
        </div>
      </div>
    </div>
    
   
<!-- 智能问答（点击跳转到独立页面） -->
<div class="section" @click="router.push('/qa')" style="cursor: pointer;">
  <div class="section-header">
    <h3>🤖 智能问答</h3>
    <a href="#" class="more" @click.prevent="router.push('/qa')">去问答 &gt;</a>
  </div>
  <div class="qa-preview">
    <div class="preview-question">💬 疫苗间隔多久？</div>
    <div class="preview-question">💬 驱虫药怎么选？</div>
    <div class="preview-question">💬 狗狗咳嗽怎么办？</div>
    <div class="preview-hint">点击进入完整问答界面 →</div>
  </div>
</div>
    
    <!-- 最近动态（仅宠物主人显示） -->
 <div class="section" @click="router.push('/daily-record')" style="cursor: pointer;">
  <div class="section-header">
    <h3>📋 最近动态</h3>
    <a href="#" class="more" @click.prevent="router.push('/daily-record')">查看全部 &gt;</a>
  </div>
      <div class="activity-list">
        <div class="activity-item">
          <span class="activity-icon">🐕</span>
          <span class="activity-text">豆豆 · 便便评分 ⭐⭐⭐⭐ 健康</span>
          <span class="activity-time">2小时前</span>
        </div>
        <div class="activity-item">
          <span class="activity-icon">🐈</span>
          <span class="activity-text">咪咪 · 狗粮余量不足</span>
          <span class="activity-time">昨天</span>
          <button class="buy-btn">立即购买</button>
        </div>
      </div>
    </div>

    <!-- 发视频模态框 -->
    <div v-if="showVideoModal" class="modal" @click.self="showVideoModal = false">
      <div class="modal-content">
        <h3>发布视频</h3>
        <input type="text" v-model="videoForm.title" placeholder="视频标题" class="modal-input">
        <textarea v-model="videoForm.desc" placeholder="视频描述" class="modal-textarea"></textarea>
        <div class="upload-area" @click="triggerFileUpload">
          <span v-if="!videoForm.file">📹 点击上传视频</span>
          <span v-else>✅ 已选择：{{ videoForm.file.name }}</span>
        </div>
        <input type="file" ref="fileInput" @change="handleFileSelect" accept="video/*" style="display:none">
        <div class="modal-actions">
          <button class="btn-cancel" @click="showVideoModal = false">取消</button>
          <button class="btn-submit" @click="submitVideo">发布</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { useRouter } from 'vue-router'
import { useUserStore } from '../stores/user'
import { storeToRefs } from 'pinia'

const router = useRouter()
const userStore = useUserStore()
const { currentRole } = storeToRefs(userStore)

// const question = ref('')

// const askQuestion = () => {
//   if (question.value) {
//     alert(`提问：${question.value}\n（AI回答功能开发中）`)
//     question.value = ''
//   }
// }

// 根据角色获取快捷入口
const getShortcutsByRole = () => {
  const role = currentRole.value
  
  // 宠物主人
  if (role === 'petOwner') {
    return [
      { name: '教程', icon: '📺', path: '/tutorial' },
      { name: '记录', icon: '📝', path: '/daily-record' },
      { name: 'AI识别', icon: '🤖', path: '/ai-recognize' },
      { name: '问答', icon: '💬', path: '/qa' },
      { name: '疫苗', icon: '💉', path: '/health-manage?tab=vaccine' },
      { name: '体重', icon: '⚖️', path: '/health-manage?tab=weight' },
      { name: '预约', icon: '🛎️', path: '/service-hall?tab=merchant' },
      { name: '发视频', icon: '📹', action: 'upload' }
    ]
  }
  
  // 护工
  if (role === 'caregiver') {
    return [
      { name: '教程', icon: '📺', path: '/tutorial' },
      { name: 'AI识别', icon: '🤖', path: '/ai-recognize' },
      { name: '问答', icon: '💬', path: '/qa' },
      { name: '我的工作', icon: '✅', path: '/my-work' },
      { name: '收益', icon: '💰', path: '/earnings' },
      { name: '任务大厅', icon: '📋', path: '/service-hall?tab=task' },
      { name: '发视频', icon: '📹', action: 'upload' }
    ]
  }
  
  // 商家
  if (role === 'merchant') {
    return [
      { name: '教程', icon: '📺', path: '/tutorial' },
      { name: 'AI识别', icon: '🤖', path: '/ai-recognize' },
      { name: '问答', icon: '💬', path: '/qa' },
      { name: '店铺管理', icon: '🏪', path: '/shop-manage' },
      { name: '订单处理', icon: '📦', path: '/orders' },
      { name: '数据统计', icon: '📊', path: '/statistics' },
      { name: '发视频', icon: '📹', action: 'upload' }
    ]
  }
  
  // 管理员
  return [
    { name: '视频审核', icon: '🎬', path: '/video-audit' },
    { name: '资质审核', icon: '✅', path: '/qualification-audit' },
    { name: '纠纷仲裁', icon: '⚖️', path: '/dispute' },
    { name: '用户反馈', icon: '💬', path: '/user-feedback' }
  ]
}

const shortcuts = ref(getShortcutsByRole())

// 监听角色变化，动态更新快捷入口
watch(currentRole, () => {
  shortcuts.value = getShortcutsByRole()
})

// 视频列表（所有角色都可见）
const videos = [
  { title: '训练狗狗坐下', cover: '🎬' },
  { title: '猫咪喂养指南', cover: '🎬' },
  { title: '宠物医疗急救', cover: '🎬' }
]

// 快捷入口点击处理
const handleShortcut = (item) => {
  if (item.action === 'upload') {
    showVideoModal.value = true
  } else if (item.path) {
    router.push(item.path)
  } else {
    alert(`${item.name}功能开发中`)
  }
}

// 发视频相关
const showVideoModal = ref(false)
const fileInput = ref(null)
const videoForm = ref({
  title: '',
  desc: '',
  file: null
})

const triggerFileUpload = () => {
  fileInput.value.click()
}

const handleFileSelect = (e) => {
  videoForm.value.file = e.target.files[0]
}

const submitVideo = () => {
  if (!videoForm.value.title || !videoForm.value.file) {
    alert('请填写标题并选择视频文件')
    return
  }
  
  // 保存到待审核列表
  const pendingVideos = JSON.parse(localStorage.getItem('pendingVideos') || '[]')
  pendingVideos.push({
    id: Date.now(),
    title: videoForm.value.title,
    desc: videoForm.value.desc,
    user: '当前用户',
    duration: '未知',
    status: 'pending'
  })
  localStorage.setItem('pendingVideos', JSON.stringify(pendingVideos))
  
  alert(`视频「${videoForm.value.title}」已提交，等待管理员审核`)
  showVideoModal.value = false
  videoForm.value = { title: '', desc: '', file: null }
}
</script>

<style scoped>
/* 样式保持不变，和之前一样 */
.home { max-width: 1200px; }
.banner { background: linear-gradient(135deg, #2ecc71, #27ae60); color: white; padding: 16px 24px; border-radius: 16px; margin-bottom: 28px; cursor: pointer; font-weight: 500; }
.shortcuts { display: grid; grid-template-columns: repeat(4, 1fr); gap: 16px; margin-bottom: 32px; }
.shortcut-item { background: white; padding: 20px; border-radius: 16px; text-align: center; cursor: pointer; box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05); transition: transform 0.2s; }
.shortcut-item:hover { transform: translateY(-4px); box-shadow: 0 6px 16px rgba(0, 0, 0, 0.1); }
.shortcut-icon { font-size: 32px; margin-bottom: 8px; }
.shortcut-name { font-size: 14px; color: #333; }
.section { background: white; border-radius: 16px; padding: 20px 24px; margin-bottom: 24px; box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05); }
.section-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 16px; }
.section-header h3 { font-size: 18px; color: #1a3c34; }
.more { color: #2ecc71; text-decoration: none; font-size: 14px; cursor: pointer; }
.video-grid { display: flex; gap: 20px; }
.video-card { flex: 1; background: #f8f9fa; border-radius: 12px; padding: 16px; text-align: center; cursor: pointer; }
.video-cover { font-size: 48px; margin-bottom: 12px; }
.video-title { font-size: 14px; color: #333; }
.qa-input { display: flex; gap: 12px; margin-bottom: 16px; }
.qa-input input { flex: 1; padding: 12px 16px; border: 1px solid #e0e0e0; border-radius: 24px; font-size: 14px; outline: none; }
.qa-input input:focus { border-color: #2ecc71; }
.qa-input button { padding: 12px 24px; background: #2ecc71; color: white; border: none; border-radius: 24px; cursor: pointer; font-weight: 500; }
.qa-common { display: flex; gap: 12px; flex-wrap: wrap; }
.qa-tag { padding: 6px 16px; background: #f0f0f0; border-radius: 20px; font-size: 13px; cursor: pointer; }
.activity-item { display: flex; align-items: center; gap: 12px; padding: 12px 0; border-bottom: 1px solid #f0f0f0; }
.activity-icon { font-size: 24px; }
.activity-text { flex: 1; font-size: 14px; color: #333; }
.activity-time { font-size: 12px; color: #999; }
.buy-btn { padding: 6px 16px; background: #ff9800; color: white; border: none; border-radius: 16px; cursor: pointer; font-size: 12px; }

/* 模态框样式 */
.modal { position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: rgba(0, 0, 0, 0.5); display: flex; align-items: center; justify-content: center; z-index: 1000; }
.modal-content { background: white; border-radius: 20px; padding: 24px; width: 450px; max-width: 90%; }
.modal-content h3 { margin-bottom: 20px; color: #1a3c34; }
.modal-input { width: 100%; padding: 12px; margin-bottom: 16px; border: 1px solid #ddd; border-radius: 12px; font-size: 14px; box-sizing: border-box; }
.modal-textarea { width: 100%; padding: 12px; margin-bottom: 16px; border: 1px solid #ddd; border-radius: 12px; font-size: 14px; min-height: 100px; resize: vertical; box-sizing: border-box; font-family: inherit; }
.upload-area { background: #f8f9fa; border: 2px dashed #ddd; border-radius: 12px; padding: 20px; text-align: center; cursor: pointer; margin-bottom: 20px; }
.modal-actions { display: flex; gap: 12px; justify-content: flex-end; }
.btn-cancel { background: #f0f0f0; border: none; padding: 10px 24px; border-radius: 24px; cursor: pointer; }
.btn-submit { background: #2ecc71; color: white; border: none; padding: 10px 24px; border-radius: 24px; cursor: pointer; }
.qa-preview {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.preview-question {
  padding: 8px 12px;
  background: #f8f9fa;
  border-radius: 20px;
  font-size: 13px;
  color: #555;
}

.preview-hint {
  margin-top: 8px;
  font-size: 12px;
  color: #2ecc71;
  text-align: right;
}
</style>