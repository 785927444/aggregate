<template>
  <div class="layout-col bg-white">
    <!-- 标题 -->
    <div class="ww100 flex-sc plr20 pt15">
      <span class="card-title-home"></span>
      <span>用户负荷排名</span>
    </div>
    <!-- 内容 -->
		<div class="layout-col relative p20">
			<div class="rank_line"></div>
			<div class="flex-col ww100">
				<div class="relative ww100 h32 flex-sc mb10" v-for="(v, i) in !isNull(publicStore._public.rankChartData)?publicStore._public.rankChartData.sort((a, b) => b.load - a.load).slice(0, 3):[]" :key="i" >
					<div class="w32 h32 flex-cc bgi6 rad2 white">{{i+1}}</div>
					<div class="flex1 hh100 flex-col relative plr12">
						<div class="ww100 flex-bc f12">
              <span>{{v.station_name}}</span>
              <span class="c8">{{ v.value }}<span class="ml5">kW</span></span>
            </div>
						<div class="ww100 h10 flex-bc relative rad20 hidden mt5">
              <span class="ww100 hh100 bge"></span>
							<span class="hh100 absolute t0 l0 bgi6 z1 ww30" :style="{ width: `${v.rate}%` }"></span>
						</div>
					</div>
				</div>
			</div>
			<loopScroll :listData="!isNull(publicStore._public.rankChartData)?publicStore._public.rankChartData:[]" class="layout-col">      
				<div class="relative ww100 h32 flex-sc mb10" v-for="(v, i) in !isNull(publicStore._public.rankChartData)?publicStore._public.rankChartData.sort((a, b) => b.load - a.load).slice(3):[]" :key="i" >
					<div class="w32 h32 flex-cc bgi14 rad2 white">{{i+4}}</div>
					<div class="flex1 hh100 flex-col relative plr12">
						<div class="ww100 flex-bc f12">
              <span>{{v.station_name}}</span>
              <span class="c8">{{ v.value }}<span class="ml5">kW</span></span>
            </div>
						<div class="ww100 h10 flex-bc relative rad20 hidden mt5">
              <span class="ww100 hh100 bge"></span>
							<span class="hh100 absolute t0 l0 bgi14 z1 ww30" :style="{ width: `${v.rate}%` }"></span>
						</div>
					</div>
				</div>
			</loopScroll>
		</div>
  </div>
</template>

<script lang="ts" setup>
  import loopScroll from 'loop-scroll'
  const { proxy }:any = getCurrentInstance()
  const publicStore = proxy.publicStore()
	const configStore = proxy.configStore()
  const dictStore = proxy.dictStore()
  const state = reactive({
	  ...publicStore.$state,
  })
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
.t-30{ top: -15px;}
:deep(.el-carousel__container){
	height: 100%;
}
:deep(.el-carousel__arrow){
	background: rgba(28,128,245,0.2);
}
.rank_line{
  width: 4px;
  height: 100%;
  position: absolute;
  top: 24px;
  left: 24px;
  z-index: -1;
  opacity: 1;
  box-sizing: border-box;
  background: linear-gradient(to bottom, rgba(238, 246, 255, 1) 10%, rgba(238, 246, 255, 0) 100%);
}
</style>
