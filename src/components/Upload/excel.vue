<template>
  <div class="excel_upload_container">
    <input
      ref="excel-upload-input"
      :accept="accept"
      class="excel-upload-input"
      type="file"
      @change="handleClick">
    <div
      class="drop_wrapper"
      @drop="handleDrop"
      @dragover="handleDragover"
      @dragenter="handleDragover">
      <div class="drop_wrapper__info">将excel拖拽至此处或者</div>
      <el-button
        :loading="loading"
        class="drop_wrapper__upload"
        size="mini"
        type="primary"
        @click="handleUpload">上传Excel</el-button>
    </div>
  </div>
</template>

<script>
import XLSX from 'xlsx'

export default {
  name: 'CompUploadExcel',
  props: {
    beforeUpload: {
      type: Function,
      default () {
        return () => {}
      }
    },
    onSuccess: {
      type: Function,
      default () {
        return () => {}
      }
    }
  },
  data () {
    return {
      loading: false,
      accept: '.xlsx, .xls, .csv',
      excelData: {
        header: null,
        results: null
      }
    }
  },
  methods: {
    generateData ({ header, results }) {
      this.excelData.header = header
      this.excelData.results = results
      this.onSuccess && this.onSuccess(this.excelData)
    },
    handleDrop (e) {
      e.stopPropagation()
      e.preventDefault()
      if (this.loading) {
        return false
      }
      const files = e.dataTransfer.files
      if (files.length !== 1) {
        this.$message.error('只支持上传单个Excel文件')
        return false
      }
      const rawFile = files[0]

      if (!this.isExcel(rawFile)) {
        this.$message.error(`请上传${this.accept}文件`)
        return false
      }
      this.upload(rawFile)
      e.stopPropagation()
      e.preventDefault()
    },
    handleDragover (e) {
      e.stopPropagation()
      e.preventDefault()
      e.dataTransfer.dropEffect = 'copy'
    },
    handleUpload () {
      this.$refs['excel-upload-input'].click()
    },
    handleClick (e) {
      const files = e.target.files
      const rawFile = files[0]
      if (!rawFile) {
        return false
      }
      if (!this.isExcel(rawFile)) {
        this.$message.error(`请上传${this.accept}文件`)
        return false
      }
      this.upload(rawFile)
    },
    upload (rawFile) {
      this.$refs['excel-upload-input'].value = null
      if (!this.beforeUpload) {
        this.readerData(rawFile)
        return false
      }
      const before = this.beforeUpload(rawFile)
      if (before) {
        this.readerData(rawFile)
      }
    },
    readerData (rawFile) {
      this.loading = true
      return new Promise((resolve, reject) => {
        const reader = new FileReader()
        reader.onload = e => {
          const data = e.target.result
          const fixedData = this.fixData(data)
          const workbook = XLSX.read(btoa(fixedData), { type: 'base64' })
          const firstSheetName = workbook.SheetNames[0]
          const worksheet = workbook.Sheets[firstSheetName]
          const header = this.getHeaderRow(worksheet)
          const results = XLSX.utils.sheet_to_json(worksheet)
          this.generateData({ header, results })
          this.loading = false
          resolve()
        }
        reader.readAsArrayBuffer(rawFile)
      })
    },
    fixData (data) {
      let o = ''
      let l = 0
      const w = 10240
      for (; l < data.byteLength / w; ++l) {
        o += String.fromCharCode.apply(null, new Uint8Array(data.slice(l * w, l * w + w)))
      }
      o += String.fromCharCode.apply(null, new Uint8Array(data.slice(l * w)))
      return o
    },
    getHeaderRow (sheet) {
      const headers = []
      const range = XLSX.utils.decode_range(sheet['!ref'])
      let C
      const R = range.s.r
      for (C = range.s.c; C <= range.e.c; ++C) {
        const cell = sheet[XLSX.utils.encode_cell({ c: C, r: R })]
        let hdr = '未知列名 ' + C
        if (cell && cell.t) {
          hdr = XLSX.utils.format_cell(cell)
        }
        headers.push(hdr)
      }
      return headers
    },
    isExcel (file) {
      return /\.(xlsx|xls|csv)$/.test(file.name)
    }
  }
}
</script>

<style scoped>
.excel-upload-input{
  display: none;
  z-index: -9999;
}
.excel_upload_container {
  margin: 0 auto;
}
.drop_wrapper{
  border: 2px dashed #bbb;
  height: 160px;
  font-size: 24px;
  border-radius: 5px;
  color: #bbb;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
