<template>
  <div class="health-manage">
    <!-- 提醒卡片 -->
    <div class="reminder-card">
      <div class="reminder-title">💉 疫苗/驱虫提醒</div>
      <div class="reminder-list">
        <div class="reminder-item warning">🔔 狂犬疫苗：剩余7天 ⚠️</div>
        <div class="reminder-item danger">🔔 体内驱虫：已过期 ⚠️⚠️</div>
      </div>
      <button class="appoint-btn" @click="goToAppointment">📅 一键预约复诊</button>
    </div>
    
    <!-- 体重趋势 + 医疗记录 左右布局 -->
    <div class="two-columns">
      <!-- 左侧：体重趋势 -->
      <div class="weight-card">
        <h4>📈 体重趋势（近30天）</h4>
        <!-- ECharts 图表容器 -->
        <div id="weightChart" class="chart-container"></div>
        <div class="ai-warning" v-if="weightTrend">
          ⚠️ AI预警：{{ weightTrend }}
        </div>
        <button class="add-weight" @click="openAddWeightModal">+ 手动添加体重</button>
      </div>
      
      <!-- 右侧：医疗记录 -->
      <div class="medical-card">
        <h4>📋 医疗记录（可追溯）</h4>
        <div class="medical-list">
          <div class="medical-item" v-for="record in recentMedicalRecords" :key="record.id">
            <div class="medical-date">{{ record.date }}</div>
            <div class="medical-info">{{ record.info }}</div>
          </div>
        </div>
        <a href="#" class="more-link" @click.prevent="goToAllMedicalRecords">查看全部医疗记录 &gt;</a>
      </div>
    </div>
    
    <!-- 医疗云档案共享摘要 -->
    <div class="share-card">
      <h4>🔗 医疗云档案共享摘要</h4>
      <div class="share-info">
        已授权医院：安心宠物医院（病历可查看）
      </div>
      <div class="share-info">
        最近共享：2026-03-01 疫苗记录被查看
      </div>
    </div>

    <!-- 添加体重弹窗 -->
    <div v-if="showAddWeightModal" class="modal" @click.self="closeAddWeightModal">
      <div class="modal-content">
        <h3>📊 添加体重记录</h3>
        <select v-model="weightForm.petId" class="modal-select">
          <option value="1">豆豆（金毛）</option>
          <option value="2">咪咪（英短）</option>
        </select>
        <input type="number" v-model="weightForm.weight" placeholder="体重（kg）" class="modal-input" step="0.1">
        <input type="date" v-model="weightForm.date" class="modal-input">
        <input type="text" v-model="weightForm.remark" placeholder="备注（选填）" class="modal-input">
        <div class="modal-actions">
          <button class="btn-cancel" @click="closeAddWeightModal">取消</button>
          <button class="btn-submit" @click="saveWeight">保存</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import { useRouter } from 'vue-router'
import * as echarts from 'echarts'

const router = useRouter()

// 体重数据（从 localStorage 读取）
const weightRecords = ref([])

// 当前选中的宠物
const selectedPetId = ref(1)

// 计算体重趋势预警
const weightTrend = ref('')

// 最近医疗记录（假数据）
const recentMedicalRecords = ref([
  { id: 1, date: '2026-03-01', info: '疫苗第三针 · 安心医院 · 李医生' },
  { id: 2, date: '2026-02-10', info: '年度体检 · 报告：各项正常' },
  { id: 3, date: '2025-12-20', info: '皮肤病就诊 · 处方：药膏+口服药' }
])

