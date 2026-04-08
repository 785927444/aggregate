<template>
  <div class="layout-col">
    <!-- 标题 -->
    <ab-title title="设备">
      <template #right-content>
        <div class="rad4 ptb5 plr8 flex-cc cursor bgi1 white ml10" 
          v-if="state.content.find(v=>{return typeof(v.key) == 'object'}) && state.content.find(v=>{return typeof(v.key) == 'object'})['key'].hasOwnProperty('add')" 
          @click.stop="addRef.onVisable(state.addItem)">
          <i-ep-plus class="f12 fw" /><span class="f14 ml2">{{state.content.find(v=>{return typeof(v.key) == 'object'})['key'].add}}</span>
        </div>
      </template>
    </ab-title>
    <!-- 内容 -->
    <div class="layout-col bg-white">
      <ListTree @handleNodeClick="handleNodeClick" @handleClick="handleClick" :state="state" ref="listTreeRef"/>
    </div>
    <Add @init="init" :state="state" ref="addRef" />
  </div>
</template>

<script lang="ts" setup>
  const route = useRoute()
	const { proxy }:any = getCurrentInstance()
  const publicStore = proxy.publicStore()
  const configStore = proxy.configStore()
  let addRef = $ref()
  let exposeRef = $ref()
  let listTreeRef = $ref()
  const state = reactive({
	  ...publicStore.$state,
    listRow: true,
    content: [
      { width: 'flex1', show: true, align: 'start', key: 'name', name: '名称', type: 'icon2' }, 
    ],
    editFrom: [
      { required: true, editshow: true,  name: '上级', key: 'up_id', type: ['tree', [], 'id', 'up_id', '顶级'] },
      { required: true, editshow: true,  name: '名称', key: 'name',  type: ['input'] },
    ],
    addItem: {},
    model: 't_sensor',
  })
  
  onMounted(async() => {
    // 路由参数
    state.route = Object.assign({}, route.query)
    if(!configStore.distributId) configStore.distributId = '0'
    let query = {model: "t_station", args: `parent_id='0'`}
    let params = {Api: query}
    let res = await publicStore.http(params)
    publicStore._public.stations = proxy.isNull(res)? [] : res.sort((a, b) => a.order - b.order)
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

  const init = async(key) => {
    state.query = configStore.distributId=='0'? {model: state.model} : {model: state.model, args: `stationNum='${configStore.distributId}'`}
    state.params = {Api: state.query}
    let res = await publicStore.http(state.params)
    state.empty = proxy.isNull(res)? true : false
    state.list = proxy.isNull(res)? [] : res.sort((a, b) => a.order - b.order)
    publicStore._public.list = state.list
    setTree(key)
  }

  const setTree = async(key) => {
    state.tree = proxy.treeData(state.list, state.key, state.parent, 'children')
    let node1 = proxy.findNode(state.tree, (node) => { return !node.children })
    let node2 = proxy.findNode(state.tree, (node) => { return node[state.key] == key })
    let node = proxy.isNull(node2)? node1 : node2
    await nextTick()
    listTreeRef.handleNodeClick(node)
  }

  const handleNodeClick = (val) => { 
    if(!val.children) publicStore.active = {...val} 
  }

  const handleClick = (remark, val) => {

  }
</script>
  
<style scoped lang="scss">

</style>
  