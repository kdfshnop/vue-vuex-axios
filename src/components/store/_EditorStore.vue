<template>
    <el-dialog :title="title" :visible.sync="dialogVisible" width="50%">
        <el-form :model="form" :rules="rules" ref="form" label-width="120px">
            <el-form-item label="门店名称" prop="name" >
                <el-input v-model="form.name" :disabled="disabled"></el-input>
            </el-form-item>
            <el-form-item label="门店类型" prop="storeType">
                <el-select :disabled="disabled" v-model="form.storeType" filterable placeholder="请选择加盟或直营">
                        <el-option label="代理商直营门店" :value="1"></el-option>
                        <el-option label="加盟门店" :value="2"></el-option>
                </el-select>
            </el-form-item>
            <el-form-item label="代理商公司名称" prop="agencyId" v-if="form.storeType==1" >
                <el-select :disabled="disabled" v-model="form.agencyId" placeholder="请选择" @focus="agencyList" filterable :clearable="true">
                    <el-option
                        v-for="item in agencyInfoList"
                        :key="item.agencyId"
                        :label="item.agencyCompanyName"
                        :value="item.agencyId">
                    </el-option>
                </el-select>
            </el-form-item>
            <el-form-item label="中介公司名称" prop="companyId" v-if="form.storeType==2" >
                <el-select :disabled="disabled" v-model="form.companyId" placeholder="请选择" @focus="companyList"  filterable :clearable="true">
                    <el-option
                        v-for="item in companyInfoList"
                        :key="item.companyId"
                        :label="item.name"
                        :value="item.companyId">
                    </el-option>
                </el-select>
            </el-form-item>
            <el-form-item label="门店地址" prop="address">
                <el-input :disabled="disabled" v-model="form.address"></el-input>
            </el-form-item>
            <el-form-item class="city-dis" label="城市-区域-板块" prop="areaInfo">
                <el-row>
                  <el-col :span="10">
                      <territory :disabled="disabled" v-model="form.areaInfo" :startLevel='startLevel' :endLevel="endLevel"></territory>
                  </el-col>
                  <el-col :span="7">
                    <el-select :disabled="disabled" v-model="form.districtId" filterable placeholder="请选择">
                      <el-option
                        v-for="item in districtoptions"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value">
                      </el-option>
                    </el-select>
                  </el-col>
                  <el-col :span="7">
                    <el-select :disabled="disabled" v-model="form.townId" filterable placeholder="请选择">
                      <el-option
                        v-for="item in plateoptions"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value">
                      </el-option>
                    </el-select>
                  </el-col>
                </el-row>
            </el-form-item>
            <el-form-item label="经纬度" prop="longlat" placeholder="经度,纬度">
                <el-input :disabled="disabled" v-model="form.longlat" @blur="longlat"></el-input>
            </el-form-item>
            <el-form-item label="附件"  v-if="!disabled && dialogVisible">
                <upload v-if="form.resources.length>0" :fileList.sync='form.resources' multiple ref="formupload"></upload>
                <upload :fileList.sync='form.resources' v-else multiple ref="formupload"></upload>
            </el-form-item>
            <el-form-item label="附件" v-if="disabled && form.resources && form.resources.length!=0">
                <ul ref="imgs" class="imgsForBrower">
                  <li v-for="(file, ids) in resourcesFiles" :key="ids"><a :href="file.url">{{file.fileName}}</a></li>
                  <li v-for="(img, index) in resourcesImgs" :key="index"><img :src="img.url" alt=""></li>
              </ul>
            </el-form-item>
            <el-form-item label="备注">
                <el-input :disabled="disabled" type="textarea" v-model="form.remark" placeholder="请输入备注(不超过100字)" rows="5" maxlength="100"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="submitForm" type="primary" v-if="!disabled">保存</el-button>
            <el-button @click="resetForm">关闭</el-button>
        </span>
    </el-dialog>
</template>

