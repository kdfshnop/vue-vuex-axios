<template>
    <CollapsePanel class="gap-2" :expand="true">
        <div align-left slot="header" class="clearfix">
            <span style="margin-right: 30px">悟空服务人员信息</span>            
            <el-button v-show="mode === 'edit' && status !== 'editing'" @click="handleEdit" type="primary" size="mini">编辑</el-button>         
            <el-button v-show="mode === 'edit' && status === 'editing'" @click="handleCancel" type="danger" size="mini">取消</el-button>         
            <el-button v-show="mode === 'edit' && status === 'editing'" @click="handleComplete" type="success" size="mini">完成</el-button>         
        </div>
        <el-form ref="form" :model="item" label-width= "180px">
            <el-row v-show="mode === 'create' || mode === 'edit' && status === 'editing'">
                <el-col :span="12">
                    <el-form-item label="BD人员姓名">                        
                        <el-select
                            v-model="bdInfo.id"                            
                            filterable                                                        
                            placeholder="请输入姓名"
                            @change="handleBdchange"
                            :loading="bdsLoading">
                            <el-option
                                v-for="item in bds"
                                :key="item.id"
                                :label="item.name"
                                :value="item.id">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="彩霞服务人员姓名">
                        <el-select
                            v-model="cxInfo.id"                            
                            filterable                                                    
                            placeholder="请输入姓名"                            
                            @change="handleCxchange"
                            :loading="cxsLoading">
                            <el-option
                                v-for="item in cxs"
                                :key="item.id"
                                :label="item.name"
                                :value="item.id">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="1对1落地指导">
                        <el-select
                            v-model="directorInfo.id"                                                     
                            filterable                                                    
                            placeholder="请输入姓名"                            
                            @change="handleDirectorchange"
                            :loading="directorsLoading">
                            <el-option
                                v-for="item in directors"
                                :key="item.id"
                                :label="item.name"
                                :value="item.id">
                            </el-option>
                        </el-select>
                    </el-form-item>
                </el-col>
                <el-col :span="12">                    
                    <el-form-item> 
                            {{bdInfo.department}} - {{bdInfo.title}}                       
                    </el-form-item>                    
                    <el-form-item> 
                            {{cxInfo.department}} - {{cxInfo.title}}                       
                    </el-form-item>                    
                    <el-form-item>      
                            {{directorInfo.department}} - {{directorInfo.title}}
                    </el-form-item>
                </el-col>
            </el-row>   
            <el-row v-show="mode === 'view' || mode === 'edit' && status === ''">
                <el-col :span="12">
                    <el-form-item label="BD人员姓名">                        
                        {{bdInfo.name}}
                    </el-form-item>
                    <el-form-item label="彩霞服务人员姓名">
                        {{cxInfo.name}}
                    </el-form-item>
                    <el-form-item label="1对1落地指导">
                        {{directorInfo.name}}
                    </el-form-item>
                </el-col>
                <el-col :span="12">
                    <el-form-item> 
                            {{bdInfo.department}} - {{bdInfo.title}}                       
                    </el-form-item>                    
                    <el-form-item> 
                            {{cxInfo.department}} - {{cxInfo.title}}                       
                    </el-form-item>                    
                    <el-form-item>      
                            {{directorInfo.department}} - {{directorInfo.title}}
                    </el-form-item>
                </el-col>
            </el-row>         
        </el-form>
    </CollapsePanel>
