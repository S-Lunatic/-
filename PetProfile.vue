<template>
  <div class="pet-profile">
    <!-- 宠物切换栏 -->
    <div class="pet-tabs">
      <div 
        v-for="pet in pets" 
        :key="pet.id"
        class="pet-tab"
        :class="{ active: currentPet?.id === pet.id }"
        @click="selectPet(pet)"
      >
        <span class="pet-avatar">{{ pet.species === '金毛' ? '🐕' : '🐈' }}</span>
        <span class="pet-name">{{ pet.name }}</span>
      </div>
      <div class="pet-tab add-pet" @click="openAddPetModal">
        <span class="pet-avatar">➕</span>
        <span class="pet-name">添加</span>
      </div>
    </div>
    
    <!-- 当前宠物详情 -->
    <div v-if="currentPet" class="pet-detail">
      <!-- 基本信息 -->
      <div class="info-card">
        <h4>📋 基本信息</h4>
        <div class="info-grid">
          <div class="info-item"><label>名字：</label>{{ currentPet.name }}</div>
          <div class="info-item"><label>品种：</label>{{ currentPet.species }}</div>
          <div class="info-item"><label>年龄：</label>{{ currentPet.age }}岁</div>
          <div class="info-item"><label>体重：</label>{{ currentPet.weight }}kg</div>
          <div class="info-item"><label>芯片号：</label>{{ currentPet.chipNumber || '未录入' }}</div>
        </div>
      </div>
      
      <!-- 医疗云档案 -->
      <div class="info-card">
        <h4>🏥 医疗云档案</h4>
        <button class="auth-btn" @click="openAuthModal">+ 授权医院/护工查看</button>
        <div class="auth-list">
          <span class="auth-tag" v-for="auth in authorizedList" :key="auth.name">
            ✅ {{ auth.name }}（{{ auth.type }}）
            <span class="auth-remove" @click="removeAuth(auth)">✖</span>
          </span>
          <span v-if="authorizedList.length === 0" class="auth-empty">暂无授权</span>
        </div>
      </div>
      
      <!-- 健康数据互通 -->
      <div class="info-card">
        <h4>📊 健康数据互通</h4>
        <div class="health-stats">
          <div class="stat">最近体重：{{ currentPet.weight }}kg (3天前)</div>
          <div class="stat">下次疫苗：2026-06-15 (剩余34天)</div>
          <div class="stat">下次驱虫：2026-05-20 (剩余9天)</div>
        </div>
        <button class="goto-btn" @click="goToHealth">前往健康管理 &gt;</button>
      </div>
      
      <!-- 最近医疗记录 -->
      <div class="info-card">
        <h4>📑 最近医疗记录</h4>
        <div class="record-item">
          <span class="record-date">2025-12-01</span>
          <span class="record-hospital">安心医院</span>
          <span class="record-desc">皮肤科 · 过敏治疗</span>
        </div>
        <div class="record-item">
          <span class="record-date">2025-08-10</span>
          <span class="record-hospital">安心医院</span>
          <span class="record-desc">年度体检 · 健康</span>
        </div>
    <a href="#" class="more-link" @click.prevent="goToMedicalRecords">查看全部 &gt;</a>
      </div>
      
      <!-- 最近日常记录 -->
      <div class="info-card">
        <h4>📝 最近日常记录</h4>
        <div class="record-item">
          <span class="record-date">昨天 18:30</span>
          <span class="record-desc">便便正常 ⭐⭐⭐⭐</span>
          <span class="photo-icon">📷</span>
        </div>
        <div class="record-item">
          <span class="record-date">昨天 08:00</span>
          <span class="record-desc">狗粮150g</span>
        </div>
     <a href="#" class="more-link" @click.prevent="goToDailyRecords">查看全部 &gt;</a>
      </div>
    </div>

    <!-- 添加宠物弹窗 -->
    <div v-if="showAddPetModal" class="modal" @click.self="closeAddPetModal">
      <div class="modal-content">
        <h3>➕ 添加宠物</h3>
        <input type="text" v-model="newPet.name" placeholder="宠物名称 *" class="modal-input">
        <input type="text" v-model="newPet.species" placeholder="品种 *" class="modal-input">
        <input type="text" v-model="newPet.age" placeholder="年龄（岁）" class="modal-input">
        <input type="text" v-model="newPet.weight" placeholder="体重（kg）" class="modal-input">
        <input type="text" v-model="newPet.chipNumber" placeholder="芯片号（选填）" class="modal-input">
        <div class="modal-actions">
          <button class="btn-cancel" @click="closeAddPetModal">取消</button>
          <button class="btn-submit" @click="savePet">保存</button>
        </div>
      </div>
    </div>

    <!-- 授权弹窗 -->
    <div v-if="showAuthModal" class="modal" @click.self="closeAuthModal">
      <div class="modal-content">
        <h3>🔐 授权医疗云档案查看</h3>
        <div class="auth-select-area">
          <div class="auth-select-title">选择授权对象：</div>
          <div class="auth-options">
            <label class="auth-option">
              <input type="radio" v-model="authType" value="hospital"> 🏥 医院
            </label>
            <label class="auth-option">
              <input type="radio" v-model="authType" value="caregiver"> 🤝 护工
            </label>
          </div>
          
          <!-- 选择医院 -->
          <select v-if="authType === 'hospital'" v-model="selectedHospital" class="modal-select">
            <option value="">请选择医院</option>
            <option value="安心宠物医院">安心宠物医院</option>
            <option value="瑞康动物医院">瑞康动物医院</option>
            <option value="宠乐宠物医院">宠乐宠物医院</option>
          </select>
          
          <!-- 选择护工 -->
          <select v-if="authType === 'caregiver'" v-model="selectedCaregiver" class="modal-select">
            <option value="">请选择护工</option>
            <option value="李护工">李护工（好评率98%）</option>
            <option value="王护工">王护工（好评率95%）</option>
            <option value="张护工">张护工（好评率92%）</option>
          </select>
          
          <div class="auth-expire">
            <label>授权有效期：</label>
            <select v-model="expireDays" class="expire-select">
              <option value="7">7天</option>
              <option value="30">30天</option>
              <option value="90">90天</option>
              <option value="365">永久</option>
            </select>
          </div>
        </div>
        <div class="modal-actions">
          <button class="btn-cancel" @click="closeAuthModal">取消</button>
          <button class="btn-submit" @click="saveAuth">确认授权</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()

