<template>
  <div class="ww100 flex-sc">
    <div class="flex1 p20 rad5 bg-white flex-sc">
      <img src="@/assets/images/jh/user.png" class="w42 h42" />
      <div class="flex1 flex-col ml20">
        <span class="mb5 f16 c8">接入用户总数</span>
        <span class="fw f20">{{ publicStore.data&&publicStore.data.userCount?publicStore.data.userCount:'0' }} 个</span>
      </div>
    </div>
    <div class="w15"></div>
    <div class="flex1 p20 rad5 bg-white flex-sc">
      <img src="@/assets/images/jh/load.png" class="w42 h42" />
      <div class="flex1 flex-col ml20">
        <span class="mb5 f16 c8">实时总负荷</span>
        <span class="fw f20">{{ publicStore.data&&publicStore.data.realTimeLoad?publicStore.data.realTimeLoad:'0' }} kW</span>
      </div>
    </div>
    <div class="w15"></div>
    <div class="flex1 p20 rad5 bg-white flex-sc">
      <img src="@/assets/images/jh/device.png" class="w42 h42" />
      <div class="flex1 flex-col ml20">
        <span class="mb5 f16 c8">接入设备总数</span>
        <span class="fw f20">{{ publicStore.data&&publicStore.data.deviceCount?publicStore.data.deviceCount:'0' }} 个</span>
      </div>
    </div>
    <div class="w15"></div>
    <div class="flex1 p20 rad5 bg-white flex-sc">
      <img src="@/assets/images/jh/capacity.png" class="w42 h42" />
      <div class="flex1 flex-col ml20">
        <span class="mb5 f16 c8">当前可调容量</span>
        <span class="fw f20">{{ publicStore.data&&publicStore.data.adjustCapacity?publicStore.data.adjustCapacity:'0' }} kW</span>
      </div>
    </div>
    <div class="w15"></div>
    <div class="flex1 p20 rad5 bg-white flex-sc">
      <img src="@/assets/images/jh/online.png" class="w42 h42" />
      <div class="flex1 flex-col ml20">
        <span class="mb5 f16 c8">设备在线率</span>
        <span class="fw f20">{{ publicStore.data&&publicStore.data.onlineRate?publicStore.data.onlineRate:'0' }} %</span>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
  const { proxy }: any = getCurrentInstance();
  const publicStore = proxy.publicStore();
  const configStore = proxy.configStore();
  const state = reactive({
    ...publicStore.$state,
  });
</script>

<style scoped lang="scss">

</style>