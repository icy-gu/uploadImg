
<template>
    <div class="image-editor" style="line-height: initial;position:relative">
        <div class="imgDiv" v-if="num > imgIds.length">
            <input 
                type="file" 
                accept="image/*" 
                ref="avatarInput" 
                @change="fileChange($event)" 
                :id="'fileinput1'+cageID" 
                class="fileinput" />
            <label class="fileImgLabel" :for="'fileinput1' + cageID">
                <slot name="content"><Icon type="camera" size="20"></Icon></slot>
            </label>
        </div>
        <!-- 图片裁剪 -->
        <Modal v-model="optionCrop.showCropedImage" @on-ok="cropOk" >
            <p slot="header">图片裁剪</p>
            <div class="cropper" style="width:80%; height:300px;margin:0 auto;">
                <img :id="'cropimg2'+cageID" alt="" >
            </div>
        </Modal>
    </div>
</template>

<script>
/**
 * num : 图片上传张数
 * aspectRatio ： 图片宽高比
 * minCropBoxWidth ： 裁剪框最小宽度
 * minCropBoxHeight：裁剪框最小高度
 * isCrop ： 是否需要裁剪
 * cropBoxResizable :   裁剪框能否改变大小
 * cropBoxMovable：是否可以拖动裁剪框
 */
import Cropper from 'cropperjs';
import './cropper.min.css';
import lrz from 'lrz';
export default {
    name: 'image-editor',
    data () {
        return {
            title : '',
            cropper2: {},
            imgIds:[],
            imgName:'',
            optionCrop: {   //裁剪
                showCropedImage: false,
                cropedImg: ''
            },
            file:{},                //存储上传文件信息
            cageID: this.generateGUID(),     //用于组件中id动态变量
        };
    },
    props:{
        url:{
            type:String,
            default:''
        },
        //裁剪框能否改变大小
        cropBoxResizable:{
            type:Boolean,
            default:true
        },
        num:{
            type:Number,
            default:1
        },
        aspectRatio:{
            type:Number,
            default:1
        },
        minCropBoxWidth:{
            type:Number,
            defaulr:100
        },
        // 裁剪框最小高度
        minCropBoxHeight:{
            type:Number,
            default:200
        },
        // 是否可以拖动裁剪框
        cropBoxMovable:{
            type:Boolean,
            default:true
        },
        //是否裁剪
        isCrop:{
            type:Boolean,
            default:false
        },
    },
    mounted () {
        let img2 = document.getElementById('cropimg2'+this.cageID);
        this.cropper2 = new Cropper(img2, {
            dragMode: 'move',   //  拖动模式
            viewMode :1,    //  显示模式：0,1,2,3
            restore: false, //  是否调整窗口大小后恢复裁剪区域
            center: false,   //  是否显示裁剪框中间的+
            minCropBoxWidth:this.minCropBoxWidth,     //裁剪框的最小宽度
            minCropBoxHeight:this.minCropBoxHeight,   //裁剪框的最小高度
            cropBoxResizable:this.cropBoxResizable,  //改变裁剪框大小
            cropBoxMovable:this.cropBoxMovable,    //拖动裁剪框
            aspectRatio :this.aspectRatio       //裁剪框比例
        });
    },
    methods: {
        //组件通信
        uploadChange(){
            this.$emit( 'uploadChange' , this.imgIds );
        },
        fileChange(el) {
            if(this.isCrop){
                this.$emit( 'openCropDialog' );
            }
            this.imgIds = [];
            var _this = this;
            if (!el.target.files[0].size) return;
            _this.file = el.target.files[0];
            this.imgName = _this.file.name;
            if( this.isCrop ==false ){
                if(el.target.files[0].size >= 5242880){
                    common.toastMsg("上传图片大于5M，请重新选择图片");
                    return;
                }else{
                    var image = new FormData()
                    image.append('avatar', this.$refs.avatarInput.files[0])
                    // var url = constGlobal.HostActivity + 'uploadImg';
                    http.apiPost(this.url, image, {
                        headers: {
                            "Content-Type": "multipart/form-data"
                        }
                    }).then( res =>{
                        if( res.status == 0 ){
                            this.imgIds.push({
                                relativePath : res.data.relativePath , 
                                absolutePath : res.data.absolutePath,
                                title : this.imgName
                            });
                            this.uploadChange()
                        }else{
                            common.toastMsg( res.message );
                        }               
                    });
                }
            }else{
                if(el.target.files[0].size >= 5242880){
                    common.toastMsg("上传图片大于5M，请重新选择图片");
                    return
                }else{
                    this.optionCrop.showCropedImage = true;
                    let reader = new FileReader();
                    reader.onload = () => {
                        this.cropper2.replace(reader.result);
                        reader.onload = null;
                    };
                    reader.readAsDataURL(_this.file); 
                    var image = new FormData()
                    image.append('avatar', this.$refs.avatarInput.files[0])
                }
            }
            el.target.value = "";
        },
        //生成GUID
        generateGUID() {
            var s = [];
            var hexDigits = "0123456789abcdef";
            for (var i = 0; i < 36; i++) {
                s[i] = hexDigits.substr(Math.floor(Math.random() * 0x10), 1);
            }
            s[14] = "4";  // bits 12-15 of the time_hi_and_version field to 0010
            s[19] = hexDigits.substr((s[19] & 0x3) | 0x8, 1);  // bits 6-7 of the clock_seq_hi_and_reserved to 01
            s[8] = s[13] = s[18] = s[23] = "-";

            var guid = s.join("");
            return guid;
        },
        //确认裁剪
        cropOk(){
            //canvas转换为dataURL
            var canvas=this.cropper2.getCroppedCanvas();
            var png = canvas.toDataURL('image/png');
            function dataURLtoFile(dataurl,filename) {
                var arr = dataurl.split(','), mime = arr[0].match(/:(.*?);/)[1],
                bstr = atob(arr[1]), n = bstr.length, u8arr = new Uint8Array(n);
                while(n--){
                    u8arr[n] = bstr.charCodeAt(n);
                }
                return new File([u8arr],filename, {type:mime});
            }
            //dataURL转换为blob
            var nameImg = new Date().getTime() + '.png';
            var file = dataURLtoFile(png,nameImg);
            let reader = new FileReader();
            reader.onload = () => {
                this.cropper2.replace(reader.result);
                reader.onload = null;
            };
            reader.readAsDataURL(file);
            // 压缩图片后上传
            lrz( file ,{
                quality:0.8
            } ).then( resData =>{
                // console.log( resData )
                // var url = constGlobal.HostActivity + 'uploadImg';
                http.apiPost(this.url, resData.formData, {
                    headers: {
                        "Content-Type": "multipart/form-data"
                    }
                }).then( res =>{
                    if( res.status == 0 ){
                        this.imgIds.push({
                            relativePath : res.data.relativePath , 
                            absolutePath : res.data.absolutePath,
                            title : this.imgName
                        });
                        this.uploadChange()
                    }else{
                        common.toastMsg( res.message );
                    }               
                });
            })

        },
    },
    
};
</script>
<style lang="scss">
    .imgDiv{
        border-radius: 3px;
        text-align: center;
        .fileinput{
            position:absolute;
            top:0px;
            left : 0px;
            right : 0px;
            bottom : 0px;
            width : 100%;
            height: 100%;
            opacity : 0;
            cursor: pointer;
        }
    }
    .cropper-crop-box{
        
    }
</style>