<script>
import Region from '@/components/common/Region';
import Territory from '@/components/common/Territory';
import Upload from '@/components/common/Upload';
import Viewer from "viewerjs/dist/viewer";
export default {
    name:'editorStore',
    props:['storeId','title'],
    components:{Region,Territory,Upload},
    data(){
        return {
            startLevel:2,//二级联动城市传参
            endLevel:3,//二级联动城市传参
            dialogVisible:false,
            companyInfoList:[],
            longlatFormat:false,//经纬度格式
            form: {
                address:'',//门店地址;
                agencyId:'',//代理商Id;
                areaLinkage:[],//区域联动
                cityId: '',//城市Id;
                companyId:'',//中介公司Id;
                districtId:'',//区域Id;
                files:[],//上传文件;
                longlat:'',//先经度后纬度;
                name:'',//门店名称;
                provinceId:'',//省份Id;
                remark:'',//备注;
                storeType:'',//门店类型;
                townId:'',//板块Id;
                areaInfo:{
                  id: [],
                  name: ''
                },
                resources: [], // 附件
                remark: '' // 备注
            },
            // 必填设置
            rules: {
                companyId: [{ required: true, message: '请输入中介公司名称', trigger: 'blur' }],
                agencyId:[{ required: true, message: '请输入代理商公司名称', trigger: 'blur' }],
                "areaInfo": [{ required: true, message: '请输入城市区域板块', trigger: 'blur' }],
                address:[{ required: true, message: '请输入门店地址', trigger: 'blur' }],
                name:[{ required: true, message: '请输入门店名称', trigger: 'blur' }],
                storeType:[{ required: true, message: '请输入门店类型', trigger: 'blur' }],
                longlat:[{ required: true, message: '请输入经纬度', trigger: 'blur' }]
            },
            // 城市区域板块
            // startLevel:2,
            // endLevel:3,

            // 区域
            districtoptions: [
              // {  需要这两个字段的对象
              //   value: '选项1',
              //   label: '黄金糕'
              // }
            ],
            // 板块
            plateoptions: []
        }
    },
    created(){
        // 代理商列表;
        this.agencyList();
        // 中介公司列表;
        this.companyList();
    },
    methods:{
        open() {
            let self =this;
            if(this.title=='编辑门店' || this.title=='门店详情'){
                // 获取门店详情;
                this.$http.get(this.$apiUrl.store.add+"?id="+this.storeId)
                    .then(function(data){
                        try { // 可能会存在没有没有key的问题
                          Object.assign(self.form, data.data.data);
                        } catch(e) {
                          self.form = data.data.data;
                          self.form.areaInfo = {
                            id: [],
                            name: ''
                          }
                          console.log(e);
                        }
                        self.form.areaLinkage=[self.form.cityId,self.form.districtId,self.form.townId];
                        self.form.areaInfo = {};
                        self.form.areaInfo.id = [];
                        self.form.areaInfo.id.push(self.form.cityId);
                        self.form.areaInfo.name = self.form.cityName;
                        self.dialogVisible = true;
                        setTimeout(() => {
                          if(self.title=='门店详情') {
                            let ele = self.$refs.imgs;
                            ele ? new Viewer(ele,{zIndex: 5000}) : '';
                          }
                        }, 600);
                    })
                    .catch(function(err){
                        console.log(err);
                    })
            }else{
                this.form= {
                    address:'',//门店地址;
                    agencyId:'',//代理商Id;
                    // areaLinkage:[],//区域联动
                    cityId:'',//城市Id;
                    companyId:'',//中介公司Id;
                    districtId:'',//区域Id;
                    files:[],//上传资料;
                    longlat:'',//先经度后纬度;
                    name:'',//门店名称;
                    provinceId:'',//省份Id;
                    remark:'',//备注;
                    storeType:'',//门店类型;
                    townId:'',//板块Id;
                    areaInfo:{
                      id: [],
                      name: ''
                    },
                    resources: [],
                    remark: ''
                }
                this.districtoptions = [];
                this.plateoptions = [];
                this.dialogVisible = true;
                this.$nextTick(() => {
                  this.$refs.form.clearValidate();  // 清除验证
                })
            }
        },
        longlat(){
            let longlatArray=this.form.longlat.split(',');
            var reg = /^[0-9]+.?[0-9]*$/;
            if(longlatArray.length==2&&reg.test(longlatArray[0])&&reg.test(longlatArray[1])){
                this.longlatFormat=true;
            }else{
                this.longlatFormat=false;
                this.$message({
                    message: '经纬度格式不正确',
                    type: 'warning'
                });
            }
        },
        submitForm() {
            let self=this;
            this.longlat();
            this.$refs.form.validate((valid) => {
                if (valid) {
                    if(self.longlatFormat){
                        this.dialogVisible=false;
                        let areainfoId = self.form.areaInfo.id;
                        if(areainfoId && areainfoId.length>0){
                            self.form.cityId=areainfoId[areainfoId.length - 1];
                            // self.form.districtId=self.form.areaLinkage[1];
                            // self.form.townId=self.form.areaLinkage[2];
                        };
                        if(self.title=='编辑门店'){
                          self.$http.post(self.$apiUrl.store.add,self.form)
                            .then(function(data){
                                self.$message({
                                    message: '编辑成功',
                                    type: 'success'
                                });
                                self.$emit('editSuccess',self.form);
                            })
                            .catch(function(err){
                                self.$message({
                                    message: '编辑失败',
                                    type: 'error'
                                });
                            })
                        }else if(self.title=='增加门店'){
                            self.$http.put(self.$apiUrl.store.add,self.form)
                                .then(function(data){
                                    self.$message({
                                        message: '添加成功',
                                        type: 'success'
                                    });
                                    self.$emit('addSuccess',self.form);
                                })
                                .catch(function(err){
                                    self.$message({
                                        message: '添加失败',
                                        type: 'error'
                                    });
                                })
                                .finally(() => {  // 不管成败 清掉表单
                                    this.reset()
                                });
                        };
                    }else{
                        self.$message({
                            message: '经纬度格式不正确',
                            type: 'warning'
                        });
                    }
                    // 此处代码需要加在请求成功之后;
                    // this.$refs[formName].resetFields();
                } else {
                    self.$message({
                        message: '请填写必填信息',
                        type: 'warning'
                    });
                    return false;
                }
            });
        },
        resetForm() {
            // if(this.title !== "门店详情") this.$refs.form.resetFields();
            this.reset();
            this.dialogVisible=false;
        },
        // 代理商列表;
        agencyList(){
            let self=this;
            this.$http.get(this.$apiUrl.agent.list)
            .then(function(data){
                self.agencyInfoList=data.data.data;
            })
            .catch(function(err){
                console.log(err,'代理商列表失败');
            })
        },
        // 中介公司列表;
        companyList(){
            let self=this;
            // 中介公司列表;
            this.$http.get(this.$apiUrl.company.list)
                .then(function(data){
                    self.companyInfoList=data.data.data
                })
                .catch(function(err){
                    console.log(err,'中介公司列表err')
                })
        },
        // 重置表单
        reset() {
          try {
            this.$refs.form.resetFields();  // 清除输入
            this.$refs.form.clearValidate();  // 清除验证
            this.$refs.formupload && this.$refs.formupload.$refs.eluploadfiles.clearFiles(); //清除文件列表
          } catch(e) {
            console.log(e);
          }

        },
        changeDistrict() {  // 区域
          let areaInfo = this.form.areaInfo;
          areaInfo.name && this.$http.get("/region/list?cityId=" + areaInfo.id[areaInfo.id.length-1]).then((data) => {
            data.data.data.forEach((item) => {
              item.label = item.regionName;
              item.value = item.regionId;
            })
            this.districtoptions = data.data.data;
          })
        },
        changePlate() {
          let areaInfo = this.form.areaInfo;
          areaInfo.name && this.$http.get("/plate/list?cityId="+ areaInfo.id[areaInfo.id.length-1] +"&regionId=" + this.form.districtId ).then((data) => {
            data.data.data.forEach((item) => {
              item.label = item.NAME;
              item.value = item.id;
            })
            this.plateoptions = data.data.data;
          })
        }
    },
    computed: {
      disabled() {
        return this.title === "门店详情";
      },
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
    watch: {
      "form.areaInfo.name": function (newVlaue, old) {
        if(!newVlaue || old == undefined) {
          this.form.districtId = '';
          this.form.townId = '';
        }
        this.changeDistrict();
      },
      "form.districtId": function (newVlaue, old) {
        if(!newVlaue) {
          this.form.townId = '';
        }
        this.changePlate();
      },
    }
}
</script>

<style>
.el-input.is-disabled .el-input__inner {
  cursor: auto !important;
  color: #666;
  /* background: none;
  border: none; */
}
.el-textarea.is-disabled .el-textarea__inner {
  cursor: auto;
  color: #666;
  /* background: none;
  border: none; */
}
.el-input.is-disabled .el-input__icon {
  display: none
}
.imgsForBrower {
  list-style: none;
  padding: 0;
  margin:0;
}
.imgsForBrower img {
  width: 80px;
}
.el-col .el-select {
  width: 100% !important;
}
</style>

