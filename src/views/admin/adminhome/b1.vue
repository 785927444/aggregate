<template>
  <div class="layout-col bg-white">
    <!-- 标题 -->
    <div class="ww100 flex-sc plr20 pt15">
      <span class="card-title-home"></span>
      <span>进行中的交易</span>
    </div>
    <!-- 内容 -->
    <div class="layout-col white-rgba50 rad8 p15">
      <List @handleClick="handleClick" :state="state"/>
    </div>
    <!-- 底部 -->
    <aa-foot></aa-foot>
  </div>
</template>

<script lang="ts" setup>
  const { proxy }:any = getCurrentInstance()
  const publicStore = proxy.publicStore()
	const configStore = proxy.configStore()
  const dictStore = proxy.dictStore()
  const statuss = [
    {value: '0', name: '未申报'},
    {value: '1', name: '待执行'},
    {value: '2', name: '进行中'},
    {value: '3', name: '已完成'},
  ]
  const state = reactive({
	  ...publicStore.$state,
    content: [
      { width: 'w70',   show: true, align: 'center', key: '*', name: '序号' },
      { width: 'w50x3', show: true, align: 'left', key: 'transaction_id', name: '交易编号' },  
      { width: 'w50x3', show: true, align: 'left', key: 'title_name', name: '交易名称' },
      { width: 'w50x2', show: true, align: 'left', key: 'response_date', name: '响应日期', type: 'time', timetype: '{y}-{m}-{d}' },
      { width: 'w50x2', show: true, align: 'left', key: 'demand_time_slot', name: '响应时段' },
      { width: 'w50x2', show: true, align: 'left', key: 'demand_type', name: '需求类型' }, 
      { width: 'w50x3', show: true, align: 'left', key: 'apply_capacity', name: '申报容量(kW)' }, 
      { width: 'w50x3', show: true, align: 'left', key: 'win_capacity', name: '中标容量(kW)' }, 
      { width: 'w50x3', show: true, align: 'left', key: 'status', name: '状态', type: 'select', list: statuss, value: 'value', label: 'name' }, 
      { width: 'flex1', show: true, align: 'right', key: {detail: '详情'},  name: '操作' },
    ],
  })

  watch(() => publicStore._public.transactionData, async(val, old) => {
    if(proxy.isNull(val)) return
    if(JSON.stringify(val) == JSON.stringify(old)) return
    await nextTick()
    state.list = [...val]
  }, { immediate: false, deep: true });

</script>
 
<style scoped lang="scss">
.card-title-home {
  display: block;
  margin-right: 6px;
  float: left;
  width: 4px;
  height: 18px;
  border-radius: 133px;
  background: linear-gradient(0deg, #1B76FF 0%, rgba(77, 131, 238, 0) 99%);
}
</style>
