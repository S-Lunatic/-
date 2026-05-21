<template>
  <div class="service-hall">
    <!-- Tab 切换 -->
    <div class="tabs">
      <!-- 宠物主人：显示两个Tab -->
      <template v-if="userStore.currentRole === 'petOwner'">
        <div 
          class="tab" 
          :class="{ active: activeTab === 'merchant' }"
          @click="activeTab = 'merchant'"
        >
          🏪 商家预约
        </div>
        <div 
          class="tab" 
          :class="{ active: activeTab === 'task' }"
          @click="activeTab = 'task'"
        >
          📋 任务大厅
        </div>
      </template>
      
      <!-- 护工：只显示任务大厅（默认选中） -->
      <template v-if="userStore.currentRole === 'caregiver'">
        <div class="tab active">📋 任务大厅</div>
      </template>
      
      <!-- 商家：只显示商家预约（默认选中） -->
      <template v-if="userStore.currentRole === 'merchant'">
        <div class="tab active">🏪 商家预约</div>
      </template>
    </div>
    
    <!-- 商家预约内容 -->
    <div v-if="activeTab === 'merchant'" class="merchant-panel">
      <div class="search-bar">
        <input 
          type="text" 
          v-model="searchKeyword" 
          placeholder="🔍 搜索商家名称或服务..." 
          class="search-input"
          @input="filterMerchants"
        >
        <div class="filters">
          <button 
            class="filter-btn" 
            :class="{ active: filterType === 'all' }"
            @click="setFilter('all')"
          >
            全部
          </button>
          <button 
            class="filter-btn" 
            :class="{ active: filterType === 'petShop' }"
            @click="setFilter('petShop')"
          >
            宠物店
          </button>
          <button 
            class="filter-btn" 
            :class="{ active: filterType === 'hospital' }"
            @click="setFilter('hospital')"
          >
            医院
          </button>
        </div>
      </div>
      
      <div class="merchant-list">
        <div class="merchant-card" v-for="shop in filteredMerchants" :key="shop.name">
          <div class="shop-info">
            <div class="shop-name">{{ shop.name }}</div>
            <div class="shop-rating">⭐ {{ shop.rating }} · 距离{{ shop.distance }}</div>
            <div class="shop-services">{{ shop.services }}</div>
            <div class="shop-hours">{{ shop.hours }}</div>
          </div>
          <div class="shop-actions">
            <button class="detail-btn" @click="viewShopDetail(shop)">查看详情</button>
            <button class="appoint-btn" @click="openAppointModal(shop)">预约</button>
          </div>
        </div>
      </div>
    </div>
    
   <!-- 任务大厅内容 -->
