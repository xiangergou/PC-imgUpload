/*
 * @Author: xiangting
 * @Date: 2018-12-23 12:19:51
 * @Last Modified by: xiangting
 * @Last Modified time: 2018-12-23 12:19:51
 */

<template>
  <div class="layout-container">
    <!-- 图片上传 -->
    <upload-image
      :multiple="isMultiple"
      :on-success="handleSuccess"
      :limitSize="limitSize"
      />
    <!-- 图片预览排序 -->
    <div style="width: 500px; margin-top: 20px;">
      <Preview
        :pic-list="cropperData"
        :delete-icon="canDelete"
        :disabled="false"
        :showImageName="true"
        :item-size="imgWidth"
        @emitDelete="emitDelete"/>
    </div>
    <!-- 图片裁剪 -->
    <ImageCropper
      :cropper-list="cropperList"
      @emitCropperData="emitCropperData"/>
  </div>
</template>
<script>
import UploadImage from './Upload/image'
import Preview from './Preview/Preview'
import ImageCropper from './ImageCropper/Cropper'
export default {
  name: 'ExampleImage',
  components: {
    UploadImage,
    Preview,
    ImageCropper
  },
  props: {
    limitSize: {
      type: Number,
      default: () => 1
    },
    imgWidth: {
      type: Object,
      default: () => '{width: 122, height: 122}'
    },
    isMultiple: { // 单张或多张
      type: Boolean,
      default: () => true
    },
    hasUploadImgList: {
      required: false,
      type: Array,
      default: () => []
    },
    canDelete: {
      required: false,
      type: Boolean,
      default: true
    }
  },
  data () {
    return {
      layer_cropper: false,
      cropperList: [],
      cropperData: [...this.hasUploadImgList]
    }
  },
  computed: {

  },
  methods: {
    handleSuccess ({ cropperList }) {
      (cropperList.length > 1) && this.$message.success('上传成功')
      this.cropperList = cropperList
      this.layer_cropper = true
    },
    /* $emit */
    // 删除图片
    emitDelete (val) {
      this.cropperData = val[0]
      this.$emit('deleteImg', val[1])
    },
    // 裁剪后图片列表
    emitCropperData (list = []) {
      this.cropperData = [...this.cropperData, ...list]
    }
  },
  watch: {
    hasUploadImgList: {
      immediate: true,
      handler (val) {
        val.forEach(item => {
          item.hasUploaded = true
        })
      }
    },
    cropperData (list) {
      let arr = list.length > 0 && list.filter(item => item && !item.hasUploaded)
      this.$emit('outputImg', arr)
    }
  }
}
</script>
