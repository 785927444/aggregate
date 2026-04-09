<template>
  <div class="layout-col plr20 pt5">
    <t1/>
    <div class="layout-row">
      <div class="flex3 hh100">
        <div class="flex-sc flex1 ww100 hidden mb15 mt15">
          <div class="chart-box-wrap">
            <h3 class="card-title-home">
              <span></span>总负荷曲线
            </h3>
            <div ref="loadChartRef" class="chart-box"></div>
          </div>
        </div>
        <div class="flex-sc flex1 ww100 hidden mb15 mt15">
          <div class="chart-box-wrap">
            <h3 class="card-title-home">
              <span></span>进行中的交易
            </h3>
            <div class="chart-box">
              <List @handleClick="handleClick" :state="state"/>
            </div>
          </div>
        </div>
      </div>
      <div class="flex1 hh100">
        <div class="flex-sc flex1 ww100 hidden mb15 mt15 ml15">
          <div class="chart-box-wrap">
            <h3 class="card-title-home card-title-home-r">
              <span></span>用户负荷排名
            </h3>
            <div ref="rankChartRef" class="chart-box"></div>
          </div>
        </div>
        <div class="flex-sc flex1 ww100 hidden mb15 mt15 ml15">
          <div class="chart-box-wrap">
            <h3 class="card-title-home card-title-home-r">
              <span></span>市场需求
            </h3>
            <div class="chart-box">
              <div class="info-card-wrap">
                <div class="info-card">
                  <!-- 头部标签栏 -->
                  <div class="card-header">
                    <span class="tab-label">交易中心</span>
                    <span class="tab-value">JY45234165</span>
                  </div>

                  <!-- 内容区域 -->
                  <div class="card-body">
                    <div class="info-item">
                      <span class="label">需求容量：</span>
                      <span class="value">350000kW</span>
                    </div>
                    <div class="info-item">
                      <span class="label">需求时段：</span>
                      <span class="value">2026-02-01 10:00-12:00</span>
                    </div>
                  </div>
                </div>
              </div>

            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
const {proxy}: any = getCurrentInstance()
const publicStore = proxy.publicStore()
const configStore = proxy.configStore()
import t1 from './t1'
import * as echarts from 'echarts'

