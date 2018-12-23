/*
 * @Author: xiangting
 * @Date: 2018-12-23 12:23:50
 * @Last Modified by: xiangting
 * @Last Modified time: 2018-12-23 12:23:50
 */

<template>
  <div>
    <imgUpload
      :limitSize=1
      :isMultiple="true"
      :canDelete="true"
      :hasUploadImgList="imgList"
      @outputImg="getOutputImgList"
      @deleteImg="getDeleteImg"
      :imgWidth='{width: 122, height: 122}'
    />
  </div>
</template>

<script>

import imgUpload from './image'
export default {
  name: 'HelloWorld',
  components: {
    imgUpload
  },
  data () {
    return {
      imgList: [{
        title: 'HelloWorlddsad.png',
        src: 'http://www.17qq.com/img_qqtouxiang/69182556.jpeg'
      }]
    }
  },
  methods: {
    getOutputImgList (list) {
      console.log(list)
    },
    getDeleteImg (img) {
      console.log(img)
    }
  }
}
</script>

<style scoped>
</style>
