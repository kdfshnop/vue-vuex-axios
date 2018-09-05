<template>
    <el-dialog  :visible.sync="dialogVisible" width="50%">
        <el-form  label-width="120px">
            <el-form-item label="门店名称:">
                <span>{{form.name}}</span>
            </el-form-item>
            <el-form-item label="门店类型:">
                <span v-if="form.storeType==1">代理商直营门店</span>
                <span v-if="form.storeType==2">加盟门店</span>
            </el-form-item>
            <el-form-item label="代理商公司名称:" v-if="form.storeType==1" >
                <span>{{form.agencyName}}</span>
            </el-form-item>
            <el-form-item label="中介公司名称:" v-if="form.storeType==2" >
                <span>{{form.companyName}}</span>
            </el-form-item>
            <el-form-item label="门店地址:">
                <span>{{form.address}}</span>
            </el-form-item>
            <el-form-item label="城市-区域-板块:" v-if="form.cityName && form.districtIdName && form.townIdName">
                <span>{{form.cityName}}-{{form.districtIdName}}-{{form.townIdName}}</span>
            </el-form-item>
            <el-form-item label="经纬度:">
                <span>{{form.longlat}}</span>
            </el-form-item>
            <el-form-item label="附件:" v-if="form.resources && form.resources.length!=0">
              <ul ref="imgs" class="imgsForBrower">
                <li v-for="(file, ids) in resourcesFiles" :key="ids"><a :href="file.url">{{file.fileName}}</a></li>
                <li v-for="(img, index) in resourcesImgs" :key="index"><img :src="img.url" alt=""></li>
              </ul>
            </el-form-item>
            <el-form-item label="备注:" v-if="form.remark">
                <span>{{form.remark}}</span>
            </el-form-item>
        </el-form>
    </el-dialog>
</template>

<script>
import Viewer from "viewerjs/dist/viewer";
export default {
    name:'storeDetail',
    data(){
        return {
            dialogVisible:false,
            form: {
                address:'',//门店地址;
                agencyId:'',//代理商Id;
                cityId:'',//城市Id;
                companyId:'',//中介公司Id;
                districtId:'',//区域Id;
                longlat:'',//先经度后纬度;
                name:'',//门店名称;
                provinceId:'',//省份Id;
                storeType:'',//门店类型;
                townId:'',//板块Id;
                resources: [],
                remark: ''
            }
        }
    },
    methods:{
        open(storeId) {
            let self =this;
            // 获取门店详情;
            this.$http.get(this.$apiUrl.store.add+"?id="+storeId)
                .then(function(data){
                    self.form=data.data.data;
                    self.dialogVisible = true;
                    self.$nextTick(() => {
                      let ele = self.$refs.imgs;
                      ele ? new Viewer(ele,{zIndex: 5000}) : '';
                    });
                })
                .catch(function(err){
                    console.log(err);
                })
        }
    },
    computed: {
      resourcesImgs() {
        return this.form.resources && this.form.resources.filter((item) => {
          let itemurl = item.url;
          return itemurl.indexOf(".jpg")>0 || itemurl.indexOf(".jpeg")>0 || itemurl.indexOf(".bmp")>0 || itemurl.indexOf(".png")>0;
        })
      },
      resourcesFiles() {
        return this.form.resources && this.form.resources.filter((item) => {
          let itemurl = item.url;
          return itemurl.indexOf(".jpg")===-1 && itemurl.indexOf(".jpeg")===-1 && itemurl.indexOf(".bmp")===-1 && itemurl.indexOf(".png")===-1;
        })
      }
    },
}
</script>

<style>
.imgsForBrower {
  list-style: none;
  padding: 0;
  margin:0;
}
.imgsForBrower img {
  width: 80px;
}
</style>

