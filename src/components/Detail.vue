<template>
  <el-dialog v-model="state.isFalse" title="补充信息" :before-close="onVisable" :draggable="true" width="60%">
    <div class="layout-col">
      <el-form ref="formRef" :model="state.area" :rules="state.ruleList" label-width="85px">
        <div class="flex-sc warp ww100">
          <el-form-item label="联系人员" prop="contact" class="ww50">
            <el-input size="large" v-model="state.form.contact" placeholder="请输入" style="width:100%" clearable filterable />
          </el-form-item>
          <el-form-item label="联系号码" prop="telephone" class="ww50">
            <el-input size="large" v-model="state.form.telephone" placeholder="请输入" style="width:100%" clearable filterable />
          </el-form-item>
          <el-form-item label="日前响应" prop="day_out_response" class="ww50">
            <el-select size="large" v-model="state.form.day_out_response" placeholder="请选择" style="width:100%" clearable filterable>
              <el-option v-for="(v, i) in values?values:[]" 
              :key="i" :label="v.name"  :value="v.value" />
            </el-select>        
          </el-form-item>
          <el-form-item label="日内响应" prop="day_in_response" class="ww50">
            <el-select size="large" v-model="state.form.day_in_response" placeholder="请选择" style="width:100%" clearable filterable>
              <el-option v-for="(v, i) in values?values:[]" 
              :key="i" :label="v.name"  :value="v.value" />
            </el-select>        
          </el-form-item>
          <el-form-item label="是否签约" prop="yn_sign" class="ww50">
            <el-select size="large" v-model="state.form.yn_sign" placeholder="请选择" style="width:100%" clearable filterable>
              <el-option v-for="(v, i) in values?values:[]" 
              :key="i" :label="v.name"  :value="v.value" />
            </el-select>        
          </el-form-item>
          <el-form-item label="最大上调容量" prop="up_adjustable" class="ww50">
            <el-input size="large" v-model="state.form.up_adjustable" placeholder="请输入" style="width:100%" clearable filterable />     
          </el-form-item>
          <el-form-item label="最大下调容量" prop="down_adjustable" class="ww50">
            <el-input size="large" v-model="state.form.down_adjustable" placeholder="请输入" style="width:100%" clearable filterable />     
          </el-form-item>
          <el-form-item label="所属行业" prop="industry_type" class="ww50">
            <el-select size="large" v-model="state.form.industry_type" placeholder="请选择" style="width:100%" clearable filterable>
              <el-option v-for="(v, i) in state.industrys?state.industrys:[]" 
              :key="i" :label="v.name"  :value="v.id" />
            </el-select>
          </el-form-item>
          <el-form-item v-for="(v, i) in state.area" :key="i" class="ww50"
          :label="i==0?'所属省份':i==1?'所属城市':i==2?'所属区县':''" 
          :prop="i + ''">
            <el-select size="large" v-model="state.area[i]" placeholder="请选择" style="width:100%" clearable filterable @change="init(i)">
              <el-option v-for="(vv, ii) in i==0?state.provinceList:i==1?state.cityList:i==2?state.districtList:[]" 
              :key="vv.properties.adcode.toString()" 
              :label="vv.properties.name" 
              :value="vv.properties.adcode.toString()" />
            </el-select>
          </el-form-item>
        </div>
      </el-form>
    </div>
    <template #footer>
      <div class="dialog-footer">
        <el-button size="large" type="info" class="bga white" @click="onVisable">取 消</el-button>
        <el-button size="large" type="info" class="bgi1 white" @click.stop="handleSubmit(formRef)">确 定</el-button>
      </div>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
  import axios from 'axios'
  import api from '../api'
  const { proxy }:any = getCurrentInstance()
  const publicStore = proxy.publicStore()
  const configStore = proxy.configStore()
  const emit = defineEmits(['init'])
  let formRef = $ref()
  let values = [
    {name: '否', value: 0},
    {name: '是', value: 1},
  ]
  const state = reactive({
	  ...publicStore.$state,
    ruleList: {
      // '0': [{ required: true, message: '请选择', trigger:['blur','change'] }],
      // '1': [{ required: true, message: '请选择', trigger:['blur','change'] }],
      // '2': [{ required: true, message: '请选择', trigger:['blur','change'] }],
    },
    industrys: [],
    area: [],
    provinceList: [],
    cityList: [],
    districtList: [],
  })
  const props = defineProps({
    state: {
      type: [Object, Array],
      default: ()=>{return {}}
    },
  })

  const onVisable = async (val?: any) => {
    state.isFalse = !state.isFalse
    if(!state.isFalse) return
    state.form = {...val}
    state.industrys = []
    state.area = []
    state.area = [state.form.province?state.form.province:'',state.form.city?state.form.city:'',state.form.district?state.form.district:'']
    init()
  }

  const init = async(i?: any) => {
    console.log('state.area', state.area)
    // 行业
    getIndustry(i)
    // 地区
    getArea(i)

  }

  // 行业
  const getIndustry = async(i?: any) => {
    let query = {model: 't_industry', args: `parent_id='0' and type='0'`}
    let res = await publicStore.http({Api: query})
    state.industrys = proxy.isNull(res)?[] : res
  }

  // 地区
  const getArea = (i?: any) => {
    if(i == 0) state.area[1] = ''
    if(i == 1) state.area[2] = ''
    getList('100000', 'provinceList')
    if(proxy.isNull(state.area[0])){
      state.cityList = []
      state.districtList = []
      state.area[0] = ''
      state.area[1] = ''
      state.area[2] = ''
      return
    }
    getList(state.area[0], 'cityList')
    if(proxy.isNull(state.area[1])){
      state.districtList = []
      state.area[1] = ''
      state.area[2] = ''
      return
    }
    getList(state.area[1], 'districtList')
    if(proxy.isNull(state.area[2])){
      state.area[2] = ''
      return
    }
  }

  const getList = async(adcode, remark) => {
    axios.get(`./china/${adcode}.json`).then(temp => {
      state[remark] = temp.data.features
      // console.log('list', state[remark])
    })
  }

  const getAreaName = (list, code) => {
    if (!list || !code) return ''
    const item = list.find(v => v.properties.adcode.toString() === code)
    return item ? item.properties.name : ''
  }

  const handleSubmit = (formEl) => {
    formEl.validate(async valid =>{
      // if(proxy.isNull(state.area[0])) return ElNotification({ title: '提示', message: '请选择省份', type: 'error' })
      const province_name = getAreaName(state.provinceList, state.area[0])
      const city_name = getAreaName(state.cityList, state.area[1])
      const district_name = getAreaName(state.districtList, state.area[2])
      let form = {
      "model": "t_station", 
      "list":[
        {
          id: state.form.id,
          contact_name: state.form.contact_name,
          contact_phone: state.form.contact_phone,
          day_out_response: state.form.day_out_response,
          day_in_response: state.form.day_in_response,
          sign: state.form.sign,
          up_adjustable: state.form.up_adjustable,
          down_adjustable: state.form.down_adjustable,
          industry: state.form.industry,
          administrative_areaid: `${province_name?province_name:''}${city_name?'-'+city_name:''}`,
          province: state.area[0], 
          city: state.area[1], 
          district: state.area[2],
          province_name: province_name,
          city_name: city_name,
          district_name: district_name
        }
      ]}
      console.log('form----', form)
      api.updApi(form).then((res:any) => {
        if(res.code == 200){
          ElNotification({ title: '提示', message: '操作成功', type: 'success' })
          emit('init')
          onVisable()
        }
      }).catch((err) => {
        ElNotification({ title: '提示', message: '操作失败', type: 'error' })
      })
	  }).catch((err:any) =>{
			state.loading = false
		  console.log('表单错误信息：', err);
	  })
  }

 
  // 暴露给父组件
  defineExpose({
    onVisable
  })
</script>

<style scoped lang="scss">

</style>
