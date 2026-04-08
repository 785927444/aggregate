<template>
  <div class="layout-col">
    <!-- 标题 -->
    <aa-title title="网关">
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
    <Add @init="init" :state="state" ref="addRef" />
  </div>
</template>

<script lang="ts" setup>
	const { proxy }:any = getCurrentInstance()
  const publicStore = proxy.publicStore()
  const configStore = proxy.configStore()
  const dictStore = proxy.dictStore()
  let asyncsRef = $ref()
  let addRef = $ref()
  const state = reactive({
	  ...publicStore.$state,
    content: [
      { width: 'w50x4', show: true, align: 'left',  key: 'machine_name', name: '名称' },
      { width: 'w50x4', show: true, align: 'left',  key: 'gateway_type', name: '类型', type: 'select', list: [], value: 'gateway_class', label: 'gateway_name' },
      { width: 'w50x8', show: true, align: 'left',  key: 'machine_code', name: '物理地址' },
      { width: 'w50x3', show: true, align: 'left',  key: 'parent_node_ip', name: 'IP' },
      { width: 'w50x3', show: true, align: 'left',  key: 'parent_node_port', name: '端口' },
      { width: 'flex1', show: true, align: 'right', key: {},  name: '操作' },
    ],
    editFrom: [],
    addItem: {},
  })

  onMounted(async() => {
    await publicStore.init({path: '/gateway'}, state)
    if(!configStore.distributId) configStore.distributId = '0'
    let query1 = {model: "t_station", args: `parent_id='0'`}
    let query2 = {model: "t_gateway_type"}
    let params = {Api1: query1, Api2: query2}
    let res = await publicStore.http(params)
    publicStore._public.stations = proxy.isNull(res.Api1)? [] : res.Api1.sort((a, b) => a.order - b.order)
    publicStore._public.gatewayTypes = proxy.isNull(res.Api2)? [] : res.Api2
    init()
  })

  watch(() => configStore.distributId, async(val, old) => {
    await nextTick()
    if(proxy.isNull(val)) return
    init()
  }, {immediate: false, deep: true})

  watch(() => publicStore.isTrue, async(val) => {
    await nextTick()
    console.log('更新---')
    init(publicStore.active.id)
  }, {immediate: false, deep: true})
  
  const init = async() => {
    state.query = !configStore.distributId || configStore.distributId=='0'? {model: state.model} : {model: state.model, args: `station_code='${configStore.distributId}'`}
    state.params = {Api: state.query}
    let res = await publicStore.http(state.params)
    state.empty = proxy.isNull(res)? true : false
    state.list = proxy.isNull(res)? [] : res
    setInit()
  }

  const setInit = async() => {
    state.content.forEach(v => { if(v.key == 'gateway_type') v.list = publicStore._public.gatewayTypes })
    if(!configStore.distributId || configStore.distributId == '0') {
      state.addItem = {}
      state.editFrom = [
        { required: true, editshow: true, name: '上级', key: 'station_code', type: 'select', list: publicStore._public.stations, value: 'id', label: 'station_name' },
        { required: true, editshow: true, name: '名称', key: 'machine_name', type: 'input' },
        { required: false, editshow: true, name: '类型', key: 'gateway_type', type: 'select', list: publicStore._public.gatewayTypes, value: 'gateway_class', label: 'gateway_name' },
        { required: false, editshow: true, name: '物理地址', key: 'machine_code', type: 'input' },
        { required: false, editshow: true, name: 'IP', key: 'parent_node_ip', type: 'input' },
        { required: false, editshow: true, name: '端口', key: 'parent_node_port', type: 'input' },
      ]
    }else{
      state.addItem = {station_code: configStore.distributId}
      state.editFrom = [
        { required: true, editshow: 'none', name: '上级', key: 'station_code', type: 'input' },
        { required: true, editshow: true, name: '名称', key: 'machine_name', type: 'input' },
        { required: false, editshow: true, name: '类型', key: 'gateway_type', type: 'select', list: publicStore._public.gatewayTypes, value: 'gateway_class', label: 'gateway_name' },
        { required: false, editshow: true, name: '物理地址', key: 'machine_code', type: 'input' },
        { required: false, editshow: true, name: 'IP', key: 'parent_node_ip', type: 'input' },
        { required: false, editshow: true, name: '端口', key: 'parent_node_port', type: 'input' },
      ]
    }
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
  