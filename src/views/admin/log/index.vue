<template>
  <div class="layout-col">
    <!-- 标题 -->
    <aa-title title="操作日志" class="mb10">
      <template #right-content>
        <div class="flex-ec cursor flex1">
          <div class="ml15 mr10">用户</div>
          <el-input class="w100x2" v-model.trim="state.username" placeholder="请输入" />
          <div class="ml15 mr10">等级</div>
          <div class="flex w50x2">
            <el-select v-model="state.level" placeholder="请选择" style="width: 100%" clearable filterable>
              <el-option label="全部" value="" />
              <el-option v-for="(v, i) in levels" :key="v.value" :label="v.name" :value="v.value" />
            </el-select>
          </div>
          <div class="ml15 mr10">时间</div>
          <div class="flex w50x8">
            <el-date-picker v-model="state.datetime" format="YYYY-MM-DD HH:mm:ss" value-format="YYYY-MM-DD HH:mm:ss"
              type="datetimerange" range-separator="-" start-placeholder="开始时间" end-placeholder="结束时间" />
          </div>
          <el-button class="rad4 ptb5 plr8 flex-cc cursor bgi1 white ml10 bo-i21-1 ml15" @click.stop="state.page = 1; init()" :icon="Search">搜索</el-button>
          <el-button class="rad4 ptb5 plr8 flex-cc cursor bgi1 white ml10 bo-i21-1" @click.stop="exports()" :icon="Filter">导出</el-button>
        </div>
        <div class="rad4 ptb5 plr8 flex-cc cursor bgi22 i21 ml10 bo-i21-1" 
          v-if="state.content.find(v=>{return typeof(v.key) == 'object'}) && state.content.find(v=>{return typeof(v.key) == 'object'})['key'].hasOwnProperty('add')" 
          @click.stop="handleClick('add', state.addItem)">
          <i-ep-plus class="f12 fw" /><span class="f14 ml4">{{state.content.find(v=>{return typeof(v.key) == 'object'})['key'].add}}</span>
        </div>
      </template>
    </aa-title>
    <!-- 内容 -->
    <div class="layout-col white-rgba50 rad8 p15">
      <List @handleClick="handleClick" :state="state"/>
      <Pagination class="plr20" v-show="state.total>0" :total="state.total" v-model:page.sync="state.page" v-model:limit.sync="state.limit" @pagination="init" />
    </div>
    <Add @init="init" :state="state" ref="addRef" />
  </div>
</template>

