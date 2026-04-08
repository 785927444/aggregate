<template>
  <div class="layout-col ">
    <!-- 标题 -->
    <aa-title title="聚合用户">
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
    <Detail @init="init" :state="state" ref="detailRef" />
  </div>
</template>

<script lang="ts" setup>
	const { proxy }:any = getCurrentInstance()
  const publicStore = proxy.publicStore()
  const configStore = proxy.configStore()
  const dictStore = proxy.dictStore()
  let addRef = $ref()
  let detailRef = $ref()
  let values = [
    {name: '否', value: 0},
    {name: '是', value: 1},
  ]
  const state = reactive({
	  ...publicStore.$state,
    content: [
      { width: 'w50',   show: true, align: 'left', key: '*', name: '序号' },
      { width: 'w50x3', show: true, align: 'left',  key: 'station_name', name: '名称', type: 'icon1' },
      { width: 'w50x3', show: true, align: 'left',  key: 'industry_type', name: '行业类型', type: 'select', list: [], value: 'id', label: 'name' },
      { width: 'w50x3', show: true, align: 'left',  key: 'administrative_areaid', name: '行政区域' },
      { width: 'w50x3', show: true, align: 'left',  key: 'contact', name: '联系人员' },
      { width: 'w50x3', show: true, align: 'left',  key: 'telephone', name: '联系号码' },
      { width: 'w50x2', show: true, align: 'left',  key: 'day_out_response', name: '日前响应', type: 'select', list: values, label: 'name', value: 'value' },
      { width: 'w50x2', show: true, align: 'left',  key: 'day_in_response', name: '日内响应', type: 'select', list: values, label: 'name', value: 'value' },
      { width: 'w50x2', show: true, align: 'left',  key: 'yn_sign', name: '是否签约', type: 'select', list: values, label: 'name', value: 'value' },
      { width: 'w50x2', show: true, align: 'left',  key: 'devices', name: '设备数量' },
      { width: 'w50x2', show: true, align: 'left',  key: 'up_adjustable', name: '最大上调容量' },
      { width: 'w50x2', show: true, align: 'left',  key: 'down_adjustable', name: '最大下调容量' },
      { width: 'flex1', show: true, align: 'right', key: {}, name: '操作' }, 
    ],
    editFrom: [
      { required: true, editshow: 'none', name: '上级', key: 'parent_id', type: 'input' },
      { required: true, editshow: true, name: '名称', key: 'station_name', type: 'input' },
      // { required: false, editshow: true, name: '关联', key: 'node_id', type: 'select', list: [], label: 'name', value: 'id' },
      { required: false, editshow: true, name: '排序', key: 'order',  type: 'input' },
    ],
    addItem: {parent_id: '0'},
  })

  onMounted(async() => {
    publicStore.init({path: '/station'}, state)
    if(!configStore.distributId) configStore.distributId = '0'
    await setInit()
    getInit()
  })

  const setInit = async() => {
    // 获取行业
    let query1 = {model: 't_industry', args: `parent_id='0'`}
    publicStore.http({Api: query1}).then(res=>{
      state.industrys = proxy.isNull(res)? [] : res.sort((a, b) => a.order - b.order)
      state.content.forEach(v => {
        if(v.key == 'industry_type') v.list = [...state.industrys]
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

  const getInit = async() => {
    let list = []
    if(!proxy.isNull(configStore.stations)){
      list = configStore.stations
    }else{
      state.query = {model: state.model, args: `parent_id='0'`}
      state.params = {Api: state.query}
      let res = await publicStore.http(state.params)
      list = proxy.isNull(res)? [] : res.sort((a, b) => a.order - b.order)
    }
    state.empty = proxy.isNull(list)? true : false
    state.list = list
    publicStore._public.list = state.list
    configStore.stations = state.list
    getNewList()
  }

  const init = async(key) => {
    state.query = {model: state.model, args: `parent_id='0'`}
    state.params = {Api: state.query}
    let res = await publicStore.http(state.params)
    let list = proxy.isNull(res)? [] : res.sort((a, b) => a.order - b.order)
    state.empty = proxy.isNull(list)? true : false
    state.list = list
    publicStore._public.list = state.list
    configStore.stations = state.list
    getNewList()
  }

  const getNewList = () => {
    state.list.forEach(v => {
      v.area = v.province_name && v.city_name? `${v.province_name}-${v.city_name}` : ''
      let query = {model: 't_sensor', args: `stationNum='${v.id}' and class='0'`, field: `COUNT(*)`}
      publicStore.http({Api: query}).then(ress=>{
        v.devices = proxy.isNull(ress)? 0 : ress[0]['COUNT(*)']
      })
    })
  }

  const handleClick = async(remark, val) => {
    if(remark == 'add' || remark == 'upd'){
      addRef.onVisable(val)
    }
    if(remark == 'del'){
      addRef.del(val)
    }
    if(remark == 'detail'){
      detailRef.onVisable(val)
    }
    if(remark == 'active'){
      publicStore.active = val
    }
  }
</script>
  
<style scoped lang="scss">

</style>
  