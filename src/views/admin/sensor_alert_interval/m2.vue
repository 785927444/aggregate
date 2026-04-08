<template>
  <div class="layout-col">
    <!-- 标题 -->
    <aa-title :title="isNull(publicStore.active)?'':publicStore.active.name + ' - 设备告警'">
      <template #left-content></template>
      <template #right-content>
        <div class="rad4 ptb5 plr8 flex-cc cursor bgi1 white ml10" 
          v-if="state.content.find(v=>{return typeof(v.key) == 'object'}) && state.content.find(v=>{return typeof(v.key) == 'object'})['key'].hasOwnProperty('setup')" 
          @click.stop="handleClick('setup')">
          <i-ep-starFilled class="f12 fw" /><span class="f14 ml4">{{state.content.find(v=>{return typeof(v.key) == 'object'})['key'].setup}}</span>
        </div>
        <div class="rad4 ptb5 plr8 flex-cc cursor bgi1 white ml10" 
          v-if="state.content.find(v=>{return typeof(v.key) == 'object'}) && state.content.find(v=>{return typeof(v.key) == 'object'})['key'].hasOwnProperty('sync')" 
          @click.stop="handleClick('sync')">
          <i-ep-helpFilled class="f12 fw" /><span class="f14 ml4">{{state.content.find(v=>{return typeof(v.key) == 'object'})['key'].sync}}</span>
        </div>
        <div class="rad4 ptb5 plr8 flex-cc cursor bgi1 white ml10" @click.stop="router.go(-1)" v-if="route.query&&route.query.sensorid">
          <i-ep-promotion class="f12 fw" /><span class="f14 ml4">返回</span>
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
  import api from '@/api'
  const router = useRouter()
  const route = useRoute()
	const { proxy }:any = getCurrentInstance()
  const publicStore = proxy.publicStore()
  const configStore = proxy.configStore()
  const dictStore = proxy.dictStore()
  let addRef = $ref()
  let statuss = $ref([
    {name: '未配置', value: 0, color: 'i8'},
    {name: '已配置', value: 1, color: 'i12'},
  ])
  const state = reactive({
	  ...publicStore.$state,
    ...dictStore.$state,
    content: [
      { width: 'w50x2', show: true, align: 'left', key: 'alarm_code', name: '告警码' },  
      { width: 'w50x4', show: true, align: 'left', key: 'alarm_name', name: '告警名' },
      { width: 'w50x4', show: true, align: 'left', key: 'attribute_name', name: '告警点' },
      { width: 'w50x4', show: true, align: 'left', key: 'statistics_type', name: '统计类型', type: 'select', list: dictStore.statisticsType, value: 'value', label: 'name' },
      { width: 'w50x4', show: true, align: 'left', key: 'range', name: '范围' },
      { width: 'w50x4', show: true, align: 'left', key: 'status', name: '状态', type: 'select', list: statuss, value: 'value', label: 'name' },
      { width: 'flex1', show: true, align: 'right', key: {setup: '默认', sync: '同步', upd:'编辑', del: '删除'}, name: '操作' },
    ],
    editFrom: [
      { required: true, editshow: 'none', name: '告警点', key: 'attribute_name', type: 'input' }, 
      { required: true, editshow: 'none', name: '告警码', key: 'alarm_code',  type: 'input' },
      { required: true,  editshow: true, name: '告警名', key: 'alarm_name',  type: 'input' },
      { required: false, editshow: true, name: '统计类型', key: 'statistics_type',  type: 'select', list: dictStore.statisticsType, value: 'value', label: 'name' },
      { required: false, editshow: true, name: '左类型', key: 'left_range_type',  type: 'select', list: dictStore.leftRangeType, value: 'value', label: 'name' },
      { required: false, editshow: true, name: '左值', key: 'left_range_value',  type: 'input' },
      { required: false, editshow: true, name: '右类型', key: 'right_range_type',  type: 'select', list: dictStore.rightRangeType, value: 'value', label: 'name' },
      { required: false, editshow: true, name: '右值', key: 'right_range_value',  type: 'input' },
    ],
    addItem: {},
    imgs: [],
  })

  watch(() => publicStore.active, async(val) => {
    await nextTick()
    if(proxy.isNull(val)){
      state.empty = true
      state.list = []
      return
    }
    state.addItem = {sensor_id: publicStore.active.id}
    init()
  }, {immediate: false, deep: true})

  const init = async(key) => {
    let query1 = {model: 't_sensor_template_alarm_point', args: `sensor_type='${publicStore.active.type}'`}
    let query2 = {model: 't_sensor_alert_interval', args: `sensor_id='${publicStore.active.id}'`}
    let params = {Api1: query1, Api2: query2}
    let res = await publicStore.http(params)
    state.empty = proxy.isNull(res.Api1)? true : false
    let list = proxy.isNull(res.Api1)? [] : res.Api1.sort((a, b) => a.order - b.order)
    let arr = []
    list.forEach(v => {
      delete v.id
      if(v.calcRuleExpres){
        try {
          v.calcRuleExpres = JSON.parse(v.calcRuleExpres)
          Object.assign(v, v.calcRuleExpres)
          delete v.calcRuleExpres
          if(!proxy.isNull(v.alarm)){
            v.alarm.forEach(vv => {
              let temp = Object.assign({}, vv)
              temp.attribute_name = v.code
              if(temp.right_range_value && temp.left_range_value){
                let left_range_type = temp.left_range_type=='0'?"(":"["
                let right_range_type = temp.right_range_type=='0'?")":"]"
                temp.range = left_range_type + ' ' + temp.left_range_value + ' , ' + temp.right_range_value + ' ' + right_range_type
              }
              arr.push(temp)
            })
          }
        } catch (err) {
          console.error(err)
        }
      }
    })
    state.list = [...arr]
    state.list.forEach(v => {
      if(proxy.isNull(res.Api2)){
        v.status = 0
      }else{
        let exist = res.Api2.find(a=>a.alarm_code == v.alarm_code)
        if(exist){
          v.status = 1
         if(exist.right_range_value && exist.left_range_value){
            exist.statistics_type = exist.statistics_type + ''
            exist.left_range_type = exist.left_range_type + ''
            exist.right_range_type = exist.right_range_type + ''
            let left_range_type = exist.left_range_type=='0'?"(":"["
            let right_range_type = exist.right_range_type=='0'?")":"]"
            exist.range = left_range_type + ' ' + exist.left_range_value + ' , ' + exist.right_range_value + ' ' + right_range_type
          }
          Object.assign(v, exist)
        }else{
          v.status = 0
        }
      }
    })
    // console.log("state.list---", state.list)
  }

  const handleClick = async(remark, val) => {
    if(remark == 'add' || remark == 'upd'){
      addRef.onVisable(val)
    }
    if(remark == 'del'){
      addRef.del(val)
    }
    if(remark == 'setup'){
      handleSetup()
    }
    if(remark == 'sync'){
      handleSync()
    }
  }

  const handleSetup = async() => {
     ElMessageBox.confirm('【未配置】设置成模板默认值?', '提示', {confirmButtonText: '确定', cancelButtonText: '关闭', type: 'error'}).then(() => {
        let params = {model: state.model, list: []}
        let list = state.list.filter(a=>a.status == 0)
        params.list = list.map(v => {
          let temp = Object.assign({}, v) 
          delete temp.status
          temp.id = ""
          Object.assign(temp, state.addItem) 
          return temp
        })
        console.log("params---", params)
        api.addApi(params).then((res:any) => {
          if(res.code == 200){
            ElNotification({ title: '提示', message: '操作成功', type: 'success' })
            init()
          }else{
            ElNotification({ title: '提示', message: '操作失败', type: 'error' })
          }
        }).catch((err) => {
          ElNotification({ title: '提示', message: '操作失败', type: 'error' })
        })
      })
  }

  const handleSync = async() => {
    let list = state.list.filter(a=>a.status == 1)
    if(proxy.isNull(list)) return ElNotification({ title: '提示', message: '请先配置至少一个', type: 'error' })
    ElMessageBox.confirm('是否同步【已配置】到同类型设备?', '提示', {confirmButtonText: '确定', cancelButtonText: '关闭', type: 'error'}).then(async() => {
      let query = {model: 't_sensor', args: `type='${publicStore.active.type}'`}
      let res = await publicStore.http({Api: query})
      if(!proxy.isNull(res)){
          await delList()
          await addList(res)
      }
      
    })
  }

  const delList = async() => {
    let params = {model: state.model, list: []}
    let query = {
      model: 't_sensor_alert_interval', 
      join: 't_sensor ON t_sensor.id = t_sensor_alert_interval.sensor_id', 
      field: `t_sensor_alert_interval.id`, 
      args: `t_sensor.type='${publicStore.active.type}'`
    }
    let res = await publicStore.http({Api: query})
    if(!proxy.isNull(res)) {
      params.list = [...res]
      console.log("params---", params)
      api.delApi(params).then((res:any) => {
        if(res.code == 200){
          ElNotification({ title: '提示', message: '删除成功', type: 'success' })
        }else{
          ElNotification({ title: '提示', message: '删除失败', type: 'error' })
        }
      }).catch((err) => {
        ElNotification({ title: '提示', message: '删除失败', type: 'error' })
      })
    }
  }

  const addList = async(res) => {
    let params = {model: state.model, list: []}
    let list = state.list.filter(a=>a.status == 1)
    res.forEach(item => {
      let aynclist = list.map(v => {
        let temp = Object.assign({}, v) 
        delete temp.status
        temp.id = ""
        Object.assign(temp, {sensor_id: item.id}) 
        return temp
      })
      params.list = [...params.list, ...aynclist]
    })
    console.log("params---", params)
    api.addApi(params).then((res:any) => {
      if(res.code == 200){
        ElNotification({ title: '提示', message: '添加成功', type: 'success' })
        init()
      }else{
        ElNotification({ title: '提示', message: '添加失败', type: 'error' })
      }
    }).catch((err) => {
      ElNotification({ title: '提示', message: '添加失败', type: 'error' })
    })
  }
</script>
  
<style scoped lang="scss">

</style>
  