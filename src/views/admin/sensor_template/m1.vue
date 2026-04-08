<template>
  <div class="layout-col flex1">
    <!-- 标题 -->
		<ab-title title="类型">
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
		<!-- 底部 -->
		<aa-foot></aa-foot>
    <Add @init="init" :state="state" ref="addRef" />
    <!-- <ImportData @init="init" :state="state" ref="importRef"/> -->
  </div>
</template>

<script lang="ts" setup>
  const { proxy }:any = getCurrentInstance()
  const publicStore = proxy.publicStore()
  const configStore = proxy.configStore()
  const dictStore = proxy.dictStore()
  let importRef = $ref()
  let addRef = $ref()
  let treeRef = $ref()
  let defaultId = $ref()
  let listTreeRef = $ref()
  let defaultProps = {
    children: 'children',
    label: 'name',
  }
  const state = reactive({
    ...publicStore.$state,
    ...dictStore.$state,
    content: [
      { width: 'flex1', show: true, align: 'left', key: 'name', name: '名称', type: 'icon5' },  
      { width: 'w50x2', show: true, align: 'right', key: {add:'添加', upd:'编辑', del: '删除'},  name: '操作' },
    ],
    editFrom: [
      { required: true,  editshow: false, name: '上级', key: 'parent_id', type: 'select', list: [], value: 'id', label: 'name' },
      { required: true,  editshow: true, name: '分类', key: 'attribute', type: 'select', list: dictStore.classify, value: 'value', label: 'name' },
      { required: true,  editshow: true, name: '编码', key: 'type',  type: 'input' },
      { required: true,  editshow: true, name: '名称', key: 'name',  type: 'input' },
      { required: false, editshow: true, name: '排序', key: 'order',  type: 'input' },
      { required: false, editshow: true, name: '通信协议', key: 'ismapping2star', type: 'select', list: dictStore.agreement, value: 'value', label: 'name' },
      { required: false, editshow: true, name: '通信参数', key: 'comm_params',  type: 'json' },
      { required: false, editshow: true, name: '详细参数', key: 'description',  type: 'json' },
    ],
    addItem: {},
    model: 't_sensor_template',
  })

  onMounted(async() => {
    publicStore.actIndex = 2
    await setInit()
    init()
  })

  const setInit = async() => {
    state.query = {model: 't_class'}
    state.params = {Api: state.query}
    let res = await publicStore.http(state.params)
    res = proxy.isNull(res)? [] : res.sort((a, b) => a.order - b.order)
    publicStore._public.classList = res.map(v => {
      let temp = res.find(a=>a.parent_id == v.id)
      v.classify = temp? 1 : 2  // 1：有子级 2：无子级
      return v
    })
    state.editFrom[0].list = publicStore._public.classList
  }

  const init = async(key) => {
    state.lists = []
    state.query = {model: state.model}
    state.params = {Api: state.query}
    let res = await publicStore.http(state.params)
    state.empty = proxy.isNull(res)? true : false
    let list = proxy.isNull(res)? [] : res.sort((a, b) => a.attribute - b.attribute)
    state.list = list.sort((a, b) => a.order - b.order)
    publicStore._public.list = state.list
    state.lists = [...publicStore._public.classList, ...publicStore._public.list]
    let node = proxy.findNode(state.tree, (node) => { return !node.children })
    setTree(key)
    listTreeRef.handleNodeClick(node)
  }

   const setTree = async(key) => {
    state.tree = proxy.treeData(state.lists, state.key, state.parent, 'children')
    let node1 = proxy.findNode(state.tree, (node) => { return !node.children })
    let node2 = proxy.findNode(state.tree, (node) => { return node[state.key] == key })
    let node = proxy.isNull(node2)? node1 : node2
    await nextTick()
    defaultId = node[state.key]
    treeRef.setCurrentKey(defaultId, true)
    if(!node.classify) publicStore.active = Object.assign({}, node)
  }

  const handleNodeClick = async(node) => {
    defaultId = node[state.key]
    if(!node.classify) publicStore.active = Object.assign({}, node)
  }

  const handleClick = (remark, val) => {
    if(remark == 'add'){
      state.addItem = {parent_id: val.id}
      addRef.onVisable(state.addItem)
    }
    if(remark == 'upd'){
      addRef.onVisable(val)
    }
    if(remark == 'child'){
      let vall = proxy.varObj(state.parent, val[state.key])
      addRef.onVisable(vall)
    }
    if(remark == 'del'){
      addRef.del(val)
    }
  }
</script>
  
<style scoped lang="scss">

</style>
  