<div v-if="activeTab === 'task'" class="task-panel">
  <div class="task-header" v-if="userStore.currentRole === 'petOwner'">
    <button class="publish-btn" @click="openPublishModal">➕ 发布新任务</button>
    <span class="guarantee">平台担保交易 ✅</span>
  </div>

        <!-- 护工：不显示发布按钮 -->
  <div class="task-header" v-if="userStore.currentRole === 'caregiver'">
    <span class="guarantee">平台担保交易 ✅</span>
  </div>
      
      <div class="task-list">
        <div class="task-card" v-for="task in tasks" :key="task.id">
          <div class="task-info">
            <div class="task-title">{{ task.icon }} {{ task.title }}</div>
            <div class="task-reward">💰 {{ task.reward }}</div>
            <div class="task-detail">时间：{{ task.time }} · 地点：{{ task.location }}</div>
            <div class="task-detail">宠物：{{ task.pet }} · 要求：{{ task.requirement }}</div>
            <div class="task-footer">距离您 {{ task.distance }} · {{ task.postTime }}</div>
          </div>
          
          <!-- 根据角色和发布者显示不同内容 -->
          <button 
            v-if="userStore.currentRole === 'caregiver'" 
            class="accept-btn" 
            @click="acceptTask(task)"
          >
            立即接单
          </button>
          
          <span 
            v-else-if="userStore.currentRole === 'petOwner' && isMyTask(task)" 
            class="my-task-badge"
          >
            📋 我发布的任务
          </span>
        </div>
        
        <div v-if="tasks.length === 0" class="empty-state">
          暂无任务，快来发布第一个任务吧！
        </div>
      </div>
    </div>

    <!-- 预约弹窗 -->
    <div v-if="showAppointModal" class="modal" @click.self="closeAppointModal">
      <div class="modal-content">
        <h3>📅 预约服务</h3>
        <div class="shop-info-modal">
          <p><strong>{{ selectedShop?.name }}</strong></p>
          <p>{{ selectedShop?.services }}</p>
        </div>
        
        <select v-model="appointForm.service" class="modal-select">
          <option value="">请选择服务项目</option>
          <option v-for="service in serviceOptions" :key="service" :value="service">{{ service }}</option>
        </select>
        
        <select v-model="appointForm.petId" class="modal-select">
          <option value="1">豆豆（金毛）</option>
          <option value="2">咪咪（英短）</option>
        </select>
        
        <input type="datetime-local" v-model="appointForm.datetime" class="modal-input">
        <input type="text" v-model="appointForm.remark" placeholder="备注（选填）" class="modal-input">
        
        <div class="modal-actions">
          <button class="btn-cancel" @click="closeAppointModal">取消</button>
          <button class="btn-submit" @click="submitAppointment">提交预约</button>
        </div>
      </div>
    </div>

    <!-- 发布任务弹窗 -->
    <div v-if="showPublishModal" class="modal" @click.self="closePublishModal">
      <div class="modal-content">
        <h3>📝 发布新任务</h3>
        
        <select v-model="publishForm.type" class="modal-select">
          <option value="">任务类型</option>
          <option value="遛狗">🐕 遛狗</option>
          <option value="上门喂养">🐈 上门喂养</option>
          <option value="陪玩">🐕 陪玩</option>
          <option value="其他">📝 其他</option>
        </select>
        
        <select v-model="publishForm.petId" class="modal-select">
          <option value="1">豆豆（金毛）</option>
          <option value="2">咪咪（英短）</option>
        </select>
        
        <input type="text" v-model="publishForm.title" placeholder="任务标题" class="modal-input">
        <input type="number" v-model="publishForm.reward" placeholder="报酬（元）" class="modal-input" step="10">
        <input type="datetime-local" v-model="publishForm.datetime" class="modal-input">
        <input type="text" v-model="publishForm.location" placeholder="服务地点" class="modal-input">
        <input type="text" v-model="publishForm.requirement" placeholder="特殊要求（选填）" class="modal-input">
        
        <div class="modal-actions">
          <button class="btn-cancel" @click="closePublishModal">取消</button>
          <button class="btn-submit" @click="submitTask">发布任务</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import { useRouter } from 'vue-router'
import { useUserStore } from '../stores/user'
import { storeToRefs } from 'pinia'

const router = useRouter()
const userStore = useUserStore()
const { currentRole } = storeToRefs(userStore)

const activeTab = ref('merchant')

// ========== 搜索筛选 ==========
const searchKeyword = ref('')
const filterType = ref('all')

const allMerchants = ref([
  { 
    name: '🏥 安心宠物医院', 
    type: 'hospital',
    rating: '4.9', 
    distance: '300m', 
    services: '疫苗/体检/绝育/皮肤病诊疗', 
    hours: '营业：09:00-20:00 · 今日可预约',
    address: '阳光大道123号',
    phone: '138-0000-1234'
  },
  { 
    name: '🏪 宠乐宠物店', 
    type: 'petShop',
    rating: '4.8', 
    distance: '500m', 
    services: '美容/洗澡/寄养/用品销售', 
    hours: '营业：10:00-21:00 · 需提前2小时',
    address: '阳光大道88号',
    phone: '138-0000-5678'
  },
  { 
    name: '🏥 瑞康动物医院', 
    type: 'hospital',
    rating: '4.7', 
    distance: '1.2km', 
    services: '24小时急诊/住院/影像科', 
    hours: '营业：24小时 · 夜间加收20%',
    address: '健康路45号',
    phone: '138-0000-9012'
  }
])

const filteredMerchants = computed(() => {
  let result = allMerchants.value
  
  if (filterType.value === 'petShop') {
    result = result.filter(m => m.type === 'petShop')
  } else if (filterType.value === 'hospital') {
    result = result.filter(m => m.type === 'hospital')
  }
  
  if (searchKeyword.value) {
    const keyword = searchKeyword.value.toLowerCase()
    result = result.filter(m => 
      m.name.toLowerCase().includes(keyword) || 
      m.services.toLowerCase().includes(keyword)
    )
  }
  
  return result
})

const setFilter = (type) => {
  filterType.value = type
}

// 查看商家详情
const viewShopDetail = (shop) => {
  sessionStorage.setItem('currentShop', JSON.stringify(shop))
  router.push('/shop-detail')
}

// ========== 预约弹窗 ==========
const showAppointModal = ref(false)
const selectedShop = ref(null)
const appointForm = ref({
  service: '',
  petId: '1',
  datetime: '',
  remark: ''
})

const serviceOptions = ref(['疫苗接种', '体检套餐', '驱虫服务', '美容洗澡', '寄养服务'])

const openAppointModal = (shop) => {
  selectedShop.value = shop
  appointForm.value = {
    service: '',
    petId: '1',
    datetime: '',
    remark: ''
  }
  showAppointModal.value = true
}

