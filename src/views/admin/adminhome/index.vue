<template>
  <div class="layout-col plr20 pt5">
    <t1/>
    <div class="ww100 flex1 flex-bc hidden mt15">
      <c1 class="flex3 hh100 rad8 mr15" />
      <c2 class="flex1 hh100 rad8" />
    </div>
    <div class="ww100 flex1 flex-bc hidden mt15">
      <b1 class="flex3 hh100 rad8 mr15" />
      <b2 class="flex1 hh100 rad8" />
    </div>
  </div>
</template>

<script lang="ts" setup>
  import t1 from './t1'
  import c1 from './c1'
  import c2 from './c2'
  import b1 from './b1'
  import b2 from './b2'
  import scheduled from "@/utils/scheduled"
  const { createScheduled } = scheduled()
  const {proxy}: any = getCurrentInstance()
  const publicStore = proxy.publicStore()
  const configStore = proxy.configStore()

  // 模拟开始------------------Start
  // 统计数据
  const data = { userCount: 12, realTimeLoad: 274, deviceCount: 198, adjustCapacity: 198, onlineRate: 95 }
  // 负荷曲线数据
  const loadChartData = [
    { name: '昨日负荷', data: [ ['00:00', 35, 1767590400000], ['04:00', 50, 1767604800000], ['08:00', 25, 1767619200000], ['12:00', 40, 1767633600000], ['16:00', 65, 1767648000000], ['20:00', 55, 1767662400000], ['24:00', 70, 1767676800000] ] },
    { name: '今日负荷', data: [ ['00:00', 30, 1767590400000], ['04:00', 45, 1767604800000], ['08:00', 20, 1767619200000], ['12:00', 35, 1767633600000], ['16:00', 60, 1767648000000], ['20:00', 50, 1767662400000], ['24:00', 75, 1767676800000] ] }
  ]
  // 用户排名数据
  const rankChartData = [
    {station_name: '用户1', value: '475', rate: '45'},
    {station_name: '用户2', value: '432', rate: '40'},
    {station_name: '用户3', value: '398', rate: '40'},
    {station_name: '用户4', value: '367', rate: '35'},
    {station_name: '用户5', value: '350', rate: '30'},
    {station_name: '用户6', value: '321', rate: '25'},
    {station_name: '用户7', value: '302', rate: '20'},
    {station_name: '用户8', value: '280', rate: '15'},
  ]
  // 模拟结束------------------End

  const state = reactive({
    ...publicStore.$state,
    content: [
      {width: 'w50', show: true, align: 'left', key: '*', name: '序号'},
      {width: 'w50x3', show: true, align: 'left', key: 'station_name', name: '名称', type: 'icon1'},
      {width: 'w50x2', show: true, align: 'left', key: 'devices', name: '设备数量'},
      {width: 'w50x2', show: true, align: 'left', key: 'up_adjustable', name: '最大上调容量'},
      {width: 'w50x2', show: true, align: 'left', key: 'down_adjustable', name: '最大下调容量'},
      {width: 'flex1', show: true, align: 'right', key: {}, name: '操作'},
    ],
    code: 'Total_Power',
  })

  onMounted(async () => {
    // 初始化站点
    if (!configStore.distributId) configStore.distributId = '0'
    // 获取数据
    getInit()
    createScheduled("home_scheduled", 10 * 1000, () => { init() })
  })

  // 切换站点（用户）
  watch(() => configStore.distributId, async(val, old) => {
    await nextTick()
    if(proxy.isNull(val)) return
    getInit()
  }, {immediate: false, deep: true})

  const getInit = async () => {
    // 站点（用户）
    let query1 = {model: 't_station', args: `parent_id='0'`}
    if(configStore.distributId != '0') query1.args += `and id='${configStore.distributId}'`
    // 设备
    let query2 = {model: 't_sensor', args: `class='0'`}
    if(configStore.distributId != '0') query2.args += `and stationNum='${configStore.distributId}'`
    // 请求
    let params = {Api1: query1, Api2: query2}
    let res = await publicStore.http(params)
    let stations = proxy.isNull(res.Api1) ? [] : res.Api1
    publicStore._public.stations = [...stations]
    let devices = proxy.isNull(res.Api2) ? [] : res.Api2
    publicStore._public.devices = [...devices]
    // 模拟数据 注意需要注释
    publicStore.data = {...data}
    publicStore._public.loadChartData = [...loadChartData]
    publicStore._public.rankChartData = [...rankChartData]
    // 组装动态请求
    getQuery()
    // 获取动态数据
    init()
  }

  // 组装动态请求
  const getQuery = () => {
    let queryRedis = {model: "redis"}
    let ids = []
    queryRedis.key = publicStore._public.devices.map((a) => {
      // ids
      let id = ids.find(b=>b == a.id)
      if(!id) ids.push(a.id)
      // redis
      return "mo:Hash:sensor:" + a.id
    })
    state.ids = ids
    state.queryRedis = queryRedis
  }

  // 获取动态数据
  const init = async (key) => {
    // 实时数据
    getRedis()
    // 总负荷曲线
    getLoadChartData()
    // 进行中的交易
    getTransactionData()
    // 市场需求
    getDemandData()
  }

  // 实时数据
  const getRedis = async() => {
    if (!proxy.isNull(publicStore._public.devices)){
      let params = {getApi: state.queryRedis}
      let res = await publicStore.http(params)
      // 平台统计
      let homeInfo = {}
      // 平台设备
      let homeDevices = []
      // 按站统计
      let homeStations = JSON.parse(JSON.stringify(publicStore._public.stations))
      publicStore._public.devices.forEach(v => {
        // 重置实时
        let device =  Object.assign({}, v)
        let redis_exist = res.find(a=>a.id == v.id)
        let redis_data = redis_exist? redis_exist : {}
        Object.assign(device, redis_data)
        // 特殊处理告警
        if(device.isAlarm) device.alarm = device.isAlarm>0? '1' : '0'
        // 站点计算
   			let station = homeStations.find(a=>a.id == v.stationNum)
				if(station) setData(station, redis_data)
        // 数据计算
        setData(homeInfo, redis_data)
        // 获取设备
        homeDevices.push(device)
      })
      publicStore._public.homeStations = homeStations
      publicStore._public.homeInfo = {...homeInfo}
      publicStore._public.homeDevices = [...homeDevices]
      console.log("publicStore._public.homeStations---", publicStore._public.homeStations)
      console.log("publicStore._public.homeInfo---", publicStore._public.homeInfo)
      console.log("publicStore._public.homeDevices---", publicStore._public.homeDevices)
    } else {
      publicStore._public.homeStations = []
      publicStore._public.homeInfo = {}
      publicStore._public.homeDevices = []
    }
    // 统计
    getCount()
    // 排名
    getRank()
  }

  // 数据计算
  const setData = (info, data) => {
    info['sum'] = !info['sum'] ? 1 : info['sum'] + 1
    Object.keys(data).forEach((key)=>{
      if (key.indexOf('_tc')==-1 && (typeof data[key] === 'number' || (typeof data[key] === 'string' && data[key].trim() !== '' && !isNaN(Number(data[key].trim()))))) {
        if(key == 'timestamp'){
          // 最大
          if(!info[key] || info[key] && info[key]<data[key]) info[key] = data[key]
        }else{
          // 累计
          info[key] = !info[key] ? Number(data[key]) : Number(info[key]) + Number(data[key])
        }
        // 告警
        if(key == 'isAlarm') {
          if(data[key] > 0) info['alarm1'] = !info['alarm1'] ? 1 : info['alarm1'] + 1
          if(data[key] == 0) info['alarm0'] = !info['alarm0'] ? 1 : info['alarm0'] + 1
        }
        // 工况 
        if(key == 'offline') {
          info[key+data[key]] = !info[key+data[key]] ? 1 : info[key+data[key]] + 1
        }
        // 逆变器\充电桩\空调
        if(key == 'pv_inverter_status' || key == 'Operation_State' || key == '2022' || key == 'hvac_on_off_status' || key == 'loadStatus' || key == 'fanStatus') {
          info[key+data[key]] = !info[key+data[key]] ? 1 : info[key+data[key]] + 1
        }
      } 
    })
  }

  // 统计
  const getCount = () => {
    let userCount = 0
    let deviceCount = 0
    let realTimeLoad = 0
    let adjustCapacity = 0
    let onlineRate = 0
    // 站点 / 用户数
    userCount = !proxy.isNull(publicStore._public.stations)?publicStore._public.stations.length:0
    if(!proxy.isNull(publicStore._public.homeDevices)){
      // 设备总数
      deviceCount = publicStore._public.homeDevices.length
      // 实时总负荷
      if(publicStore._public.homeDevices.realTimeLoad){
        realTimeLoad = Math.floor((publicStore._public.homeDevices.realTimeLoad*100))/100
      }
      // 当前可调节容量
      if(publicStore._public.homeDevices.adjustCapacity){
        adjustCapacity = Math.floor((publicStore._public.homeDevices.adjustCapacity*100))/100
      }
      // 设备在线率
      if(publicStore._public.homeDevices.offline0){
        onlineRate = Math.floor(((publicStore._public.homeDevices.offline0/publicStore._public.homeDevices.length)*10000))/100
      }
    }
    publicStore.data.userCount = userCount
    publicStore.data.deviceCount = deviceCount
    publicStore.data.realTimeLoad = realTimeLoad
    publicStore.data.adjustCapacity = adjustCapacity
    publicStore.data.onlineRate = onlineRate
  }

  // 排名
  const getRank = () => {
    // 模拟100 实际需改成0
    let totle = publicStore._public.homeInfo[state.code]?publicStore._public.homeInfo[state.code]:100
    let rankChartData = publicStore._public.homeStations.map(v => {
      // temp模拟100内 实际需改成0
      let temp = Math.floor(Math.random() * 100) + 1
      v.value = v[state.code]?v[state.code]:temp
      v.rate = Math.floor(((v.load / totle)*10000))/100
      return v
    })
    publicStore._public.rankChartData = rankChartData.sort((a, b) => b['value'] - a['value'])
  }

  // 总负荷曲线
  const getLoadChartData = async() => {
    // 获取当前时间
    const now = new Date('2025-08-22')
    // 当日开始时间（00:00:00.000）
    const startOfDay = new Date(now.getFullYear(), now.getMonth(), now.getDate()).getTime()
    // 当日最后时间（23:59:59.999）
    const endOfDay = new Date(now.getFullYear(), now.getMonth(), now.getDate(), 23, 59, 59, 999).getTime()
    // 昨日开始时间（昨日 00:00:00.000）
    const startOfYesterday = new Date(now.getFullYear(), now.getMonth(), now.getDate() - 1).getTime()
    // 昨日最后时间（昨日 23:59:59.999）
    const endOfYesterday = new Date(now.getFullYear(), now.getMonth(), now.getDate() - 1, 23, 59, 59, 999).getTime()
    // console.log("startOfDay---", startOfDay)
    // console.log("endOfDay---", endOfDay)
    // console.log("startOfYesterday---", startOfYesterday)
    // console.log("endOfYesterday---", endOfYesterday)
    // 请求
    let query = {model: 't_sensor_measure_float_station', args: `code='${state.code}' and date>'${startOfYesterday}' and date<'${endOfDay}'`}
    if(configStore.distributId != '0') query.args += `and station_num='${configStore.distributId}'`
    let params = {Api: query}
    let res = await publicStore.http(params)
    let todayData = res.filter(a=>{ return a>= startOfDay})
    let yesterday = res.filter(a=>{ return a< startOfDay})
    let loadChartData = [{ name: '昨日负荷', data: [] }, { name: '今日负荷', data: [] }] 
    res.forEach(v => {
      let time = proxy.parseTime(v.date, '{h}:{i}:{s}')
      let item = [time, v.value, v.date]
      if(v.date < startOfDay){
        item[2] = Number(item[2]) + 24*60*60*1000 + ''
        loadChartData[0].data.push(item)
      }else{
        loadChartData[1].data.push(item)
      }
    })
    publicStore._public.loadChartData = [...loadChartData]
  }

  // 进行中的交易
  const getTransactionData = async() => {
    let status = '2'
    let query = {model: 't_trade_demand', args: `status='${status}'`}
    if(configStore.distributId != '0') query.args += `and station_num='${configStore.distributId}'`
    let res = await publicStore.http({Api: query})
    publicStore._public.transactionData = proxy.isNull(res)? [] : res
  }

  // 市场需求
  const getDemandData = async() => {
    let query = {model: 't_trade_demand'}
    // if(configStore.distributId != '0') query.args += `and station_num='${configStore.distributId}'`
    let res = await publicStore.http({Api: query})
    publicStore._public.demandData = proxy.isNull(res)? [] : res
  }
</script>

<style scoped lang="scss">

</style>
