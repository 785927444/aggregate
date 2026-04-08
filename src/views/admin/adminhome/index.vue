<template>
  <div class="layout-col plr20 pt5">
    <t1 />
    <div class="layout-row">
      <div class="flex3 hh100">
        11
      </div>
      <div class="flex1 hh100">
        22
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import t1 from './t1'
import * as echarts from 'echarts'

// 🔹 模拟数据
const statsData = ref({
  userCount: 12,
  realTimeLoad: 274,
  deviceCount: 198,
  adjustCapacity: 198,
  onlineRate: 95
})

const loadChartRef = ref(null)
const rankChartRef = ref(null)
let loadChart = null
let rankChart = null

// 负荷曲线数据
const loadChartData = {
  xAxis: ['00:00', '04:00', '08:00', '12:00', '16:00', '20:00', '24:00'],
  series: [
    { name: '昨日负荷', data: [35, 50, 25, 40, 65, 55, 70] },
    { name: '今日负荷', data: [30, 45, 20, 35, 60, 50, 75] }
  ]
}

// 用户排名数据
const rankChartData = {
  xAxis: ['302kW', '321kW', '350kW', '367kW', '398kW', '432kW', '457kW'],
  yAxis: ['用户7', '用户6', '用户5', '用户4', '用户3', '用户2', '用户1'],
  data: [302, 321, 350, 367, 398, 432, 457],
  colors: ['#8392A5', '#8392A5', '#8392A5', '#409EFF', '#E6A23C', '#E6A23C', '#F56C6C']
}

// 交易表格数据
const transactionData = ref([
  { id: 1, tradeNo: 23245, responseDate: '2025-10-11', responseTime: '10:00-12:00', demandType: '填谷', applyCap: 43.00, winCap: 43.00, status: '带出错(市场)' },
  { id: 2, tradeNo: 23245, responseDate: '2025-10-11', responseTime: '10:00-12:00', demandType: '填谷', applyCap: 43.00, winCap: 43.00, status: '待执行' },
  { id: 3, tradeNo: 23245, responseDate: '2025-10-11', responseTime: '10:00-12:00', demandType: '填谷', applyCap: 43.00, winCap: 43.00, status: '正在执行' },
  { id: 4, tradeNo: 23245, responseDate: '2025-10-11', responseTime: '10:00-12:00', demandType: '填谷', applyCap: 43.00, winCap: 43.00, status: '执行完成' },
  { id: 5, tradeNo: 23245, responseDate: '2025-10-11', responseTime: '10:00-12:00', demandType: '填谷', applyCap: 43.00, winCap: 43.00, status: '执行完成' },
  { id: 6, tradeNo: 23245, responseDate: '2025-10-11', responseTime: '10:00-12:00', demandType: '填谷', applyCap: 43.00, winCap: 43.00, status: '执行完成' }
])

// 市场需求数据
const demandData = ref([
  { id: 'JY45234165', capacity: 350000, timePeriod: '2026-02-01 10:00-12:00' },
  { id: 'JY123452565', capacity: 350000, timePeriod: '2026-02-01 10:00-12:00' },
  { id: 'JY123452565', capacity: 350000, timePeriod: '2026-02-01 10:00-12:00' }
])

// 🔹 初始化图表
const initCharts = () => {
  // 总负荷曲线
  if (loadChartRef.value) {
    loadChart = echarts.init(loadChartRef.value)
    loadChart.setOption({
      tooltip: { trigger: 'axis' },
      legend: { data: ['昨日负荷', '今日负荷'], top: 0 },
      xAxis: { type: 'category', data: loadChartData.xAxis },
      yAxis: { type: 'value', name: 'kW' },
      series: [
        { name: '昨日负荷', type: 'line', data: loadChartData.series[0].data, lineStyle: { color: '#909399' } },
        { name: '今日负荷', type: 'line', data: loadChartData.series[1].data, lineStyle: { color: '#409EFF' } }
      ]
    })
  }

  // 用户负荷排名
  if (rankChartRef.value) {
    rankChart = echarts.init(rankChartRef.value)
    rankChart.setOption({
      tooltip: { trigger: 'axis', axisPointer: { type: 'shadow' } },
      xAxis: { type: 'value', boundaryGap: [0, 0.01] },
      yAxis: { type: 'category', data: rankChartData.yAxis },
      series: [
        {
          name: '负荷',
          type: 'bar',
          data: rankChartData.data,
          itemStyle: {
            color: (params) => rankChartData.colors[params.dataIndex]
          }
        }
      ]
    })
  }
}

// 🔹 自适应窗口
const resizeCharts = () => {
  loadChart?.resize()
  rankChart?.resize()
}

onMounted(() => {
  initCharts() // 取消注释，初始化图表
  window.addEventListener('resize', resizeCharts)
})

onUnmounted(() => {
  window.removeEventListener('resize', resizeCharts)
  loadChart?.dispose()
  rankChart?.dispose()
})
</script>

<style scoped>
.dashboard-container {
  padding: 20px;
  background: #f5f7fa;
  min-height: 100vh;
}

.stats-row {
  display: flex;
  gap: 20px;
  margin-bottom: 0px;
  flex-wrap: wrap;
  height: 100px;
}

.stat-col {
  flex: 1;
  min-width: 160px;
}

.stat-card {
  background: #fff;
  border-radius: 8px;
  padding: 15px;
  display: flex;
  align-items: center;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
}

/* 图标统一尺寸 */
.stat-icon {
  width: 40px;
  height: 40px;
  margin-right: 20px;
  background-size: contain;
  background-repeat: no-repeat;
}

.user-icon { background-image: url(../../../assets/images/jh/user.png); }
.load-icon { background-image: url(../../../assets/images/jh/load.png); }
.device-icon { background-image: url(../../../assets/images/jh/device.png); }
.capacity-icon { background-image: url(../../../assets/images/jh/capacity.png); }
.online-icon { background-image: url(../../../assets/images/jh/online.png); }

.stat-info {
  flex: 1;
}

.stat-title {
  font-size: 14px;
  color: #666;
  margin-bottom: 4px;
}

.stat-value {
  font-size: 20px;
  font-weight: bold;
  color: #333;
}

.stat-suffix {
  font-size: 12px;
  font-weight: normal;
  color: #999;
  margin-left: 2px;
}

/* 图表与表格布局 */
.content-row {
  display: flex;
  gap: 20px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.chart-col {
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  padding: 15px;
}

.chart-col.large {
  flex: 2;
  min-width: 400px;
}

.chart-col.medium {
  flex: 1;
  min-width: 300px;
}

.table-col {
  background: #fff;
  border-radius: 8px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
  padding: 15px;
}

.table-col.large {
  flex: 2;
  min-width: 500px;
}

.table-col.medium {
  flex: 1;
  min-width: 300px;
}

.chart-title, .card-title {
  font-size: 16px;
  font-weight: bold;
  margin-bottom: 15px;
  color: #333;
}

.chart-container {
  width: 100%;
  height: 250px;
}

.data-card {
  height: 100%;
}

.more {
  font-size: 14px;
  color: #409EFF;
  float: right;
  cursor: pointer;
}

.demand-list {
  margin-top: 10px;
}

.demand-item {
  padding: 10px 0;
  border-bottom: 1px solid #f0f0f0;
}

.demand-item:last-child {
  border-bottom: none;
}

.demand-id {
  font-weight: bold;
  margin-bottom: 4px;
}

.demand-cap, .demand-time {
  font-size: 14px;
  color: #666;
}
</style>