const closeAppointModal = () => {
  showAppointModal.value = false
  selectedShop.value = null
}

const submitAppointment = () => {
  if (!appointForm.value.service) {
    alert('请选择服务项目')
    return
  }
  if (!appointForm.value.datetime) {
    alert('请选择预约时间')
    return
  }
  
  const appointments = JSON.parse(localStorage.getItem('appointments') || '[]')
  appointments.push({
    id: Date.now(),
    shopName: selectedShop.value.name,
    service: appointForm.value.service,
    petId: appointForm.value.petId,
    datetime: appointForm.value.datetime,
    remark: appointForm.value.remark,
    status: 'pending',
    createdAt: new Date().toISOString()
  })
  localStorage.setItem('appointments', JSON.stringify(appointments))
  
  alert(`已预约「${selectedShop.value.name}」的${appointForm.value.service}服务，请等待商家确认`)
  closeAppointModal()
}

// ========== 任务大厅 ==========
const tasks = ref([])

// 加载所有待接单的任务
const loadTasks = () => {
  const publishedTasks = JSON.parse(localStorage.getItem('publishedTasks') || '[]')
  tasks.value = publishedTasks.filter(t => t.status === 'pending')
}

// 判断是否是当前用户发布的任务
const isMyTask = (task) => {
  const myTasks = JSON.parse(localStorage.getItem('publishedTasks') || '[]')
  return myTasks.some(t => t.id === task.id)
}

// ========== 发布任务弹窗 ==========
const showPublishModal = ref(false)
const publishForm = ref({
  type: '',
  petId: '1',
  title: '',
  reward: '',
  datetime: '',
  location: '',
  requirement: ''
})

const openPublishModal = () => {
  publishForm.value = {
    type: '',
    petId: '1',
    title: '',
    reward: '',
    datetime: '',
    location: '',
    requirement: ''
  }
  showPublishModal.value = true
}

const closePublishModal = () => {
  showPublishModal.value = false
}

const submitTask = () => {
  if (!publishForm.value.type) {
    alert('请选择任务类型')
    return
  }
  if (!publishForm.value.title) {
    alert('请填写任务标题')
    return
  }
  if (!publishForm.value.reward) {
    alert('请填写报酬')
    return
  }
  if (!publishForm.value.datetime) {
    alert('请选择时间')
    return
  }
  if (!publishForm.value.location) {
    alert('请填写服务地点')
    return
  }
  
  const publishedTasks = JSON.parse(localStorage.getItem('publishedTasks') || '[]')
  const newTask = {
    id: Date.now(),
    icon: publishForm.value.type === '遛狗' ? '🐕' : publishForm.value.type === '上门喂养' ? '🐈' : '📝',
    title: publishForm.value.title,
    reward: `${publishForm.value.reward}元/小时`,
    time: new Date(publishForm.value.datetime).toLocaleString(),
    location: publishForm.value.location,
    pet: publishForm.value.petId === '1' ? '金毛' : '英短',
    requirement: publishForm.value.requirement || '无',
    distance: '待接单',
    postTime: '刚刚发布',
    status: 'pending',
    acceptedBy: null,
    disputeFiled: false,
    messages: []
  }
  
  publishedTasks.push(newTask)
  localStorage.setItem('publishedTasks', JSON.stringify(publishedTasks))
  loadTasks()
  
  alert(`任务「${publishForm.value.title}」已发布，等待护工接单`)
  closePublishModal()
  activeTab.value = 'task'
}
// 接单（护工）
const acceptTask = (task) => {
  if (!confirm(`确定要接单「${task.title}」吗？`)) return
  
  // 获取当前护工名称（后续从 store 获取）
  const currentCaregiver = userStore.userInfo?.name || '当前护工'
  
  // 1. 更新任务状态
  const publishedTasks = JSON.parse(localStorage.getItem('publishedTasks') || '[]')
  const taskIndex = publishedTasks.findIndex(t => t.id === task.id)
  
  if (taskIndex !== -1) {
    publishedTasks[taskIndex].status = 'processing'
    publishedTasks[taskIndex].acceptedBy = currentCaregiver
    publishedTasks[taskIndex].acceptedTime = new Date().toLocaleString()
    localStorage.setItem('publishedTasks', JSON.stringify(publishedTasks))
    
    // 2. 刷新任务大厅
    loadTasks()
    
    alert(`已接单：「${task.title}」\n请前往「我的工作」查看`)
  }
}

// ========== 根据角色设置默认Tab（移到所有变量定义之后） ==========
const setDefaultTab = () => {
  if (userStore.currentRole === 'petOwner') {
    activeTab.value = 'merchant'
  } else if (userStore.currentRole === 'caregiver') {
    activeTab.value = 'task'
  } else if (userStore.currentRole === 'merchant') {
    activeTab.value = 'merchant'
  }
}