// 加载体重数据
const loadWeightData = () => {
  const stored = localStorage.getItem('weightRecords')
  if (stored) {
    weightRecords.value = JSON.parse(stored)
  } else {
    // 初始化示例数据
    weightRecords.value = [
      { id: 1, petId: 1, weight: 28.5, date: '2026-04-18', remark: '' },
      { id: 2, petId: 1, weight: 28.2, date: '2026-04-21', remark: '' },
      { id: 3, petId: 1, weight: 28.0, date: '2026-04-25', remark: '' },
      { id: 4, petId: 1, weight: 27.5, date: '2026-04-28', remark: '' },
      { id: 5, petId: 1, weight: 27.2, date: '2026-05-02', remark: '' },
      { id: 6, petId: 1, weight: 27.0, date: '2026-05-06', remark: '' },
      { id: 7, petId: 1, weight: 26.8, date: '2026-05-10', remark: '' },
      { id: 8, petId: 2, weight: 4.5, date: '2026-04-20', remark: '' },
      { id: 9, petId: 2, weight: 4.6, date: '2026-04-27', remark: '' },
      { id: 10, petId: 2, weight: 4.5, date: '2026-05-04', remark: '' },
      { id: 11, petId: 2, weight: 4.4, date: '2026-05-11', remark: '' }
    ]
    localStorage.setItem('weightRecords', JSON.stringify(weightRecords.value))
  }
}

// 保存体重数据
const saveWeightData = () => {
  localStorage.setItem('weightRecords', JSON.stringify(weightRecords.value))
}

// 获取当前宠物的体重数据（近30天）
const getPetWeightData = () => {
  const now = new Date()
  const thirtyDaysAgo = new Date()
  thirtyDaysAgo.setDate(now.getDate() - 30)
  
  const records = weightRecords.value
    .filter(r => r.petId === selectedPetId.value && new Date(r.date) >= thirtyDaysAgo)
    .sort((a, b) => new Date(a.date) - new Date(b.date))
  
  return {
    dates: records.map(r => r.date.slice(5)),
    weights: records.map(r => r.weight)
  }
}

// 计算趋势预警
const calculateTrend = () => {
  const data = getPetWeightData()
  if (data.weights.length < 3) {
    weightTrend.value = '数据不足，无法分析趋势'
    return
  }
  const latest = data.weights[data.weights.length - 1]
  const earliest = data.weights[0]
  const change = ((latest - earliest) / earliest * 100).toFixed(1)
  
  if (change < -5) {
    weightTrend.value = `近30天体重下降${Math.abs(change)}%，建议关注饮食状况`
  } else if (change > 5) {
    weightTrend.value = `近30天体重上升${change}%，建议增加运动量`
  } else {
    weightTrend.value = `近30天体重变化${change}%，状态稳定`
  }
}

// 渲染图表
let chart = null
const renderChart = () => {
  const data = getPetWeightData()
  const chartDom = document.getElementById('weightChart')
  if (!chartDom) return
  
  if (chart) {
    chart.dispose()
  }
  chart = echarts.init(chartDom)
  
  const option = {
    tooltip: { trigger: 'axis' },
    xAxis: {
      type: 'category',
      data: data.dates,
      name: '日期',
      axisLabel: { rotate: 45 }
    },
    yAxis: {
      type: 'value',
      name: '体重 (kg)',
      min: function(value) {
        return Math.floor(value.min - 1)
      }
    },
    series: [{
      data: data.weights,
      type: 'line',
      smooth: true,
      lineStyle: { color: '#2ecc71', width: 3 },
      areaStyle: { opacity: 0.1, color: '#2ecc71' },
      symbol: 'circle',
      symbolSize: 8,
      itemStyle: { color: '#2ecc71' }
    }],
    grid: { containLabel: true, left: 50, right: 20, top: 30, bottom: 30 }
  }
  
  chart.setOption(option)
  calculateTrend()
}

// 添加体重
const showAddWeightModal = ref(false)
const weightForm = ref({
  petId: 1,
  weight: '',
  date: new Date().toISOString().slice(0, 10),
  remark: ''
})

const openAddWeightModal = () => {
  showAddWeightModal.value = true
}

const closeAddWeightModal = () => {
  showAddWeightModal.value = false
  weightForm.value = {
    petId: 1,
    weight: '',
    date: new Date().toISOString().slice(0, 10),
    remark: ''
  }
}

