<template>
  <el-container class="window flex-col">
    <admin-title />
    <div class="layout-row">
      <admin-aside class="mt1" />
      <div class="layout-col">
        <admin-crumb />
        <router-view class="plr15 pb15"/>
      </div>
    </div>
  </el-container>
</template>

<script lang="ts" setup>
  import adminTitle from '@/views/layout/adminTitle'
  import adminAside from '@/views/layout/adminAside.vue'
  import adminCrumb from '@/views/layout/adminCrumb.vue'
  const { proxy }:any = getCurrentInstance()
  const publicStore = proxy.publicStore()
  const configStore = proxy.configStore()
  const state = reactive({
	  ...publicStore.$state,
  })
</script>

<style lang="scss" scoped>

</style>