</template>
<script>
    // TODO: 根据mode不同分为创建、编辑和详情三种模式
    import CollapsePanel from '@/components/common/CollapsePanel';
    import {generateComputed} from "./_Utils";
    import {mapMutations} from "vuex";

    // 服务人员信息
    export default {
        name: "serviceStaffInfo",
        components: {CollapsePanel},
        props: {
            item: Object,
            mode: {
                type: String,
                default: 'view'
            }
        },
        data() {
            return {               
                bdsLoading: false,
                cxsLoading: false,
                directorsLoading: false,
                selectedBD: "",
                selectedCX: "",
                selectedDirector: "",
                status: '',
                innerItem: {},
                bds:[],
                cxs:[],
                directors:[],
                callback: [],             
            };
        },
        computed: {
            "bdInfo": generateComputed("bdInfo", "ServiceStaffInfo", "updateBdInfo"),
            "cxInfo": generateComputed("cxInfo", "ServiceStaffInfo", "updateCxInfo"),
            "directorInfo": generateComputed("directorInfo", "ServiceStaffInfo", "updateDirectorInfo")
        },
        methods: {
            s() {
                // 为了保证
                this.c = this.c || 0;
                this.c++;
                if(this.c == 3) {
                    for(let i = 0; i < this.callback.length; i++) {
                        this.callback[i]();
                    }                    
                }
            },
            getBDs() {    
                // if(keyword === '' || keyword == null) {
                //     return;
                // }
                this.$http.get(this.$apiUrl.common.employee,{
                    params: {
                        personType: 2,
                        name: null
                    }
                }).then((data)=>{                    
                    this.bds = data.data.data;
                    this.s();
                });
                           
            },
            getCXs() {
                this.$http.get(this.$apiUrl.common.employee,{
                    params: {
                        personType: 7,
                        name: null
                    }
                }).then((data)=>{
                    this.cxs = data.data.data;
                    this.s();
                });
            },
            getDirectors() {
                this.$http.get(this.$apiUrl.common.employee, {
                    params: {
                        personType: 8,
                        name: null
                    }
                }).then((data)=>{
                    this.directors = data.data.data;
                    this.s();
                });
            },
            handleEdit() {
                this.status = 'editing';
                this.innerItem = JSON.parse(JSON.stringify(this.$store.state.ServiceStaffInfo||{}));
            },
            handleComplete() {
                this.status = '';
                this.updateItem(this.innerItem);
            },
            handleCancel() {
                this.status = '';
            },

            handleBdchange(val) {
                let t = this.bds.filter((b)=>b.id == val);
                if(t && t.length) {
                    this.bdInfo.name = t[0].name;
                    this.bdInfo.department = t[0].department;
                    this.bdInfo.title = t[0].title;
                    this.bdInfo.id = t[0].id;

                    if(this.mode=='create'){
                        this.updateBdInfo(this.bdInfo);
                    }
                }
            },
            handleCxchange(val) {
                let t = this.cxs.filter((b)=>b.id == val);
                if(t && t.length) {
                    this.cxInfo.name = t[0].name;
                    this.cxInfo.department = t[0].department;
                    this.cxInfo.title = t[0].title;
                    this.cxInfo.id = t[0].id;

                    if(this.mode=='create'){
                        this.updateCxInfo(this.cxInfo);
                    }
                }
            },
            handleDirectorchange(val) {
                let t = this.directors.filter((b)=>b.id == val);
                if(t && t.length) {
                    this.directorInfo.name = t[0].name;
                    this.directorInfo.department = t[0].department;
                    this.directorInfo.title = t[0].title;
                    this.directorInfo.id = t[0].id;

                    if(this.mode=='create'){
                        this.updateDirectorInfo(this.directorInfo);
                    }
                }
            },
            validate(fn) {
                this.$refs.form.validate(fn);
            },

            ...mapMutations("ServiceStaffInfo", ['updateItem', 'updateBdInfo', 'updateCxInfo', 'updateDirectorInfo'])
        },
        created() {
            this.getBDs();
            this.getCXs();
            this.getDirectors();
        },
        watch: { 
            '$store.state.ServiceStaffInfo.bdInfo.id': function(val) {
                if(this.c == 3) {
                    this.selectedBD = val;    
                } else {
                    this.callback.push(()=>{
                        this.selectedBD = val;
                   });
                }                
            },
            '$store.state.ServiceStaffInfo.bdInfo': function(val) {
                if(this.c == 3) {
                    this.selectedBD = val.id;
                } else {
                    this.callback.push(()=>{
                        this.selectedBD = val.id;
                   });
                }                
            },
            '$store.state.ServiceStaffInfo.cxInfo.id': function(val) {
                if(this.c == 3) {
                    this.selectedCX = val;
                } else {
                    this.callback.push(()=>{
                        this.selectedCX = val;
                   });
                }                
            },
            '$store.state.ServiceStaffInfo.cxInfo': function(val) {
                if(this.c == 3) {
                    this.selectedCX = val.id;
                } else {
                    this.callback.push(()=>{
                        this.selectedCX = val.id;
                   });
                }                
            },
            '$store.state.ServiceStaffInfo.directorInfo.id': function(val) {
                if(this.c == 3) {
                    this.selectedDirector = val;
                } else {
                    this.callback.push(()=>{
                        this.selectedDirector = val;
                   });
                }                
            },
            '$store.state.ServiceStaffInfo.directorInfo': function(val) {
                if(this.c == 3) {
                    this.selectedDirector = val.id;
                } else {
                    this.callback.push(()=>{
                        this.selectedDirector = val.id;
                   });
                }                
            }          
        }

    }
</script>
<style scoped>
.el-select {
    width: 100%;
}
</style>