// 宠物列表
const pets = ref([
  { id: 1, name: '豆豆', species: '金毛', age: 3, weight: 28, chipNumber: '123456789' },
  { id: 2, name: '咪咪', species: '英短', age: 2, weight: 4.5, chipNumber: '' }
])
const currentPet = ref(pets.value[0])

// 授权列表
const authorizedList = ref([
  { name: '安心宠物医院', type: '医院' },
  { name: '李护工', type: '护工' }
])

// 切换宠物
const selectPet = (pet) => {
  currentPet.value = pet
}

// 跳转健康管理
const goToHealth = () => {
  router.push('/health-manage')
}

// ========== 添加宠物弹窗 ==========
const showAddPetModal = ref(false)
const newPet = ref({
  name: '',
  species: '',
  age: '',
  weight: '',
  chipNumber: ''
})

const openAddPetModal = () => {
  showAddPetModal.value = true
}

const closeAddPetModal = () => {
  showAddPetModal.value = false
  newPet.value = { name: '', species: '', age: '', weight: '', chipNumber: '' }
}

const savePet = () => {
  if (!newPet.value.name || !newPet.value.species) {
    alert('请填写宠物名称和品种')
    return
  }
  
  const newId = Math.max(...pets.value.map(p => p.id), 0) + 1
  const petToAdd = {
    id: newId,
    name: newPet.value.name,
    species: newPet.value.species,
    age: newPet.value.age || '0',
    weight: newPet.value.weight || '0',
    chipNumber: newPet.value.chipNumber || ''
  }
  
  pets.value.push(petToAdd)
  alert(`宠物「${newPet.value.name}」添加成功！\n（后端开发完成后将保存到数据库）`)
  closeAddPetModal()
}

// ========== 授权弹窗 ==========
const showAuthModal = ref(false)
const authType = ref('hospital')
const selectedHospital = ref('')
const selectedCaregiver = ref('')
const expireDays = ref(30)

const openAuthModal = () => {
  authType.value = 'hospital'
  selectedHospital.value = ''
  selectedCaregiver.value = ''
  expireDays.value = 30
  showAuthModal.value = true
}

const closeAuthModal = () => {
  showAuthModal.value = false
}

