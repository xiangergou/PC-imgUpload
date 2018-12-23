<template>
  <div class="excel_upload_container">
    <label
      class="el-button el-button--primary el-button--small"
      for="uploadImages">上传图片</label>
    <input
      id="uploadImages"
      :accept="accept"
      :multiple="multiple"
      type="file"
      @change="uploadImg($event)"
    >
  </div>
</template>

<script>
export default {
  name: 'UploadImage',
  props: {
    limitSize: {
      type: Number,
      default: () => 1
    },
    onSuccess: {
      type: Function,
      default () {
        return () => {}
      }
    },
    multiple: {
      type: Boolean,
      default () {
        return false
      }
    }
  },
  data () {
    return {
      accept: 'image/png, image/jpeg, image/jpg',
      cropperList: null
    }
  },
  methods: {
    /* 选择图片 */
    async uploadImg (e) {
      if (!e.target.value) {
        console.warn('取消上传...')
        return false
      }
      const uploadList = []
      const readFileAsync = file => new Promise(resolve => {
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
          resolve({
            img,
            imageName
          })
        }
        // 转化为base64
        reader.readAsDataURL(file)
        // 转化为blob 不知道啥错误，还是用base64吧
        // reader.readAsArrayBuffer(file)
      })

      const files = e.target.files
      for (let i = 0; i < files.length; i++) {
        if (!this.accept.includes(files[i].type)) {
          this.$message.error(`请上传${this.accept.replace(/image\//gi, '')}的图片`)
          e.target.value = null
          return false
        } else if (files[i].size / 1024 / 1024 > this.limitSize) {
          let msg = files.length === 1 ? '该' : `第${i + 1}`
          // this.$message.error(`第${i}图片大小超过${this.limitSize}M限制`)
          this.$notify({
            title: '警告',
            message: `${msg}图片大小超过${this.limitSize}M限制`,
            type: 'warning'
          })
          e.target.value = null
          continue
        }
        uploadList.push(await readFileAsync(files[i]))
      }

      this.cropperList = uploadList.map((item, kindex) => {
        return {
          img: item.img,
          imageName: item.imageName
        }
      })
      this.onSuccess && this.onSuccess({
        cropperList: this.cropperList
      })
      e.target.value = null
    }
  }
}
</script>

<style scoped>

</style>
