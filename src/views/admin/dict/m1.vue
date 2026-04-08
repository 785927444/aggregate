<template>
  <div class="layout-col flex1">
    <!-- 标题 -->
    <aa-title title="字典"><template #right-content></template> </aa-title>    
    <!-- 内容 -->
    <div class="layout-col bg-white">
      <ListTree @handleNodeClick="handleNodeClick" @handleClick="handleClick" :state="state" ref="listTreeRef"/>
    </div>
    <!-- 底部 -->
    <aa-foot></aa-foot>
    <!-- <Add @init="init" :state="state" ref="addRef" /> -->
  </div>
</template>

<script lang="ts" setup>
  import axios from 'axios'
	const { proxy }:any = getCurrentInstance()
  const publicStore = proxy.publicStore()
  const configStore = proxy.configStore()
  const dictStore = proxy.dictStore()
  let defaultId = $ref()
  let listTreeRef = $ref()
  const state = reactive({
	  ...publicStore.$state,
    content: [
      { width: 'w70',   show: true, align: 'left', key: '*', name: '序号' },
      { width: 'w50x3', show: true, align: 'left',  key: 'name', name: '名称', type: 'icon1' },
    ],
    types: [{id: '1', name: '算法', type: 'algorithm', order: '100'}],
  })

  const init = async() => {
    let node = proxy.findNode(state.tree, (node) => { return !node.children })
    await nextTick()
    listTreeRef.handleNodeClick(node)
  }

  onMounted(async() => {
    state.list = state.types
    publicStore._public.list = state.list
    publicStore.active = state.list[0]
    init()
  })

  const handleClick = (remark, val) => {
    if(remark == 'active'){
      publicStore.active = {...val}
    }
  }
  const handleNodeClick = async(node) => {
    defaultId = node[state.key]
    if(!node.classify) publicStore.active = Object.assign({}, node)
  }
</script>
  
<style scoped lang="scss">

</style>
  