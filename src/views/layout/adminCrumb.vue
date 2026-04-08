<template>
  <div class="flex-sc plr15 h50 f14 ww100 ml1">
    <span class="flex1 flex-sc">
      <svg class="w18 h18 mr5" style="color: #6c6c6c;" viewBox="0 0 48 48" width="18" height="18">
        <g>
          <path d="M7 17L24 7L41 17V41H7V17Z" stroke="#6c6c6c" fill="transparent" stroke-width="4"></path>
          <path d="M20 28H28V41H20V28Z" stroke="#6c6c6c" fill="#6c6c6c" stroke-width="4"></path>
        </g>
      </svg>
      <span class="flex1">{{configStore.crumbList}}</span>
    </span>
    <span class="mr15" v-if="configStore.main || configStore.config.main"><span class="i21">主站：</span>{{configStore.main?configStore.main:configStore.config.main}}</span>
    <span class="mr5" v-if="configStore.stationIp"><span class="i21">当前：</span>{{configStore.stationIp}} - </span>
    <span class="flex-sc" v-if="!isNull(configStore.stations)&&!isNull(configStore.token)&&!isNull(configStore.user)">
      <span class="flex-sc" v-if="isNull(configStore.user.station_num) || (!isNull(configStore.user.role_id) && configStore.user.role_id==1)">
        <span>聚合用户：</span>
        <span class="w50x4">
          <el-select v-model="configStore.distributId" placeholder="请选择用户" style="width:100%" clearable filterable >
            <el-option label="全部" value="0" />
            <el-option v-for="v in configStore.stations" :key="v.id" :value="String(v.id)" :label="v.station_name" />
          </el-select>
        </span>
      </span>
    </span>
  </div>
</template>

<script lang="ts" setup>
  import { adminRoutes } from '@/router/routes'
  const { proxy }:any = getCurrentInstance()
  const publicStore = proxy.publicStore()
  const configStore = proxy.configStore()
  const state = reactive({
	  ...publicStore.$state,
    distributId: ''
  })
  const route = useRoute()
  const defaultProps = {
    children: 'children',
    label: 'name',
  } 

  // 在这里使用 watch 监听路由变化
  watch(() => route.path, (to, from) => {
    let crumb = ''
    proxy.parentNodes(configStore.isRoute?configStore.routes:adminRoutes, route.path, 'path').reverse().forEach(item => {
      crumb = crumb + item.name + ' / '
    })
    crumb  = crumb + route.name
    configStore.crumbList = crumb
  }, {immediate: true, deep: true});
  
</script>

<style lang="scss" scoped>

</style>
