<template>
  <div class="layout-col flex3 ml16">
    <!-- 标题 -->
		<aa-title :title="isNull(publicStore.active)?'':publicStore.active.name + ' - 测量'">
      <template #right-content>
        <div class="flex-ec flex1">
          <div class="rad4 hidden flex mr15" v-if="dictStore.typeSelect">
            <div class="ptb5 plr10 flex-cc cursor white" :class="publicStore.actIndex == v.value?'bs4':'bg3'" :style="{marginLeft: i==0?'':'1px'}" v-for="(v, i) in dictStore.typeSelect" :key="i"
            @click.stop="publicStore.actIndex = v.value; init()">{{v.name}}</div>
          </div>
          <!-- 新增导入按钮 -->
          <div class="rad4 ptb5 plr8 flex-cc cursor bgi22 i21 ml10 bo-i21-1" 
            @click.stop="handleClick('in')"> 
            <i-ep-switch class="f12 fw" /><span class="f14 ml5">导入</span>
          </div>
          
          <!-- 新增导出按钮 -->
          <div class="rad4 ptb5 plr8 flex-cc cursor bgi22 i21 ml10 bo-i21-1" 
            @click.stop="handleClick('out')"> 
            <i-ep-filter class="f12 fw" /><span class="f14 ml5">导出</span>
          </div>
          <div class="rad4 ptb5 plr8 flex-cc cursor bgi22 i21 ml10 bo-i21-1" 
            v-if="state.content.find(v=>{return typeof(v.key) == 'object'}) && state.content.find(v=>{return typeof(v.key) == 'object'})['key'].hasOwnProperty('add')" 
            @click.stop="handleClick('add', state.addItem)">
            <i-ep-plus class="f12 fw" /><span class="f14 ml4">{{state.content.find(v=>{return typeof(v.key) == 'object'})['key'].add}}</span>
          </div>
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
    <!-- 新增ImportData组件 -->
    <ImportData @init="init" :state="state" ref="importRef"/>
  </div>
</template>

<script lang="ts" setup>
  // 引入ImportData组件
  import ImportData from '@/components/ImportData.vue'
	const { proxy }:any = getCurrentInstance()
  const publicStore = proxy.publicStore()
  const configStore = proxy.configStore()
  const dictStore = proxy.dictStore()
  let addRef = $ref()
  // 在setup中声明importRef
  let importRef = $ref()
  const state = reactive({
	  ...publicStore.$state,
    ...dictStore.$state,
    content: [
      { width: 'w50x5', show: true, align: 'left', key: 'name', name: '名称' },
      { width: 'w50x5', show: true, align: 'left', key: 'code', name: '编码' },
      { width: 'w50x2', show: true, align: 'center', key: 'value_type', name: '数据', type: 'select', list: dictStore.valueType, value: 'value', label: 'name' },
      { width: 'w50x2', show: true, align: 'center', key: 'type', name: '运算', type: 'select', list: dictStore.measureType, value: 'value', label: 'name' },
      { width: 'w50x2', show: true, align: 'center', key: 'ismapping2star', name: '映射', type: 'select', list: dictStore.isList, value: 'value', label: 'name' },
      { width: 'w50x6', show: true, align: 'left', key: 'description', name: '描述' },
      { width: 'flex1', show: true, align: 'right', key: {add: '新增', upd:'编辑', del: '删除'},  name: '操作' },
    ],
    editFrom: [
      { required: true,  editshow: 'none', name: '上级', key: 'sensor_type', type: 'input'},
      { required: true,  editshow: true, name: '编码', key: 'code',  type: ['input'] },
      { required: true,  editshow: true, name: '名称', key: 'name',  type: ['input'] },
      { required: true,  editshow: true, name: '数据', key: 'value_type', type: 'select', list: dictStore.valueType, value: 'value', label: 'name'},
      { required: false, editshow: true, name: '运算', key: 'type', type: 'select', list: dictStore.measureType, value: 'value', label: 'name'},
      { required: false, editshow: true, name: '描述', key: 'description', type: ['input'] },
      { required: false, editshow: true, name: '映射', key: 'ismapping2star', type: 'select', list: dictStore.isList, value: 'value', label: 'name' },
      { required: false, editshow: true, name: '排序', key: 'order', type: ['input'] },
      { required: false, editshow: true, name: '自定义', key: 'calcRuleExpres', type: ['input'] },
    ],
    addItem: {},
  })
	onMounted(async() => {
    if(proxy.isNull(publicStore.active)) return
    state.form = Object.assign({}, publicStore.active)
    state.addItem = {sensor_type: state.form.type}
    state.query = {args: `sensor_type='${state.form.type}'`}
	  init()
	})

  watch(() => publicStore.active, async(val) => {
    await nextTick()
    if(proxy.isNull(val)) return
    state.form = Object.assign({}, val)
    state.addItem = {sensor_type: state.form.type}
    state.query = {args: `sensor_type='${state.form.type}'`}
    init()
  }, {immediate: false, deep: true})

  const init = async(key) => {
    if(publicStore.actIndex != 2) return
    let query = proxy.find(dictStore.typeSelect, ['value', publicStore.actIndex], 'query')
    state.model = query.model
    let params = {Api: Object.assign(query, state.query)}
    let res = await publicStore.http(params)
    state.empty = proxy.isNull(res)? true : false
    state.list = proxy.isNull(res)? [] : res
    publicStore._public.meaList = state.list
  }

  const handleClick = (remark, val) => {
    if(remark == 'add' || remark == 'upd'){
       addRef.onVisable(val)
    }
    if(remark == 'child'){
      let vall = proxy.varObj(state.parent, val[state.key])
      addRef.onVisable(vall)
    }
    if(remark == 'del'){
      addRef.del(val)
    }
    if (remark == 'out') {
      // 准备导出内容
      let content = [...state.editFrom] 
      let list = [...state.list] 
      console.log(list, '打印梳理2')
      console.log(content, '打印梳理1')
       // 根据页面类型设置导出名称 
      const pageNames = {
        2: '测量',
        3: '配置',
        4: '告警',
        5: '控制'
      }
      let name = `${publicStore.active.name  || ''}${pageNames[publicStore.actIndex]}数据`
      // 调用导出方法
      proxy.handleExport(content,  list, name)
    }
    if (remark == 'in') {
      // 准备导入配置
      let content = [...state.editFrom] 
      let ctx = {
        content: content,
        key: state.key, 
        query: state.query, 
        unique: '' // 根据实际情况设置唯一键
      }
      importRef.onVisable(ctx) 
    }
  }
</script>
  
<style scoped lang="scss">

</style>
  