let detailRef = $ref()
let values = [
  {name: '否', value: 0},
  {name: '是', value: 1},
]

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
let addRef = $ref()
const state = reactive({
  ...publicStore.$state,
  content: [
    {width: 'w50', show: true, align: 'left', key: '*', name: '序号'},
    {width: 'w50x3', show: true, align: 'left', key: 'station_name', name: '名称', type: 'icon1'},
    {
      width: 'w50x3',
      show: true,
      align: 'left',
      key: 'industry_type',
      name: '行业类型',
      type: 'select',
      list: [],
      value: 'id',
      label: 'name'
    },
    {
      width: 'w50x2',
      show: true,
      align: 'left',
      key: 'day_out_response',
      name: '日前响应',
      type: 'select',
      list: values,
      label: 'name',
      value: 'value'
    },
    {
      width: 'w50x2',
      show: true,
      align: 'left',
      key: 'day_in_response',
      name: '日内响应',
      type: 'select',
      list: values,
      label: 'name',
      value: 'value'
    },
    {
      width: 'w50x2',
      show: true,
      align: 'left',
      key: 'yn_sign',
      name: '是否签约',
      type: 'select',
      list: values,
      label: 'name',
      value: 'value'
    },
    {width: 'w50x2', show: true, align: 'left', key: 'devices', name: '设备数量'},
    {width: 'w50x2', show: true, align: 'left', key: 'up_adjustable', name: '最大上调容量'},
    {width: 'w50x2', show: true, align: 'left', key: 'down_adjustable', name: '最大下调容量'},
    {width: 'flex1', show: true, align: 'right', key: {}, name: '操作'},
  ],
  editFrom: [
    {required: true, editshow: 'none', name: '上级', key: 'parent_id', type: 'input'},
    {required: true, editshow: true, name: '名称', key: 'station_name', type: 'input'},
    // { required: false, editshow: true, name: '关联', key: 'node_id', type: 'select', list: [], label: 'name', value: 'id' },
    {required: false, editshow: true, name: '排序', key: 'order', type: 'input'},
  ],
  addItem: {parent_id: '0'},
})
onMounted(async () => {
  publicStore.init({path: '/station'}, state)
  if (!configStore.distributId) configStore.distributId = '0'
  await setInit()
  getInit()
})
// 负荷曲线数据
const loadChartData = {
  xAxis: ['00:00', '04:00', '08:00', '12:00', '16:00', '20:00', '24:00'],
  series: [
    {name: '昨日负荷', data: [35, 50, 25, 40, 65, 55, 70]},
    {name: '今日负荷', data: [30, 45, 20, 35, 60, 50, 75]}
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
  {
    id: 1,
    tradeNo: 23245,
    responseDate: '2025-10-11',
    responseTime: '10:00-12:00',
    demandType: '填谷',
    applyCap: 43.00,
    winCap: 43.00,
    status: '带出错(市场)'
  },
  {
    id: 2,
    tradeNo: 23245,
    responseDate: '2025-10-11',
    responseTime: '10:00-12:00',
    demandType: '填谷',
    applyCap: 43.00,
    winCap: 43.00,
    status: '待执行'
  },
  {
    id: 3,
    tradeNo: 23245,
    responseDate: '2025-10-11',
    responseTime: '10:00-12:00',
    demandType: '填谷',
    applyCap: 43.00,
    winCap: 43.00,
    status: '正在执行'
  },
  {
    id: 4,
    tradeNo: 23245,
    responseDate: '2025-10-11',
    responseTime: '10:00-12:00',
    demandType: '填谷',
    applyCap: 43.00,
    winCap: 43.00,
    status: '执行完成'
  },
  {
    id: 5,
    tradeNo: 23245,
    responseDate: '2025-10-11',
    responseTime: '10:00-12:00',
    demandType: '填谷',
    applyCap: 43.00,
    winCap: 43.00,
    status: '执行完成'
  },
  {
    id: 6,
    tradeNo: 23245,
    responseDate: '2025-10-11',
    responseTime: '10:00-12:00',
    demandType: '填谷',
    applyCap: 43.00,
    winCap: 43.00,
    status: '执行完成'
  }
])

// 市场需求数据
const demandData = ref([
  {id: 'JY45234165', capacity: 350000, timePeriod: '2026-02-01 10:00-12:00'},
  {id: 'JY123452565', capacity: 350000, timePeriod: '2026-02-01 10:00-12:00'},
  {id: 'JY123452565', capacity: 350000, timePeriod: '2026-02-01 10:00-12:00'}
])

