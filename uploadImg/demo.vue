<template>
    <div>
        <div style="width:100px;height:100px" v-if="imgid !==''">
            <img :src="imgid " alt="" style="width:100%">
        </div>
        <upload-img 
            :num=1 
            :aspectRatio=1
            :minCropBoxWidth=100
            :is-crop="true"
            :cropBoxResizable="true"
            v-on:uploadChange ="uploadChangeEvent"
            >
            <div slot="content" style=" width: 100px;height: 100px; border: 1px dashed #ccc;display:flex;justify-content:center;align-items:center">
                <Icon type="camera" style="font-size:18px;"></Icon>
            </div>
        </upload-img>
    </div>
</template>
<script>
/**
 * num : 图片上传张数
 * aspectRatio ： 图片宽高比
 * minCropBoxWidth ： 图片最小宽度
 * isCrop ： 是否需要裁剪
 * cropBoxResizable :   裁剪框能否改变大小
 */
import uploadImg from '../uploadImg/uploadImg.vue'
export default {
    data(){
        return{
            imgid:'',
            imgName:''
        }
    },
    components:{uploadImg},
    mounted(){},
    methods:{
        //获取图片
        uploadChangeEvent(imgobj){
           this.imgid = imgobj[0].absolutePath
        },
    }
}
</script>