const saveWeight = () => {
  if (!weightForm.value.weight) {
    alert('请填写体重')
    return
  }
  
  const newRecord = {
    id: Date.now(),
    petId: weightForm.value.petId,
    weight: parseFloat(weightForm.value.weight),
    date: weightForm.value.date,
    remark: weightForm.value.remark
  }
  
  weightRecords.value.push(newRecord)
  saveWeightData()
  renderChart()
  
  const petName = weightForm.value.petId === 1 ? '豆豆' : '咪咪'
  alert(`已为「${petName}」添加体重记录：${weightForm.value.weight}kg`)
  closeAddWeightModal()
}

// 跳转函数
const goToAppointment = () => {
  router.push('/service-hall?tab=merchant')
}

const goToAllMedicalRecords = () => {
  router.push('/medical-records')
}

// 监听宠物切换（如果需要）
// 监听窗口大小变化
window.addEventListener('resize', () => {
  if (chart) chart.resize()
})

onMounted(() => {
  loadWeightData()
  renderChart()
})
</script>

<style scoped>
/* 样式保持不变，添加图表容器样式 */
.chart-container {
  width: 100%;
  height: 300px;
  margin-bottom: 16px;
}

/* 其他样式同之前... */
.health-manage { max-width: 1200px; }
.reminder-card { background: linear-gradient(135deg, #fff3e0, #ffe0b2); border-radius: 16px; padding: 20px 24px; margin-bottom: 24px; }
.reminder-title { font-size: 18px; font-weight: 600; margin-bottom: 12px; }
.reminder-list { margin-bottom: 16px; }
.reminder-item { padding: 8px 0; font-size: 14px; }
.reminder-item.warning { color: #ff9800; }
.reminder-item.danger { color: #f44336; }
.appoint-btn { background: #2ecc71; color: white; border: none; padding: 10px 20px; border-radius: 24px; cursor: pointer; font-weight: 500; }
.two-columns { display: grid; grid-template-columns: 1fr 1fr; gap: 24px; margin-bottom: 24px; }
.weight-card, .medical-card, .share-card { background: white; border-radius: 16px; padding: 20px 24px; box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05); }
.weight-card h4, .medical-card h4, .share-card h4 { font-size: 18px; color: #1a3c34; margin-bottom: 16px; }
.ai-warning { background: #fff3e0; padding: 12px; border-radius: 8px; font-size: 13px; color: #ff9800; margin-bottom: 12px; }
.add-weight { background: none; border: 1px solid #2ecc71; color: #2ecc71; padding: 8px 16px; border-radius: 20px; cursor: pointer; }
.medical-list { margin-bottom: 16px; }
.medical-item { padding: 10px 0; border-bottom: 1px solid #f0f0f0; display: flex; gap: 16px; }
.medical-date { width: 100px; color: #999; font-size: 13px; }
.medical-info { flex: 1; font-size: 14px; color: #555; }
.share-info { padding: 8px 0; font-size: 14px; color: #555; }
.more-link { color: #2ecc71; text-decoration: none; font-size: 13px; cursor: pointer; }

/* 模态框 */
.modal { position: fixed; top: 0; left: 0; right: 0; bottom: 0; background: rgba(0, 0, 0, 0.5); display: flex; align-items: center; justify-content: center; z-index: 1000; }
.modal-content { background: white; border-radius: 20px; padding: 24px; width: 400px; max-width: 90%; }
.modal-content h3 { margin-bottom: 20px; color: #1a3c34; }
.modal-input, .modal-select { width: 100%; padding: 12px; margin-bottom: 16px; border: 1px solid #ddd; border-radius: 12px; font-size: 14px; box-sizing: border-box; }
.modal-actions { display: flex; gap: 12px; justify-content: flex-end; margin-top: 8px; }
.btn-cancel { background: #f0f0f0; border: none; padding: 10px 20px; border-radius: 24px; cursor: pointer; }
.btn-submit { background: #2ecc71; color: white; border: none; padding: 10px 20px; border-radius: 24px; cursor: pointer; }
</style>