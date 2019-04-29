# imgupload

> 基于vue+element+sass实现的图片上传、裁剪、预览、下载等集多功能于一体的图片操作组件，常用于后端管理项目图片上传中。


本组件将上传的图片file转为base64，以期能在页面直接展示， 待裁剪等操作之后返回转换后的图片地址。

基于photoswipe、vue-cropper、vuedraggable， 站在前人肩膀整合而成。

![image](https://github.com/xiangergou/imgUpload-npm/blob/master/static/img.gif)
```js
<imgUpload
    :limitSize=1 // 图片限制大小 默认1M
    :isMultiple="true" // 是否开启多张上传， 默认多张
    :canDelete="true"  // 是否开启删除功能，默认开启
    :hasUploadImgList="imgList"  // 已上传的图片回显，将接口返回的图片地址以对象数组的格式传入。 {src: 'address', title: 'img.png'}
    @outputImg="getOutputImgList" // 上传后转成的base64位地址，可转为file
    @deleteImg="getDeleteImg" // 视图层面删除返回的图片
    :imgWidth='{width: 122, height: 122}' // 设置图片默认大小
  />
```



