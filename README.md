# uploadImg
### 用于PC端的图片上传组件，包含裁剪、压缩功能

## 需要安装的插件

#### 1、cropperjs(图片裁剪)
` npm install cropperjs `

#### 2、lrz(图片压缩)
` npm install lrz `

## API

属性|说明|类型|默认值
---|:--:|:--:|---:
url|上传地址|String|
num|图片上传张数|Number|1
aspectRatio|图片宽高比|Number|1
minCropBoxWidth|裁剪框最小宽度|Number|100
minCropBoxHeight|裁剪框最小高度|Number|200
isCrop|是否需要裁剪|Boolean|false
cropBoxResizable|裁剪框能否改变大小|Boolean|true
cropBoxMovable|是否可以拖动裁剪框|Boolean|true

## Events

事件名|说明|返回值
---|:--:|---:
uploadChange|图片上传成功后的回调|imgList

## Slot

名称|说明
---|---:
content|图片上传插件中内容的插槽

### 代码示例
```
<upload-img 
    :url="uploadUrl"
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
```

- 具体代码见demo.vue
