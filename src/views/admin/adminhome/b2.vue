<template>
  <div class="layout-col bg-white">
    <!-- 标题 -->
    <div class="ww100 flex-sc plr20 pt15">
      <span class="card-title-home"></span>
      <span class="flex1">市场需求</span>
      <span class="i1 flex-sc cursor">
        <span>更多</span>
        <i-ep-arrowRight class="ml2 f12" />
      </span>
    </div>
    <!-- 内容 -->
    <div class="layout-col white-rgba50 rad8 p6">
      <div class="layout-col overlay p6">
        <div class="ww100 flex-col p12 bs3 " :class="i!=0?'mt10':''" v-for="(v, i) in !isNull(state.list)?state.list:[]" :key="i">
          <div class="ww100 flex-sc">
            <span class="w90"><span class="i1 plr10 ptb3 rad2 f12" style="background: #B7D4FF;">{{ v.source?v.source:'-' }}</span></span>
            <span class="flex1">{{ v.transaction_id?v.transaction_id:'-' }}</span>
            <span>{{ v.demand_type?v.demand_type:'-' }}</span>
          </div>
          <div class="bss33 ww100 pt1 mtb12"></div>
          <div class="ww100 flex-sc">

            <div class="flex-sc flex1">
              <span class="w90 c6">需求容量：</span>
              <span class="flex1">{{ v.market_demand_capacity?v.market_demand_capacity:'-' }} kW</span>
            </div>
            <div class="flex-sc flex1">
              <span class="w90 c6">市场单价：</span>
              <span class="flex1">{{ v.unit_price?v.unit_price:'-' }} 元</span>
            </div>
          </div>
          <div class="ww100 flex-sc mt5">
             <div class="flex-sc flex1">
              <span class="w90 c6">响应日期：</span>
              <span class="flex1">{{ v.response_date?parseTime(v.response_date, '{y}-{m}-{d}'):'-' }}</span>
            </div>
            <div class="flex-sc flex1">
              <span class="w90 c6">需求时段：</span>
              <span class="flex1">{{ v.demand_time_slot?v.demand_time_slot:'-' }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
    <!-- 底部 -->
    <aa-foot></aa-foot>
  </div>
</template>

<script lang="ts" setup>
import { parseTime } from '@/utils/common'

  const { proxy }:any = getCurrentInstance()
  const publicStore = proxy.publicStore()
	const configStore = proxy.configStore()
  const dictStore = proxy.dictStore()
  const state = reactive({
	  ...publicStore.$state,
  })

  watch(() => publicStore._public.demandData, async(val, old) => {
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
.bss33{
  background: linear-gradient(270deg, rgba(110, 158, 231, 0) 0%, #6E9EE7 100%);
}
</style>
