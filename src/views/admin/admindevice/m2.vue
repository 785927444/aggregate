<template>
  <div class="layout-col">
    <!-- 标题 -->
    <aa-title :title="isNull(publicStore.active)?'':publicStore.active.station_name">
      <template #left-content></template>
      <template #right-content>
        <div class="rad4 ptb5 plr8 flex-cc cursor bgi1 white ml10" 
          v-if="state.content.find(v=>{return typeof(v.key) == 'object'}) && state.content.find(v=>{return typeof(v.key) == 'object'})['key'].hasOwnProperty('add')" 
          @click.stop="handleClick('add', state.addItem)">
          <i-ep-plus class="f12 fw" /><span class="f14 ml4">{{state.content.find(v=>{return typeof(v.key) == 'object'})['key'].add}}</span>
        </div>
      </template>
    </aa-title>
    <!-- 内容 -->
    <div class="layout-col white-rgba50 rad8 p15">
      <List @handleClick="handleClick" :state="state"/>
    </div>
    <!-- 底部 -->
    <aa-foot></aa-foot>
    <AddDevice @init="init" :state="state" ref="addRef" />
    <Password @init="init" :state="state" ref="passwordRef" />
  </div>
</template>

<script lang="ts" setup>
  import scheduled from "@/utils/scheduled";
  const { createScheduled } = scheduled();
	const { proxy }:any = getCurrentInstance()
  const publicStore = proxy.publicStore()
  const configStore = proxy.configStore()
  const dictStore = proxy.dictStore()
  let addRef = $ref()
  let importRef = $ref()
  const offlines = [
    {name: '', value: '未知'},
    {name: '在线', value: '0', color: 'i1'},
    {name: '离线', value: '1', color: 'i1'},
  ]
  const runs = [
    {name: '', value: '未知'},
    {name: '制冷', value: '0', color: 'i1'},
    {name: '制热', value: '1', color: 'i1'},
  ]
  const adrTypes = [
    {name: '否', value: 2},
    {name: '是', value: 1},
  ]
  const loadTypes = [
    {name: '可调节', value: 1},
    {name: '不可调节', value: 2},
  ]
  const responseTypes = [
    {name: '人工响应', value: 1},
    {name: '自动响应', value: 2},
  ]
  const state = reactive({
	  ...publicStore.$state,
    content: [
      { width: 'w70',   show: true, align: 'center', key: '*', name: '序号' },
      // { width: 'w50x3', show: true, align: 'left', key: 'id', name: 'ID', type: 'icon5' },  
      { width: 'w50x5', show: true, align: 'left', key: 'name', name: '名称' },
      { width: 'w50x3', show: true, align: 'left', key: 'type', name: '类型', type: 'select', list: [], value: 'type', label: 'name' },
      { width: 'w50x3', show: true, align: 'left', key: 'snr', name: '调节方式', type: 'select', list: loadTypes, value: 'value', label: 'name' },
      { width: 'w50x3', show: true, align: 'left', key: 'response_type', name: '响应方式', type: 'select', list: responseTypes, value: 'value', label: 'name' },
      { width: 'w50x3', show: true, align: 'left', key: 'offline', name: '在线状态', type: 'select', list: offlines, value: 'value', label: 'name' },
      { width: 'w50x3', show: true, align: 'left', key: 'run', name: '运行状态', type: 'select', list: runs, value: 'value', label: 'name' },
      { width: 'w50x3', show: true, align: 'left', key: 'temperature', name: '当前设定温度' },
      { width: 'flex1', show: true, align: 'right', key: {}, name: '操作' },
    ],
    editFrom: [
      { required: true, editshow: 'disabled',  name: 'ID', key: 'id',  type: 'input', class: 'number' },
      { required: true,  editshow: true, name: '名称', key: 'name', type: 'input' },
      { required: true,  editshow: true,  name: '节点', key: 'parent_id', type: 'select', list: [], value: 'id', label: 'name'},
      { required: true,  editshow: true,  name: '类型', key: 'type', type: 'select', list: [], value: 'type', label: 'name', relation: ['comm_params'] },
      { required: false, editshow: true, name: '区域', key: 'bayNum', type: 'select', list: [], value: 'id', label: 'station_name'},
      { required: false, editshow: true, name: '安装位置', key: 'instlPos',  type: 'input' },
      { required: false, editshow: true, name: 'ADR设备', key: 'adr_type', type: 'select', list: adrTypes, value: 'value', label: 'name' },
      { required: false, editshow: true, name: '调节方式', key: 'snr', type: 'select', list: loadTypes, value: 'value', label: 'name' },
      { required: false, editshow: true, name: '响应方式', key: 'response_type', type: 'select', list: responseTypes, value: 'value', label: 'name' },
      { required: false, editshow: true, name: '厂商型号', key: 'model', type: 'select', list: [], value: 'id', label: 'name', filt: ['type', 'type'] },
      { required: false, editshow: true, name: '供电局资产号',key: 'asset_num',  type: 'input' },
      { required: false, editshow: true, name: '额定功率', key: 'scale', type: 'input' },
      { required: false, editshow: true, name: '输出功率', key: 'power', type: 'input' },
      { required: false, editshow: true, name: '额定电压', key: 'voltage', type: 'input' },
      { required: false, editshow: true, name: '日前削峰能力', key: 'day_peak', type: 'input' },
      { required: false, editshow: true, name: '日前填谷能力', key: 'day_grain', type: 'input' },
      { required: false, editshow: true, name: '向上调节容量', key: 'up_adjust', type: 'input' },
      { required: false, editshow: true, name: '向下调节容量', key: 'down_adjust', type: 'input' },
      { required: false, editshow: true, name: '向上调节速率', key: 'up_adjust_speed', type: 'input' },
      { required: false, editshow: true, name: '向下调节速率', key: 'down_adjust_speed', type: 'input' },
      { required: false, editshow: true, name: '通信参数', key: 'comm_params',  type: 'json' },
    ],
    addItem: {},
    imgs: [],
    authextra: ['sensorcontrol', 'sensoralarm', 'sensorset'],
  })

  onMounted(async() => {
    await publicStore.init({path: '/sensor'}, state)
    createScheduled("device_control", 5 * 1000, () => {
      getDevicesData()
    });
  })

  watch(() => publicStore.active, async(val) => {
    await nextTick()
    if(proxy.isNull(val)) return
    init()
  }, {immediate: false, deep: true})

  const init = async(key) => {
    if(key) publicStore.isTrue = !publicStore.isTrue
    state.query = {model: state.model, args: `stationNum='${publicStore.active.id}' and class='0'`}
    state.params = {Api: state.query}
    let res = await publicStore.http(state.params)
    state.empty = proxy.isNull(res)? true : false
    state.list = proxy.isNull(res)? [] : res.sort((a, b) => a.order - b.order)
    state.list.forEach(v => {
      if(!proxy.isNull(publicStore._public.models)){
        let exist = publicStore._public.models.find(a=>a.id == v.model)
        v.picture = exist? exist.picture : ''
      } else {
        v.picture = ''
      }
    })
    setInit()
    getDevicesData()
  }
  
  const setInit = () => {
    state.query = {model: "redis"}
    state.query.key = state.list.map((a) => { return "mo:Hash:sensor:" + a.id })
    state.addItem = {
      stationNum: publicStore.active.id, 
      class: '0',
    }
    // 获取间隔
    let query_bay = {model: "t_station", args: `parent_id='${publicStore.active.id}'`, field: "id, parent_id, station_name, `order`"}
    publicStore.http({Api: query_bay}).then(res=>{
      state.bays = proxy.isNull(res)? [] : res
      state.editFrom.forEach(v => {
        if(v.key == 'bayNum') v.list = [...state.bays]
      })
      state.content.forEach(v => {
        if(v.key == 'bayNum') v.list = [...state.bays]
      })
    })
    // 获取节点
    let query_node = {model: 't_sensor', args: `stationNum='${publicStore.active.id}' and class='3' and id>0`}
    publicStore.http({Api: query_node}).then(res=>{
      state.nodes = proxy.isNull(res)? [] : res
      state.nodes = state.nodes.map(node => ({ ...node, id: String(node.id) }))
      publicStore._public.nodes = state.nodes
      state.editFrom.forEach(v => {
        if(v.key == 'parent_id') v.list = [...state.nodes]
      })
    })
    state.content.forEach(v => {
      if(v.key == 'model') v.list = publicStore._public.models
      if(v.key == 'type') v.list = publicStore._public.templates
      if(v.key == 'parent_type') v.list = publicStore._public.qzlist
    })
    state.editFrom.forEach(v => {
      if(v.key == 'model') v.list = publicStore._public.models
      if(v.key == 'type') v.list = publicStore._public.templates
      if(v.key == 'parent_type') v.list = publicStore._public.qzlist
    })
  }

  // 实时请求设置信息
  const getDevicesData = async() => {
    if (!proxy.isNull(state.list)){
      let params = {getApi: state.query}
      let res = await publicStore.http(params)
      state.list.forEach(vv => {
        // 重置实时
        let device =  Object.assign({}, vv)
        let redis_exist = res.find(a=>a.sensor_id == vv.id)
        let redis_data = redis_exist? redis_exist : {}
        Object.assign(device, redis_data)
        if(device.isAlarm) device.alarm = device.isAlarm>0? '1' : '0'
      })
    }
    console.log("state.list---", state.list)
  }

  const handleClick = async(remark, val) => {
    if(remark == 'add'){
      addRef.onVisable(val)
    }
    if(remark == 'upd'){
      addRef.onVisable(val)
    }
    if(remark == 'del'){
      addRef.del(val)
    }
  }

</script>
  
<style scoped lang="scss">

</style>
  