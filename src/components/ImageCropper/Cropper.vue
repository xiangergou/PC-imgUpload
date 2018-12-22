/*
 * @Author: FT.FE.Bolin
 * @Date: 2018-04-11 16:49:39
 * @Last Modified by: FT.FE.Bolin
 * @Last Modified time: 2018-09-20 14:20:01
 */

<template>
  <div class="dialog-cropper">
    <el-dialog
      :visible.sync="layer_cropper"
      title="图片裁剪"
      width="920px"
      @open="cropperedImgs=[]"
      @close="cropperClose">
      <div class="cropper-wrapper">
        <div
          v-for="(item,index) in cropperImgs"
          :key="index"
          :style="itemStyle"
          class="cropper-item"
        >
          <div
            :style="componentStyle"
            class="cropper-component">
            <vueCropper
              :ref="`refCropper${index}`"
              :img="item.img"
              :output-size="item.size"
              :output-type="item.outputType"
              :info="item.info"
              :can-scale="item.canScale"
              :auto-crop="item.autoCrop"
              :auto-crop-width="item.autoCropWidth"
              :auto-crop-height="item.autoCropHeight"
              :full="item.full"
              :fixed="item.fixed"
              :fixed-number="item.fixedNumber"
            />
          </div>
          <div class="cropper-btn--group clearfix">
            <el-input
              v-model="item.imageName"
              :maxlength="30"
              class="cropper-input--imageName"
              placeholder="请输入图片名称"
              size="small"
            />
            <el-button-group class="btn-group right">
              <el-tooltip
                class="item"
                effect="dark"
                content="替换图片"
                placement="top-start">
                <label
                  :for="`changeImage${index}`"
                  class="el-button el-button--primary el-button--small">
                  <i class="el-icon-upload"/>
                </label>
              </el-tooltip>
              <input
                :id="`changeImage${index}`"
                :accept="accept"
                type="file"
                @change="uploadImg($event, index)"
              >
              <el-tooltip
                class="item"
                effect="dark"
                content="放大图片"
                placement="top-start">
                <el-button
                  type="primary"
                  size="small"
                  icon="el-icon-zoom-in"
                  @click="changeScale(1, index)"/>
              </el-tooltip>
              <el-tooltip
                class="item"
                effect="dark"
                content="缩小图片"
                placement="top-start">
                <el-button
                  type="primary"
                  size="small"
                  icon="el-icon-zoom-out"
                  @click="changeScale(-1, index)"/>
              </el-tooltip>
              <el-tooltip
                class="item"
                effect="dark"
                content="逆时针旋转90°"
                placement="top-start">
                <el-button
                  type="primary"
                  size="small"
                  icon="el-icon-caret-left"
                  @click="rotateLeft(index)"/>
              </el-tooltip>
              <el-tooltip
                class="item"
                effect="dark"
                content="顺时针旋转90°"
                placement="top-start">
                <el-button
                  type="primary"
                  size="small"
                  icon="el-icon-caret-right"
                  @click="rotateRight(index)"/>
              </el-tooltip>
            </el-button-group>
          </div>
        </div>
      </div>
      <div
        slot="footer"
        class="dialog-footer">
        <el-button
          size="small"
          @click="layer_cropper = false">取 消</el-button>
        <el-button
          type="primary"
          size="small"
          @click="getCropData">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import VueCropper from 'vue-cropper'
export default {
  name: 'FhtImageCropper',
  components: {
    VueCropper
  },
  props: {
    cropperList: {
      type: Array,
      default: function () {
        return []
      }
    }
  },
  data () {
    return {
      accept: 'image/png, image/jpeg, image/jpg',
      defaultCropperOptions: {
        imageName: '',
        img: '',
        info: false,
        size: 1,
        outputType: 'png', // png解决裁剪后图片背景为黑色背景的问题，直接输出png吧
        canScale: true,
        autoCrop: true,
        // 开启宽度和高度比例
        fixed: true,
        fixedNumber: [4, 3],
        // 输出原图比例的截图，不至于大图裁剪出来的图太小啦
        full: true
      },
      layer_cropper: false,
      cropperImgs: [],
      cropperedImgs: []
    }
  },
  computed: {
    itemStyle () {
      const _width = this.cropperImgs.length > 1 ? 400 : 830
      return {
        width: _width + 'px'
      }
    },
    componentStyle () {
      const componentH = this.cropperImgs.length > 1 ? 300 : 600
      return {
        height: componentH + 'px'
      }
    }
  },
  watch: {
    cropperList (list = []) {
      this.cropperImgs = list.map((item) => {
        return {
          ...this.defaultCropperOptions,
          ...item
        }
      })
      this.layer_cropper = this.cropperImgs.length > 0
    }
  },
  created () {
    this.cropperImgs = this.cropperList.map((item) => {
      return {
        ...this.defaultCropperOptions,
        ...item
      }
    })
    this.layer_cropper = this.cropperImgs.length > 0
  },
  methods: {
    cropperClose () {
      this.cropperImgs = []
    },
    // 缩放图片
    changeScale (num, index) {
      this.$refs['refCropper' + index][0].changeScale(num)
    },
    // 旋转
    rotateLeft (index) {
      this.$refs['refCropper' + index][0].rotateLeft()
    },
    rotateRight (index) {
      this.$refs['refCropper' + index][0].rotateRight()
    },
    // 裁剪完成
    getCropData () {
      let countIndex = 0
      this.cropperImgs.forEach((item, index) => {
        this.$refs['refCropper' + index][0].getCropData((data) => {
          this.cropperedImgs.push({
            src: data,
            title: item.imageName
          })
          countIndex++
          if (countIndex === this.cropperImgs.length) {
            this.layer_cropper = false
            this.$emit('emitCropperData', this.cropperedImgs)
          }
        })
      })
    },
    // 替换图片
    uploadImg (e, index) {
      if (!e.target.value) {
        console.log('取消上传...')
        return false
      }
      const file = e.target.files[0]
      if (!this.accept.includes(file.type)) {
        this.$message.error(`请上传${this.accept.replace(/image\//gi, '')}的图片`)
        e.target.value = null
        return false
      }
      let reader = new FileReader()
      reader.onerror = error => {
        console.log(error + '读取异常....')
      }
      reader.onload = item => {
        const img = (typeof item.target.result === 'object')
          // 把Array Buffer转化为blob 如果是base64不需要
          ? window.URL.createObjectURL(new Blob([item.target.result]))
          : item.target.result
        let imageName = ''
        if (!file.name) {
          imageName = ''
        } else {
          imageName = file.name.split('.')[0].length <= 30
            ? file.name.split('.')[0]
            : file.name.split('.')[0].substr(0, 30)
        }
        this.cropperImgs[index].img = img
        this.$set(this.cropperImgs[index], 'img', img)
        this.$set(this.cropperImgs[index], 'imageName', imageName)
      }
      // 转化为base64
      reader.readAsDataURL(file)
      // 转化为blob
      // reader.readAsArrayBuffer(file)
      this.layer_cropper = true
      e.target.value = null
    }
  }
}
</script>

<style lang="scss" scoped>
  .cropper-wrapper{
  display: flex;
  flex-wrap: wrap;
  .cropper-item{
    width: 400px;
    margin: 0 20px;
    .cropper-component{
      height: 300px;
    }
    .cropper-btn--group{
      padding: 10px 0;
      display: flex;
      justify-content: space-between;
      .cropper-input--imageName{
        flex: 1;
      }
      .btn-group{
        margin-left: 10px;
      }
    }
  }
}
</style>
