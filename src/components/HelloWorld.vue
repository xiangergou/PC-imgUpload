<template>
  <div>
    <imgUpload
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
