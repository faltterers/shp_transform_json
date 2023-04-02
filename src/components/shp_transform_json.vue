<!--
 * @Descripttion: 
 * @version: 
 * @Author: flatterer
 * @Date: 2023-03-30 10:51:00
 * @LastEditors: flatterer
 * @LastEditTime: 2023-03-31 15:17:32
-->
<template>
    <div class="upload_demo">
        <el-upload
                   drag
                   :auto-upload=false
                   action=""
                   accept="shp"
                   :on-preview="handlePreview"
                   :on-remove="handleRemove"
                   :limit="1"
                   :on-change="bind"
        >
            <i class="el-icon-upload"></i>
            <div class="el-upload__text">将shp文件拖到此处，或<em>点击配置</em></div>

            <div class="el-upload__tip" slot="tip">必须是shp文件</div>
        </el-upload>
        <el-button style="margin-left: 10px;" size="small" type="success" @click="config">生成并导出GeoJson数据</el-button>
    </div>
</template>

<script>
    import {open} from "shapefile"
    import FileSaver from "file-saver";
    export default {
        name: "Config",
        data(){
            return{
                file:{},
                newarray:{
                    "type": "FeatureCollection",
                    "features": []
                }
            }
        },
        methods:{
            config() {
                // console.log(this.file)
                //判断文件是否已经加载
                if(JSON.stringify(this.file) == "{}"){
                    this.$alert('请先选择文件', {
                        confirmButtonText: '确定'
                    })
                }
                //判断文件是否为shp文件
                const name=this.file.name
                const extension=name.split('.')[1]
                // console.log(extension)
                if('shp'!==extension){
                    this.$alert('文件不是shp文件！请重新选择文件', {
                        confirmButtonText: '确定'
                    })
                }else {
                    //后面实现回调函数 这里需要把this重新载入一下
                    let _this=this
                    const reader=new FileReader()
                    const  fileData=this.file.raw //element-ui的坑点 是this.flie.raw
                    reader.readAsArrayBuffer(fileData)
                    reader.onload = function(e){
                        open(this.result)
                            .then(source => source.read()
                                .then(function log(result) {
                                    //把转换到的数据分别塞入数组
                                    const json = result.value
                                    // console.log(json)
                                    _this.newarray.features.push(json)
                                    // console.log(_this.newarray)
                                    
                                    //判断是否已经全部转换完成，都完成了再导出文件
                                    if (result.done){
                                        // console.log(result)
                                        // console.log(_this.newarray)
                                        const content = JSON.stringify(_this.newarray);//将object转换成json字符串塞入文件
                                        // console.log(content)
                                        const blob = new Blob([content], { type: "" });
                                        FileSaver.saveAs(blob, _this.file.name.split('.')[0]+'.json');
                                    return
                                    }
                                    // console.log(this.newarray);
                                    return source.read().then(log);
                                }))
                            .catch(error => console.error(error.stack));
                    }
                };

            },
            handleRemove(file, fileList) {
                //console.log(file, fileList);
            },
            handlePreview(file) {
                console.log(file);
            },
            bind(files, fileList){
                //绑定文件
                this.file=fileList[0]
                //console.log(this.file)
            },

        }
    }
</script>

<style scoped>
    .upload_demo{
        text-align: center;
        margin-top: 50px;
    }
    .el-button{
       margin-top: 10px;
    }
</style>