// 监听角色变化
watch(() => userStore.currentRole, () => {
  setDefaultTab()
}, { immediate: true })

// 初始化加载
onMounted(() => {
  loadTasks()
})
</script>

<style scoped>
.service-hall {
  max-width: 1000px;
}

.tabs {
  display: flex;
  gap: 4px;
  background: white;
  border-radius: 48px;
  padding: 4px;
  margin-bottom: 24px;
  width: fit-content;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.tab {
  padding: 10px 32px;
  border-radius: 40px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.2s;
}

.tab.active {
  background: #2ecc71;
  color: white;
}

.search-bar {
  background: white;
  border-radius: 16px;
  padding: 16px 20px;
  margin-bottom: 20px;
  display: flex;
  gap: 16px;
  flex-wrap: wrap;
  align-items: center;
}

.search-input {
  flex: 1;
  padding: 10px 16px;
  border: 1px solid #e0e0e0;
  border-radius: 24px;
  outline: none;
}

.filters {
  display: flex;
  gap: 8px;
}

.filter-btn {
  padding: 8px 16px;
  background: #f0f0f0;
  border: none;
  border-radius: 24px;
  cursor: pointer;
  transition: all 0.2s;
}

.filter-btn.active {
  background: #2ecc71;
  color: white;
}

.merchant-list, .task-list {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.merchant-card {
  background: white;
  border-radius: 16px;
  padding: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.shop-name {
  font-size: 18px;
  font-weight: 600;
  margin-bottom: 6px;
}

.shop-rating {
  font-size: 14px;
  color: #ff9800;
  margin-bottom: 6px;
}

.shop-services {
  font-size: 13px;
  color: #666;
  margin-bottom: 4px;
}

.shop-hours {
  font-size: 12px;
  color: #999;
}

.shop-actions {
  display: flex;
  gap: 12px;
}

.detail-btn, .appoint-btn {
  padding: 8px 20px;
  border-radius: 24px;
  cursor: pointer;
}

.detail-btn {
  background: none;
  border: 1px solid #ddd;
}

.appoint-btn {
  background: #2ecc71;
  color: white;
  border: none;
}

.task-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.publish-btn {
  background: #2ecc71;
  color: white;
  border: none;
  padding: 10px 24px;
  border-radius: 24px;
  cursor: pointer;
  font-weight: 500;
}

.guarantee {
  font-size: 13px;
  color: #ff9800;
}

.task-card {
  background: white;
  border-radius: 16px;
  padding: 20px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.task-info {
  flex: 1;
}

.task-title {
  font-size: 18px;
  font-weight: 600;
  margin-bottom: 6px;
}

.task-reward {
  font-size: 18px;
  color: #ff9800;
  font-weight: 600;
  margin-bottom: 8px;
}

.task-detail {
  font-size: 13px;
  color: #666;
  margin-bottom: 4px;
}

.task-footer {
  font-size: 12px;
  color: #999;
  margin-top: 8px;
}

.accept-btn {
  background: #2ecc71;
  color: white;
  border: none;
  padding: 10px 28px;
  border-radius: 24px;
  cursor: pointer;
  margin-left: 16px;
}

.my-task-badge {
  background: #e8f5e9;
  color: #2ecc71;
  padding: 8px 20px;
  border-radius: 24px;
  font-size: 13px;
  font-weight: 500;
  margin-left: 16px;
  white-space: nowrap;
}

.empty-state {
  text-align: center;
  padding: 60px;
  color: #999;
  background: white;
  border-radius: 16px;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  border-radius: 20px;
  padding: 24px;
  width: 450px;
  max-width: 90%;
}

.modal-content h3 {
  margin-bottom: 20px;
  color: #1a3c34;
}

.shop-info-modal {
  background: #f8f9fa;
  padding: 12px;
  border-radius: 12px;
  margin-bottom: 16px;
}

.shop-info-modal p {
  margin: 4px 0;
  font-size: 14px;
}

.modal-input, .modal-select {
  width: 100%;
  padding: 12px;
  margin-bottom: 16px;
  border: 1px solid #ddd;
  border-radius: 12px;
  font-size: 14px;
  box-sizing: border-box;
}

.modal-actions {
  display: flex;
  gap: 12px;
  justify-content: flex-end;
  margin-top: 8px;
}

.btn-cancel {
  background: #f0f0f0;
  border: none;
  padding: 10px 20px;
  border-radius: 24px;
  cursor: pointer;
}

.btn-submit {
  background: #2ecc71;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 24px;
  cursor: pointer;
}
</style>