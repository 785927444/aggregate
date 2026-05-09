<template>
  <!-- 设备详情 -->
  <div class="layout-row hidden">
    <!-- 设备列表 -->
    <div class="ww100 hidden flex1">
      <datas class="ww100 hidden flex1" ref="initRef" />
    </div>
    <!-- 数据分析 -->
    <div class="ww100 hidden flex5 flex-col ml15">
      <!-- 标题 -->
      <aa-title title="数据分析">
        <template #right-content>
            <div class="ww100 flex-ec i15">
              <div class="flex-sc ml15">
                <span class="pl10">时间：</span>
                <span class="w50x8 flex-sc">
                <el-date-picker style="width:100%;" v-model="publicStore.datetime" format="YYYY-MM-DD HH:mm:ss" value-format="YYYY-MM-DD HH:mm:ss"
                type="datetimerange" range-separator="-" start-placeholder="开始时间" end-placeholder="结束时间" @change="publicStore.status=false;initRef.init()" />
                </span>
              </div>
              <div class="flex-sc ml15">
                <span class="rad4 ptb5 plr8 flex-cc cursor bgi1 white" @click.stop="router.go(-1)">返回</span>
              </div>
            </div>
        </template>
      </aa-title>
      <!-- 内容 -->
      <div class="layout-row hidden">
        <data1 class="flex2 hh100 hidden mr15" />
        <div class="flex7 layout-col">
          <data2 class="hh100 hidden flex1" @init="publicStore.status=false;initRef.init()"/>
          <chart2 v-if="route.query&&route.query.code2" class="hh100 hidden flex1" />
          <!-- <alarm v-else class="hh100 hidden flex1" /> -->
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
  import datas from './datas'
  import data1 from './data1'
  import data2 from './data2'
  import alarm from './alarm'
  import chart2 from './other/chart2'
  const router = useRouter()
  const route = useRoute()
  const { proxy }: any = getCurrentInstance()
  const publicStore = proxy.publicStore()
  const state = reactive({
    ...publicStore.$state,
  });
  let initRef = $ref()
</script>

<style lang="scss">

</style>