// 🔹 初始化图表
const initCharts = () => {
  // 总负荷曲线
  if (loadChartRef.value) {
    loadChart = echarts.init(loadChartRef.value)
    loadChart.setOption({
      tooltip: {trigger: 'axis'},
      legend: {data: ['昨日负荷', '今日负荷'], top: 0},
      xAxis: {type: 'category', data: loadChartData.xAxis},
      yAxis: {type: 'value', name: 'kW'},
      series: [
        {name: '昨日负荷', type: 'line', data: loadChartData.series[0].data, lineStyle: {color: '#909399'}},
        {name: '今日负荷', type: 'line', data: loadChartData.series[1].data, lineStyle: {color: '#409EFF'}}
      ]
    })
  }

  // 用户负荷排名
  if (rankChartRef.value) {
    rankChart = echarts.init(rankChartRef.value)
    rankChart.setOption({
      tooltip: {trigger: 'axis', axisPointer: {type: 'shadow'}},
      xAxis: {type: 'value', boundaryGap: [0, 0.01]},
      yAxis: {type: 'category', data: rankChartData.yAxis},
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
const handleClick = async (remark, val) => {
  if (remark == 'add' || remark == 'upd') {
    addRef.onVisable(val)
  }
  if (remark == 'del') {
    addRef.del(val)
  }
  if (remark == 'detail') {
    detailRef.onVisable(val)
  }
  if (remark == 'active') {
    publicStore.active = val
  }
}
const setInit = async () => {
  // 获取行业
  let query1 = {model: 't_industry', args: `parent_id='0'`}
  publicStore.http({Api: query1}).then(res => {
    state.industrys = proxy.isNull(res) ? [] : res.sort((a, b) => a.order - b.order)
    state.content.forEach(v => {
      if (v.key == 'industry_type') v.list = [...state.industrys]
    })
  })
  // 获取节点
  // let query2 = {model: 't_sensor', args: `class='3'`}
  // publicStore.http({Api: query2}).then(res=>{
  //   state.nodes = proxy.isNull(res)? [] : res
  //   state.nodes = state.nodes.map(node => ({ ...node, id: String(node.id) }))
  //   state.editFrom.forEach(v => {
  //     if(v.key == 'node_id') v.list = [...state.nodes]
  //   })
  // })
}

const getInit = async () => {
  let list = []
  if (!proxy.isNull(configStore.stations)) {
    list = configStore.stations
  } else {
    state.query = {model: state.model, args: `parent_id='0'`}
    state.params = {Api: state.query}
    let res = await publicStore.http(state.params)
    console.log('resresres', res)
    list = proxy.isNull(res) ? [] : res.sort((a, b) => a.order - b.order)
  }
  state.empty = proxy.isNull(list) ? true : false
  state.list = list
  publicStore._public.list = state.list
  configStore.stations = state.list
  getNewList()
}
const init = async (key) => {
  state.query = {model: state.model, args: `parent_id='0'`}
  state.params = {Api: state.query}
  let res = await publicStore.http(state.params)
  let list = proxy.isNull(res) ? [] : res.sort((a, b) => a.order - b.order)
  console.log('resresres', list)
  state.empty = proxy.isNull(list) ? true : false
  state.list = list
  publicStore._public.list = state.list
  configStore.stations = state.list
  getNewList()
}

const getNewList = () => {
  state.list.forEach(v => {
    v.area = v.province_name && v.city_name ? `${v.province_name}-${v.city_name}` : ''
    let query = {model: 't_sensor', args: `stationNum='${v.id}' and class='0'`, field: `COUNT(*)`}
    publicStore.http({Api: query}).then(ress => {
      v.devices = proxy.isNull(ress) ? 0 : ress[0]['COUNT(*)']
    })
  })
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

.chart-box-wrap {
  width: 100%;
  border-radius: 8px;
  background-color: #fff;
}

.card-title-home {
  width: 98%;
  margin: 0 auto;
  padding: 5px 0px;
  font-size: 14px;
  font-weight: 500;
}

.card-title-home-r{
  width: 85%;
  margin: 0 0 0 15px;
}

.card-title-home span {
  display: block;
  margin-right: 6px;
  float: left;
  width: 4px;
  height: 18px;
  border-radius: 133px;
  background: linear-gradient(0deg, #1B76FF 0%, rgba(77, 131, 238, 0) 99%);

}

.chart-box {
  width: 100%;
  height: 293px; /* 可根据需求调整高度 */
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

.user-icon {
  background-image: url(../../../assets/images/jh/user.png);
}

.load-icon {
  background-image: url(../../../assets/images/jh/load.png);
}

.device-icon {
  background-image: url(../../../assets/images/jh/device.png);
}

.capacity-icon {
  background-image: url(../../../assets/images/jh/capacity.png);
}

.online-icon {
  background-image: url(../../../assets/images/jh/online.png);
}

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

.info-card-wrap {
  width: 85%;
  margin: 0 0 0 15px;
}

.info-card {
  /* 自动布局子元素 */
  height: 106px;
  background: url(../../../assets/images/home/bg-home-r2.png) left top no-repeat;
  background-size:  100% auto;
  padding: 8px 16px;
}

/* 头部样式 */
.card-header {
  display: flex;
  align-items: center;
  padding: 8px 16px;
}

.tab-label {
  font-size: 14px;
  color: #6b7280; /* 灰色文字 */
  font-weight: 500;
}

.tab-value {
  font-size: 14px;
  color: #1f2937; /* 深灰色文字 */
  font-weight: 600;
  margin-left: 4px;
}

/* 内容区域样式 */
.card-body {
  padding: 12px 16px;
}

.info-item {
  display: flex;
  align-items: center;
  margin-bottom: 8px; /* 项目之间的间距 */
}

.info-item:last-child {
  margin-bottom: 0;
}

.label {
  font-size: 14px;
  color: #6b7280;
}

.value {
  font-size: 14px;
  color: #1f2937;
  font-weight: 500;
}
</style>