<script lang="ts" setup>
  import { Refresh, Search, Filter, Connection, Edit, Operation } from '@element-plus/icons-vue'
	const { proxy }:any = getCurrentInstance()
  const publicStore = proxy.publicStore()
  const configStore = proxy.configStore()
  const dictStore = proxy.dictStore()
  let asyncsRef = $ref()
  let addRef = $ref()
  let ress = $ref([
    {name: '失败', value: '0', color: 'i8'},
    {name: '成功', value: '1', color: 'i11'},
  ])
  let types = $ref([
    {name: '常规', value: '1'},
    {name: '系统', value: '2'},
  ])
  let levels = $ref([
    {name: '注意', value: 'info', color: 'i15'},
    {name: '警告', value: 'warning', color: 'i6'},
    {name: '错误', value: 'error', color: 'i8'},
    {name: '致命', value: 'fatal', color: 'i9'}
  ])
  const state = reactive({
	  ...publicStore.$state,
    content: [
      { width: 'w70',   show: true, align: 'center', key: '*', name: '序号' },
      { width: 'w50x2', show: true, align: 'left',  key: 'type', name: '类型', type: 'select', list: types, value: 'value', label: 'name' },
      { width: 'w50x2', show: true, align: 'left',  key: 'level', name: '等级', type: 'select', list: levels, value: 'value', label: 'name' },
      { width: 'w50x2', show: true, align: 'left',  key: 'res', name: '结果', type: 'select', list: ress, value: 'value', label: 'name'  },
      { width: 'w50x3', show: true, align: 'left',  key: 'name', name: '名称' },
      { width: 'w50x3', show: true, align: 'left',  key: 'user', name: '用户', type: 'decrypt' },
      { width: 'w50x3', show: true, align: 'left',  key: 'username', name: '账号', type: 'decrypt' },
      { width: 'w50x4', show: true, align: 'left',  key: 'time', name: '时间', type: 'time' },
      { width: 'w50x8', show: true, align: 'left',  key: 'msg', name: '内容' },
      // { width: 'w50x5', show: true, align: 'left',  key: 'table', name: '表' },
      // { width: 'w50x2', show: true, align: 'left',  key: 'sensortype', name: '设备类型' },
      // { width: 'w50x2', show: true, align: 'left',  key: 'sensorid', name: '设备ID' },
      { width: 'flex1', show: true, align: 'right', key: publicStore.auth,  name: '操作' },
    ],
    editFrom: [],
    addItem: {},
    username: '',
    type: '1',
    level: '',
    model: 't_log',
  })

  onMounted(async() => {
    publicStore.init({path: '/log'}, state)
    if(!configStore.distributId) configStore.distributId = '0'
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
    state.query = !configStore.distributId || configStore.distributId=='0'? {model: state.model, order: 'time desc'} : {model: state.model, args: `stationnum='${configStore.distributId}'`, order: 'time desc'}
    let args = ''
    if(state.username) args += `user LIKE '%${proxy.encrypt(state.username)}%' and `
    if(state.type) args += `type='${state.type}' and `
    if(state.level) args += `level='${state.level}' and `
    if(!proxy.isNull(state.datetime)){
      let start = Date.parse(state.datetime[0]) + ''
      let end = Date.parse(state.datetime[1]) + ''
      args += `time>='${start}' and time<='${end}' and `
    }
    let q1 = {limit: state.limit, page: state.page}
    let q2 = {field: `COUNT(*)`}
    let qs = {}
    Object.assign(qs, state.query)
    if(args) qs.args = args.slice(0, -4)
    let query1 = {}
    let query2 = {}
    Object.assign(query1, qs, q1)
    Object.assign(query2, qs, q2)
    state.parmas = {Api1: query1, Api2: query2}
    let res = await publicStore.http(state.parmas)
    state.empty = proxy.isNull(res.Api1)? true : false
    state.list = proxy.isNull(res.Api1)? [] : res.Api1
    state.total = proxy.isNull(res.Api2)? 0 : res.Api2[0]['COUNT(*)']
  }

  const handleClick = (remark, val) => {
    if(remark == 'add' || remark == 'upd'){
      addRef.onVisable(val)
    }
    if(remark == 'del'){
      addRef.del(val)
    }
  }
  
  const exports = async() => {
    let query = Object.assign({}, state.parmas.Api1)
    delete query.limit
    delete query.page
    let tempData = await publicStore.http({Api: query})
    import('@/utils/Export2Excel').then(async excel => {
      const tHeader   = []
      const filterVal = []
      state.content.forEach(item => {
        if(item.show && item.key != '-' && item.key != '*' && typeof(item.key) != 'object'){
          tHeader.push(item.name)
          filterVal.push(item.key)
        }
      })
      const data = formatJsons(filterVal, tempData)
      excel.export_json_to_excel({
        header: tHeader,
        data,
        filename: proxy.parseTime(new Date()) + '导出'
      })
    })
  }

  const formatJsons = (filterVal, data) => {
    let list = data? data : state.list
    return list.map(v => filterVal.map(j => {
      if(j=="time"){
        return v[j]&&v[j]!='0'? proxy.parseTime(v[j]) : ''
      }else if(j=="type"){
        return proxy.find(types, ['value', v[j]], 'name')
      }else if(j=="level"){
        return proxy.find(levels, ['value', v[j]], 'name')
      }else{
        return v[j]
      }
    }))
  }
</script>
  
<style scoped lang="scss">

</style>
  