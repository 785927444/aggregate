<template>
  <div class="layout-col">
    <!-- 标题 -->
    <aa-title title="前置">
      <template #left-content></template>
      <template #right-content>
        <div class="rad4 ptb5 plr8 flex-cc cursor bgi1 white ml10" 
          v-if="state.content.find(v=>{return typeof(v.key) == 'object'}) && state.content.find(v=>{return typeof(v.key) == 'object'})['key'].hasOwnProperty('add')" 
          @click.stop="addRef.onVisable(state.addItem)">
          <i-ep-plus class="f12 fw" /><span class="f14 ml2">{{state.content.find(v=>{return typeof(v.key) == 'object'})['key'].add}}</span>
        </div>
      </template>
    </aa-title>
    <!-- 内容 -->
    <div class="layout-col white-rgba50 rad8 p15">
      <List @handleClick="handleClick" :state="state"/>
    </div>
    <!-- 底部 -->
    <aa-foot></aa-foot>
    <Add @init="init" :state="state" ref="addRef" />
  </div>
</template>

<script lang="ts" setup>
	const { proxy }:any = getCurrentInstance()
  const publicStore = proxy.publicStore()
  const configStore = proxy.configStore()
  let addRef = $ref()
  const state = reactive({
	  ...publicStore.$state,
    content: [
      { width: 'w70',   show: true, align: 'center', key: '*', name: '序号' },
      { width: 'w50x5', show: true, align: 'left', key: 'id', name: 'ID', type: 'icon5' },  
      { width: 'w50x5', show: true, align: 'left', key: 'name', name: '名称' },
      { width: 'w50x5', show: true, align: 'left', key: 'type', name: '类型', list: [], value: 'type', label: 'name' },
      { width: 'w50x4', show: true, align: 'left', key: 'localSvcIP', name: 'IP/串口号' }, 
      { width: 'w50x4', show: true, align: 'left', key: 'localSvcPort', name: '端口/波特率' }, 
      { width: 'w50x2', show: true, align: 'left', key: 'order', name: '排序' }, 
      { width: 'flex1', show: true, align: 'right', key: {},  name: '操作' },
    ],
    editFrom: [],
    addItem: {},
    model: 't_sensor',
    authextra: ['sensorcontrol', 'sensoralarm', 'sensorset'],
  })

  onMounted(async() => {
    publicStore.init({path: '/sensor'}, state)
    if(!configStore.distributId) configStore.distributId = '0'
    let query = {model: "t_sensor_template", args: `attribute='2'`}
    let res = await publicStore.http({Api: query})
    publicStore._public.templates = proxy.isNull(res)? [] : res
    init()
  })

  watch(() => configStore.distributId, async(val, old) => {
    await nextTick()
    if(proxy.isNull(val)) return
    init()
  }, {immediate: false, deep: true})

  const init = async(key) => {
    let args = !configStore.distributId || configStore.distributId == '0'? `class=2` : `stationNum='${configStore.distributId}' and class=2`
    args = configStore.user&&configStore.user.role_id=='1'?args : args + ' and id > 0'
    state.query = {model: state.model, args: args}
    state.params = {Api: state.query}
    let res = await publicStore.http(state.params)
    state.empty = proxy.isNull(res)? true : false
    state.list = proxy.isNull(res)? [] : res.sort((a, b) => a.order - b.order)
    publicStore._public.list = state.list
    setInit()
  }

  const setInit = () => {
    if(!configStore.distributId || configStore.distributId == '0') {
      state.addItem = {parent_id: '0', class: '2'}
      state.editFrom = [
        { required: true, editshow: true, name: '厂站', key: 'stationNum', type: 'select', list: configStore.stations, value: 'id', label: 'station_name' },
        { required: true, editshow: 'disabled',  name: 'ID', key: 'id',  type: 'input', class: 'number' },
        { required: true,  editshow: true,  name: '名称', key: 'name',  type: 'input' },
        { required: true,  editshow: true,  name: '类型', key: 'type', type: 'select', list: [], value: 'type', label: 'name', relation: ['comm_params'] },
        { required: false, editshow: true, name: 'IP/串口号',   key: 'localSvcIP',  type: 'input' },
        { required: false, editshow: true, name: '端口/波特率', key: 'localSvcPort', type: 'input' },
        { required: false, editshow: true, name: '排序', key: 'order', type: 'input' },
        { required: false, editshow: true, name: '通信参数', key: 'comm_params',  type: 'json' },
      ]
    }else{
      state.addItem = {stationNum: configStore.distributId, parent_id: '0', class: '2'}
      state.editFrom = [
        { required: true, editshow: 'disabled',  name: 'ID', key: 'id',  type: 'input', class: 'number' },
        { required: true,  editshow: true,  name: '名称', key: 'name',  type: 'input' },
        { required: true,  editshow: true,  name: '类型', key: 'type', type: 'select', list: [], value: 'type', label: 'name', relation: ['comm_params'] },
        { required: false, editshow: true, name: 'IP/串口号',   key: 'localSvcIP',  type: 'input' },
        { required: false, editshow: true, name: '端口/波特率', key: 'localSvcPort', type: 'input' },
        { required: false, editshow: true, name: '排序', key: 'order', type: 'input' },
        { required: false, editshow: true, name: '通信参数', key: 'comm_params',  type: 'json' },
      ]
    }
    state.content.forEach(v => {
      if(v.key == 'type') v.list = publicStore._public.templates
    })
    state.editFrom.forEach(v => {
      if(v.key == 'type') v.list = publicStore._public.templates
    })
  }

  const handleClick = (remark, val) => {
    if(remark == 'add' || remark == 'upd'){
      addRef.onVisable(val)
    }
    if(remark == 'del'){
      addRef.del(val)
    }
  }
</script>
  
<style scoped lang="scss">

</style>
  