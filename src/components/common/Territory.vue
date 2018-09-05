<!--
    @组件名称：区域搜索
    @作者：豆亚东 (douyadong@lifang.com)
    @业务逻辑说明：
        1.
        2.
-->
<template>
    <div>
        <el-select
            :disabled="disabled"
            v-model="areaDress"
            filterable
            remote
            clearable
            reserve-keyword
            :placeholder="keyWords"
            :remote-method="remoteMethod"
            :loading="loading" @change="handleChange" style="width:100%;" value-key="name" @focus="clearOption">
            <!--上面的value-key是当取值为对象时必须！！！-->
            <el-option
                v-for="(item,index) in options"
                :key="item.name"
                :label="item.name"
                :value="item">
            </el-option>
        </el-select>
    </div>
</template>

<script>
export default {
    name:'territory',
    props:{
        "startLevel":{
            type:Number,
            default:2//默认省级 1国家、2省份、3城市、4区域、5板块
        },
        "endLevel":{
            type:Number,
            default:4//默认区级
        },
        "placeholder":String,
        "value":{},//v-module默认值;
        "disabled": {
          type: Boolean,
          default: false
        }
    },
    data(){
        return {
            loading:false,
            areaDress:this.value,//获取的区域;
            options:[],//搜索获取的选项;
            keyWords:this.placeholder||'请输入关键字'
        }
    },
    created(){
        if(this.value&&JSON.stringify(this.value) !== "{}"){
            this.options=[{
                id:this.areaDress.id,
                name:this.areaDress.name
            }];
        }
    },
    methods:{
        clearOption(){
            this.options=[];
        },
        clear(){
            this.areaDress={};
        },
        remoteMethod(query){
            // query为输入的关键字;
            if(query==''){
                this.options=[];
            }
            if(query!==''){
                this.loading=true;
                let self=this;
                // 发送请求,参数为query,startLevel,endLevel,获取查询结果,options赋值搜索结果,loading=false;
                this.$http.get(this.$apiUrl.area.search+'?name='+query+'&startLevel='+this.startLevel+'&endLevel='+this.endLevel)
                    .then(function(data){
                        let ret=data.data.data;
                        self.options=[];//预先清空;
                        // 对联动级别进行处理(市级);
                        if(self.endLevel==3){
                            for(var i in ret){
                                self.options.push({
                                    id:[ret[i].provinceId,ret[i].cityId],
                                    name:ret[i].provinceName+'-'+ret[i].cityName
                                });
                            }
                        };
                        // 对区级联动处理;
                        if(self.endLevel==4){
                            for(var i in ret){
                                //因为区级会查询到上级城市,表示代理整个城市;
                                if(ret[i].regionName){
                                    self.options.push({
                                        id:[ret[i].provinceId,ret[i].cityId,ret[i].regionId],
                                        name:ret[i].provinceName+'-'+ret[i].cityName+'-'+ret[i].regionName
                                    });
                                }else{
                                    self.options.push({
                                        id:[ret[i].provinceId,ret[i].cityId],
                                        name:ret[i].provinceName+'-'+ret[i].cityName,
                                    });
                                }
                            }
                        };
                        // 对板块联动处理;
                        if(self.endLevel==5){
                            for(var i in ret){
                                self.options.push({
                                    name:ret[i].provinceName+'-'+ret[i].cityName+'-'+ret[i].regionName+'-'+ret[i].townName,
                                    id:[ret[i].provinceId,ret[i].cityId,ret[i].regionId,ret[i].townId],
                                });
                            }
                        };
                        self.loading=false;
                    })
                    .catch(function(err){
                        console.log(err);
                    })
            }
        },
        handleChange(val){
            console.log(val,'最终地区取值');
            this.options=[];
            // 此处是利用v-model双向绑定传值;
            this.$emit('input',val);
        }
    },
    watch:{
        // 监听变化，比如重置效果;
        "value":{
            handler(val){
                this.areaDress=val;
                this.options=[{
                    id:this.areaDress.id,
                    name:this.areaDress.name
                }];
            },
            deep:true
        },
        
    }
}
</script>

<style>

</style>


