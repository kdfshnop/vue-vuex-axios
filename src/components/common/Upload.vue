<template>
<div>
    <el-upload
        ref="eluploadfiles"
        class="upload-demo"
        :action="url"
        :on-preview="handlePreview"
        :on-remove="handleRemove"
        :before-remove="beforeRemove"
        :on-success="handleSuccess"
        :on-change="handleChange"
        :before-upload="beforeUpload"
        :multiple="multiple"
        :limit="20"
        :on-exceed="handleExceed"
        :file-list="fl"
        :show-file-list="true"
        list-type="picture"
        >
        <el-button size="small" type="primary">{{ btnText || '点击上传'}}</el-button>
        <div slot="tip" class="el-upload__tip">{{ tipText || '支持扩展名：.rar .zip .doc .docx .pdf .jpg .jpeg .bmp .png，最大支持10M文件上传'}}</div>
    </el-upload>
    <div style="display: none" :id="id">
        <div v-for="(item, index) in innerFileList" :key="index">
            <img :src="item.url" v-if="isImg(item.url)">
        </div>
    </div>
</div>
</template>
<script>
/**
 * 封装上传组件，把一些通用的属性隐藏掉，比如上传接口
 */
import Vue from 'vue';
// import FileList from '@/components/common/FileList';
import Viewer from "viewerjs/dist/viewer"
export default {
    name: "upload",
    // fileValidator是一个函数，接收的是file对象，跟beforeUpload一样，返回是一个对象，包含status和message,status=1表示成功
    props: ['fileList', 'btnText', 'tipText', 'mode', 'status', 'multiple', 'limit', 'fileValidator'],// mode暂时没用到,status会是editing和空
    // components: {
    //     FileList
    // },
    data() {
        return {
            url: Vue.apiUrl.getFullUrl(Vue.apiUrl.upload),
            innerFileList: this.fileList && this.fileList.slice() || [],
            fl: (this.fileList && this.fileList.slice() || []).map(f=>{
                let tmp = Object.assign({}, f);
                tmp.name = tmp.fileName;
                return tmp;
            }),
            name: "",
            id: ""
        };
    },
    methods: {
        handlePreview(file) {
            if(this.isImg(file.name)){// 图片才会有预览
                let imgs = this.innerFileList.filter((f)=>{
                    return this.isImg(f.fileName);
                });
                for(let i = 0; i < imgs.length; i++) {
                    if(imgs[i].fileName == file.name) {
                        this.viewer.view(i);
                        break;
                    }
                }
            }else{
                window.open(file.url);
            }
        },
        isImg(src) {
            let reg = /(jpg|png|jpeg|gif|bmp)$/;
            return reg.test(src);
            // return true;
        },
        handleRemove(data) {
            for(let i = this.innerFileList.length - 1; i >= 0; i--){
                if(this.innerFileList[i].url == data.url || this.innerFileList[i].uid == data.uid){
                    this.innerFileList.splice(i, 1);
                    this.$emit('update:fileList', this.innerFileList);
                    break;
                }
            }
        },
        beforeRemove() {

        },
        handleExceed() {
            this.$message.warning("超过了限制");
        },
        handleChange() {

        },
        handleSuccess(data, data2) {
            this.innerFileList.push({
                uid: data2.uid,
                ...data.data
            });
            this.$emit('update:fileList', this.innerFileList);
        },
        beforeUpload(file) {
            // 默认只支持图片，可以自己通过扩展fileTypeValidator来支持别的类型
            // let ret = true;
            // if(this.fileValidator) {
            //     let result = this.fileValidator(file);
            //     if(result.status != 1) {// 1表示成功
            //         this.$message.error(result.message || "不支持此类型的文件");
            //         ret = false;
            //     }
            // }else{
            //     let isImg = file.type.indexOf('image') > -1;
            //     let isLt2M = file.size / 1024 / 1024 < 2;
            //      if(!isImg) {
            //         this.$message.error('只能上传图片哦^_^');
            //         ret = false;
            //     }
            //     if(!isLt2M) {
            //         this.$message.error('文件大小不能超过2M:-)');
            //         ret = false;
            //     }
            // }

            // this.name = file.name;
            // return ret;
            // 以上为旭阳所写;
            let res=true;
            let isImg = file.type.indexOf('image') > -1;
            let isPdf = file.type.indexOf('pdf') > -1;
            let name = file.name.toLowerCase();
            // let isTxt = name.indexOf('.txt') > -1;
            let isDoc = name.indexOf('.doc') > -1;
            let isDocx = name.indexOf('.docx') > -1;
            // let isXls = name.indexOf('.xls') > -1;
            // let isXlsx = name.indexOf('.xlsx') > -1;
            // let isPpt = name.indexOf('.ppt') > -1;
            // let isPptx = name.indexOf('.pptx') > -1;
            let isRar = name.indexOf('.rar') > -1;
            let isZip = name.indexOf('.zip') > -1;
            let isLt10M=file.size/1024/1024<10;
            if(!isImg && !isPdf && !isDoc && !isDocx && !isRar && !isZip){
                this.$message.error('不支持此类型的文件');
                res=false;
            };
            if(!isLt10M){
                this.$message.error('文件大小不能超过10M:-)');
                res=false;
            };
            this.name = file.name;
            return res;
        },
    },
    mounted() {
        this.$nextTick(()=> {
            this.viewer = new Viewer(document.getElementById(this.id), {zIndex: 5000})
        });
    },
    created() {
        this.id = Math.random().toString().replace('0.','');
    },
    updated() {
      console.log(111);
    },
    watch: {
        innerFileList: {
            handler(now, old) {
              if(this.viewer) {
                  console.log('fileList changed...');
                  this.viewer.destroy();
                  this.$nextTick(()=>{
                      this.viewer = new Viewer(document.getElementById(this.id), {zIndex: 5000});
                  })
              }else{
                  console.log('fileList changed');
                  this.$nextTick(()=>{
                      this.viewer = new Viewer(document.getElementById(this.id), {zIndex: 5000});
                  });
              }
            },
            deep:true
        },
        "fileList": {
          handler() {
            this.innerFileList = this.fileList && this.fileList.slice() || [];
          },
          deep:true
        }
    }
}
</script>
<style>

</style>


