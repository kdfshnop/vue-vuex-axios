<!--
    @页面名称：代理商业务编辑审核是否通过
    @作者：豆亚东 (douyadong@lifang.com)
    @业务逻辑说明：
        1.代理商初次审核后再次编辑，仍需再次审核;
        2.        
-->
<template>
    <el-container>
        <el-main>    
            <bread-crumb :items="breadCrumb"></bread-crumb>
            <div class="hd">被编辑代理商公司：大锤字信息科技有限公司 <span class="skip">查看公司详情</span></div>
            <div>备注信息：</div>
            <div>为什么修改了，修改了哪些字段，此处文字来源为编辑时提交编辑弹出的二次确认弹窗所输入的相关备注信息</div>
           <el-table :data="searInfoList" border style="width: 100%;margin-top:50px;">
                <el-table-column type="index" label="序号" align="center" width="100px" ></el-table-column>
                <el-table-column prop="name" label="栏位" align="center" ></el-table-column>
                <el-table-column prop="address" label="字段" align="center" ></el-table-column>
                <el-table-column prop="address" label="原内容" align="center" ></el-table-column>
                <el-table-column prop="address" label="编辑后内容" align="center"></el-table-column>
            </el-table> 
            <div class="gap-2">
                <el-button style="width: 140px" type="danger" @click="handleReject">驳回</el-button>
                <el-button style="width: 140px" type="primary" @click="handleApprove">通过</el-button>
                <el-button style="width: 140px" type="" @click="handleClose">关闭</el-button>
            </div>
            <h2 class="gap-2">历史审核记录</h2>
            <el-table
                :data="historyRecords"
                stripe
                style="width: 100%; margin-bottom: 40px;">
                <el-table-column
                prop="commitDate"
                label="提交时间"
                width="180">
                </el-table-column>
                <el-table-column
                prop="verifyDate"
                label="审核时间"
                width="180">
                </el-table-column>
                <el-table-column
                prop="verifyStatus"
                label="审核状态">
                </el-table-column>
                <el-table-column
                prop="rejectReason"
                label="驳回理由">
                </el-table-column>
                <el-table-column
                prop="verifyPerson"
                label="审核人">
                </el-table-column>
            </el-table> 
            <el-dialog :title="title" :visible.sync="dialogVisible" width="30%">
                <textarea name="" id="" style="width:100%;" rows="10" placeholder="请添加备注" v-model="remark"></textarea>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="dialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="dialogVisible = false, submit()">确 定</el-button>
                </span>
            </el-dialog>
        </el-main>
    </el-container> 
</template>

<script>
import BreadCrumb from '@/components/common/BreadCrumb';

export default {
    name:'ProfessionAuditVerify',
    components:{BreadCrumb},
    data(){
        return {
            auditType:'1',//编辑审核
            breadCrumb: [{text:'业务审核'},{text: "编辑审核"},{text:'审核'}],
            historyRecords:[],// 历史审核记录
            searInfoList:[],
            dialogVisible:false,
            remark:'',
            title:'',
            id:this.$route.query.id//代理商公司Id;
        }
    },
    created(){
        // 编辑更改了哪些字段;
        this.$http.get(this.$apiUrl.professionAudit.detailContract+"?id="+this.id)
            .then(function(data){
                console.log('编辑详情')
            })
            .catch(function(err){
                console.log(err);
            });
        // 历史审核记录
        this.$http.get(this.$apiUrl.professionAudit.historyAudit+"?id="+this.id)
            .then(function(data){
                console.log('历史审核记录');
            })
            .catch(function(err){
                console.log(err);
            });
    },
    methods:{
        handleReject(){
            this.dialogVisible=true;
            this.title='驳回';
        },
        handleApprove(){
            this.dialogVisible=true;
            this.title='审核通过';
        },
        handleClose(){
            history.back();
        },
        submit(){
            let form={
                targetId:this.id,
                auditType:this.auditType,
                remark:this.remark
            };
            if(this.title=='驳回'){
                if(this.remark){
                    this.$http.post(this.$apiUrl.professionAudit.reject,form)
                        .then(function(data){
                            console.log('驳回');
                            this.remark="";
                        })
                        .catch(function(err){
                            console.log(err);
                        })
                }else{
                    this.$message.error('请填写驳回原因');
                }
                
            }else{
                this.$http.post(this.$apiUrl.professionAudit.pass,form)
                .then(function(data){
                    console.log('通过');
                    this.remark="";
                })
                .catch(function(err){
                    console.log(err);
                })
            }  
        }
    }
}
</script>

<style scoped>
    .hd{
        margin: 20px 0;
    }
    .skip{
        color:blue;
    }
</style>