const saveAuth = () => {
  let authName = ''
  let authTypeText = ''
  
  if (authType.value === 'hospital') {
    if (!selectedHospital.value) {
      alert('请选择医院')
      return
    }
    authName = selectedHospital.value
    authTypeText = '医院'
  } else {
    if (!selectedCaregiver.value) {
      alert('请选择护工')
      return
    }
    authName = selectedCaregiver.value
    authTypeText = '护工'
  }
  
  // 检查是否已授权
  if (authorizedList.value.some(a => a.name === authName)) {
    alert(`「${authName}」已被授权，请勿重复添加`)
    closeAuthModal()
    return
  }
  
  authorizedList.value.push({ name: authName, type: authTypeText })
  alert(`已授权「${authName}」查看「${currentPet.value.name}」的医疗档案，有效期${expireDays.value}天\n（后端开发完成后将保存到数据库）`)
  closeAuthModal()
}

const removeAuth = (auth) => {
  if (confirm(`确定要取消「${auth.name}」的授权吗？`)) {
    const index = authorizedList.value.indexOf(auth)
    if (index !== -1) {
      authorizedList.value.splice(index, 1)
      alert(`已取消「${auth.name}」的授权`)
    }
  }
}
// 跳转到医疗记录完整页面
const goToMedicalRecords = () => {
  router.push('/health-manage')
}

// 跳转到日常记录完整页面
const goToDailyRecords = () => {
  router.push('/daily-record')
}
</script>

<style scoped>
.pet-profile {
  max-width: 1000px;
}

.pet-tabs {
  display: flex;
  gap: 12px;
  margin-bottom: 28px;
  flex-wrap: wrap;
}

.pet-tab {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 20px;
  background: white;
  border-radius: 40px;
  cursor: pointer;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
  transition: all 0.2s;
}

.pet-tab.active {
  background: #2ecc71;
  color: white;
  box-shadow: 0 4px 12px rgba(46, 204, 113, 0.3);
}

.pet-tab.add-pet {
  background: #f0f0f0;
}

.pet-avatar {
  font-size: 24px;
}

.pet-name {
  font-size: 15px;
  font-weight: 500;
}

.info-card {
  background: white;
  border-radius: 16px;
  padding: 20px 24px;
  margin-bottom: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.info-card h4 {
  font-size: 18px;
  color: #1a3c34;
  margin-bottom: 16px;
}

.info-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 12px;
}

.info-item {
  font-size: 14px;
  color: #555;
}

.info-item label {
  font-weight: 600;
  color: #333;
}

.auth-btn {
  background: #f0f0f0;
  border: none;
  padding: 8px 16px;
  border-radius: 20px;
  cursor: pointer;
  font-size: 13px;
  margin-bottom: 12px;
}

.auth-list {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
}

.auth-tag {
  background: #e8f5e9;
  color: #2ecc71;
  padding: 4px 12px;
  border-radius: 16px;
  font-size: 13px;
  display: inline-flex;
  align-items: center;
  gap: 6px;
}

.auth-remove {
  cursor: pointer;
  font-size: 12px;
  color: #ff4757;
}

.auth-empty {
  font-size: 13px;
  color: #999;
}

.health-stats {
  margin-bottom: 12px;
}

.stat {
  font-size: 14px;
  color: #555;
  padding: 6px 0;
}

.goto-btn {
  background: none;
  border: none;
  color: #2ecc71;
  cursor: pointer;
  font-size: 13px;
}

.record-item {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 10px 0;
  border-bottom: 1px solid #f0f0f0;
  font-size: 14px;
}

.record-date {
  width: 100px;
  color: #999;
}

.record-hospital {
  width: 100px;
  color: #555;
}

.record-desc {
  flex: 1;
  color: #333;
}

.photo-icon {
  cursor: pointer;
  color: #2ecc71;
}

.more-link {
  display: inline-block;
  margin-top: 12px;
  color: #2ecc71;
  text-decoration: none;
  font-size: 13px;
}

/* 模态框样式 */
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

.modal-input, .modal-select {
  width: 100%;
  padding: 12px;
  margin-bottom: 16px;
  border: 1px solid #ddd;
  border-radius: 12px;
  font-size: 14px;
  box-sizing: border-box;
}

.auth-select-area {
  margin-bottom: 20px;
}

.auth-select-title {
  font-size: 14px;
  font-weight: 500;
  margin-bottom: 12px;
  color: #333;
}

.auth-options {
  display: flex;
  gap: 24px;
  margin-bottom: 16px;
}

.auth-option {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 14px;
  cursor: pointer;
}

.auth-expire {
  margin-top: 16px;
  display: flex;
  align-items: center;
  gap: 12px;
}

.expire-select {
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 12px;
  font-size: 14px;
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