# imgupload

> 基于vue+element+sass实现的图片上传、裁剪、预览、下载等集多功能于一体的图片操作组件。

![image](https://github.com/xiangergou/imgUpload-npm/blob/master/static/img.gif)
```js
<imgUpload
    :limitSize=1
    :isMultiple="true"
    :canDelete="true"
    :hasUploadImgList="imgList"
    @outputImg="getOutputImgList"
    @deleteImg="getDeleteImg"
    :imgWidth='{width: 122, height: 122}'
  />
```
