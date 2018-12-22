/*
 * @Author: FT.FE.Bolin
 * @Date: 2018-04-11 17:11:13
 * @Last Modified by: FT.FE.Bolin
 * @Last Modified time: 2018-11-20 16:02:38
 */

<template>
  <div class="layout-container">
    <!-- 图片上传 -->
    <upload-image
      :multiple="true"
      :on-success="handleSuccess"
      :before-upload="beforeUpload"/>
    <!-- 图片预览排序 -->
    <div style="width: 500px; margin-top: 20px;">
      <Preview
        :pic-list="cropperData"
        :delete-icon="true"
        :disabled="false"
        :item-size="{width: 122, height: 122}"
        @emitDelete="emitDelete"/>
    </div>
    <!-- 图片裁剪 -->
    <ImageCropper
      :cropper-list="cropperList"
      @emitCropperData="emitCropperData"/>
  </div>
</template>
<script>
import UploadImage from '@/components/Upload/image'
import Preview from '@/components/Preview/Preview'
import ImageCropper from '@/components/ImageCropper/Cropper'
export default {
  name: 'ExampleImage',
  components: {
    UploadImage,
    Preview,
    ImageCropper
  },
  filters: {

  },
  data () {
    return {
      layer_cropper: false,
      cropperList: [],
      cropperData: []
    }
  },
  computed: {

  },
  methods: {
    beforeUpload (file) {
      const isLimit1M = file.size / 1024 / 1024 < 1
      if (!isLimit1M) {
        this.$message.warning('图片大小不能超过1M！')
        return false
      }
      return true
    },
    handleSuccess ({ cropperList }) {
      this.$message.success('上传成功')
      this.cropperList = cropperList
      this.layer_cropper = true
    },
    /* $emit */
    // 删除图片
    emitDelete (val) {
      this.cropperData = val
    },
    // 裁剪后图片列表
    emitCropperData (list = []) {
      this.cropperData = [...this.cropperData, ...list]
    }
  }
}
</script>
