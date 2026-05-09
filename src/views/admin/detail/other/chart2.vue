<template>
  <div class="layout-col">
    <!-- 标题 -->
		<aa-title class="mt15" :title="(!isNull(publicStore._public.meas)&&state.code?find(publicStore._public.meas, ['code', state.code], 'name'):'数据趋势')">
      <template #right-content>
        <div class="ww100 flex-sc">
          <span class="ml10" v-if="state.num">(采样数：{{ state.num }})</span>
        </div>
      </template>
    </aa-title>
    <!-- 内容 -->
    <div class="flex1 layout-row bg-white rad8 ptb20 pl20 pr40 i15">
      <div class="ww100 hh100" id="Bar_Bars"></div>
    </div>
  </div>
</template>

<script lang="ts" setup>
  import ec from "@/utils/chart";
  const { drawChart } = ec();
  const route = useRoute();
  const { proxy }: any = getCurrentInstance();
  const publicStore = proxy.publicStore();
  const configStore = proxy.configStore();
  const state = reactive({
    ...publicStore.$state,
    sensor_id: '',
    code: '',
  });

  watch(() => publicStore.active, async(val, old) => {
    if(proxy.isNull(val)) return
    if(JSON.stringify(val) == JSON.stringify(old)) return
    await nextTick();
    state.code = route.query.code2
    if(!proxy.isNull(state.num)) state.num = ''
    if(!proxy.isNull(state.charts)) state.charts.forEach(v => { v.data = [] })
    // 获取设备
    state.sensor_id = [publicStore.active.id]
  }, { immediate: false, deep: true });

  watch(() => publicStore.model, async(val, old) => {
    if(JSON.stringify(val) == JSON.stringify(old)) return
    if(proxy.isNull(publicStore._public.sensors)) return
    await nextTick();
    init()
  }, { immediate: false, deep: true });

	const init = async() => {
    if(proxy.isNull(publicStore._public.sensors)) return
    if(proxy.isNull(state.sensor_id) || proxy.isNull(state.code)) return
    if(!proxy.isNull(publicStore._public.meas)) state.value_type = proxy.find(publicStore._public.meas, ['code', state.code], 'value_type')
    let sensor_id = ``
    state.sensor_id.forEach((v, i) => {
      if(i == 0) sensor_id+= ("'" + v + "'")
      if(i != 0)  sensor_id+=  ("," + "'" + v + "'")
    });
    let query = {model: `t_sensor_measure_float_day`, args: `code='${state.code}' and sensor_id IN (${sensor_id})`, limit: 15*state.sensor_id.length, order: 'date desc'}
    if(!proxy.isNull(publicStore.datetime)){
      // let start = Date.parse(publicStore.datetime[0]) + ''
      // let end = Date.parse(publicStore.datetime[1]) + ''
      let start = publicStore.datetime[0].split(' ')[0]
      let end = publicStore.datetime[1].split(' ')[0]
      query = {model: `t_sensor_measure_float_day`, args: `code='${state.code}' and sensor_id IN (${sensor_id}) and date>='${start}' and date<='${end}'`, limit: 15, order: 'date desc'}
    }
    let charts = []
    let res = await publicStore.http( {Api: query})
    state.num = res.length
    state.sensor_id.forEach(v => {
      let chart = charts.find(a=>a.sensor_id == v)
      if(!chart){
        let sensor = publicStore._public.sensors.find(a=>a.id == v)
        let newchart = {sensor_id: v, name: sensor.name, data: []}
        let list = res.filter(a=>a.sensor_id == v)
        let timekey = 'date'
        let codekey = 'code'
        list.forEach(v => {
          let timestamp = v[timekey]
          let time = !isNaN(parseFloat(timestamp)) && isFinite(timestamp)? proxy.parseTime(timestamp, format) : timestamp.split('T')[0].slice(5)
          const data = [time, Number(v.value), timestamp]
          newchart.data.unshift(data)
        })
        charts.push(newchart)
      }
    })
    state.charts = charts
	}

  watch(() => state.charts, async(val, old) => {
    if(proxy.isNull(val)) return
    if(JSON.stringify(val) == JSON.stringify(old)) return
    await nextTick()
    setChart(val)
  }, { immediate: false, deep: true });

  async function setChart(data){
    let chart = 'Bars'
    let id = 'Bar_Bars'
    let attr = {
      min: 0, 
      // max: 100,
      extent: 4, 
      zoomEnd: 15,
      colorList: [],
      legendArr: [],
      legendShow: true,
      labelShow: false,
      areaShow: true,
      y_name: '数值',
      tool_dw: '', 
      tool_name_x: '日期：', 
      tool_name_y: '数值：', 
    }
 		let series =[]
    data.forEach(v => {
			attr.legendArr.push(v.name)
			let data = v.data
			series.push({data})
		})
    data.forEach(v => {
			let data = v.data
			series.push({data})
		})
    drawChart(chart, id, attr, {series: series, xAxis: {}}, {});
  }
</script>

<style scoped lang="scss">

</style>
