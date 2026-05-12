<template>
  <div class="layout-col bg-white">
    <!-- 标题 -->
    <div class="ww100 flex-sc plr20 pt15">
      <span class="card-title-home"></span>
      <span>总负荷曲线</span>
    </div>
    <!-- 图表 -->
    <div class="flex1 layout-row bg-white rad8 p20 i15">
      <div class="ww100 hh100" id="Line_line_1"></div>
    </div>
  </div>
</template>

<script lang="ts" setup>
  import ec from "@/utils/chart";
  const { drawChart } = ec();
  const { proxy }: any = getCurrentInstance();
  const publicStore = proxy.publicStore();
  const configStore = proxy.configStore();
  const state = reactive({
    ...publicStore.$state,
  });

  watch(() => publicStore._public.loadChartData, async(val, old) => {
    if(proxy.isNull(val)) return
    if(JSON.stringify(val) == JSON.stringify(old)) return
    await nextTick()
    setChart(val)
  }, { immediate: false, deep: true });

  async function setChart(data){
    let chart = 'Line'
    let id = 'Line_line_1'
    let attr = {
      min: 0,
      // max: 100,
      extent: 4,
      colorList: [],
      legendArr: [],
      legendShow: true,
      labelShow: false,
      areaShow: false,
      smooth: true,
      y_name: 'kW',
      tool_dw: '',
      tool_name_x: '日期：',
      tool_name_y: '数值：',
    }
    let datas = []
    let series = {data: []}
    series = data.map(v => {
      // 先排序、去重
      let tempDatas =  [...new Map([...v.data].sort((a, b) => a[2] - b[2]).map(item => [item[0], item])).values()]
      datas = [...datas, ...tempDatas]
      attr.legendArr.push(v.name)
      let data = tempDatas
      return {data}
    })
    let xAxis = []
    let tempDatas = datas.sort((a, b) => a[2] - b[2])
    tempDatas.forEach(v => {
      let x = xAxis.find(a=>a == v[0])
      if(!x) xAxis.push(v[0])
      if(v[1] < 0) delete attr.min
    })
    drawChart(chart, id, attr, {series: series, xAxis: {data: xAxis}}, {});
  }
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
