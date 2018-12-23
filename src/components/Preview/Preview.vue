/*
 * @Author: FT.FE.Bolin
 * @Date: 2018-04-11 16:50:08
 * @Last Modified by: xiangting
 * @Last Modified time: 2018-12-23 11:55:23
 */

<template>
  <draggable
    v-model="list"
    :options="dragOptions"
    element="ul"
    class="list-group"
    @start="startDrag"
    @end="endDrag">
    <transition-group
      :name="'flip-list'"
      class="previewItems"
      type="transition">
      <li
        v-for="(item, index) in list"
        :key="item.sortNum"
        :style="itemStyle"
        class="preview-item clearfix">
        <img
          :src="item.src"
          class="preview-img img-center">
        <span
          v-if="showImageName && (item.picTag || item.title)"
          class="preview-tags">{{ item.picTag || item.title }}</span>
        <span class="preview-item-actions">
          <span
            class="preview-item__item-preview"
            @click="handlePreview(index)">
            <i class="el-icon-zoom-in"/>
          </span>
          <span
            v-if="deleteIcon"
            class="preview-item__item-delete"
            @click="handleDelete(index,item)">
            <i class="el-icon-delete"/>
          </span>
        </span>
      </li>
    </transition-group>
  </draggable>
</template>
<script>
import draggable from 'vuedraggable'

/* 阻止原生dragale打开新页面 */
document.body.ondrop = function (event) {
  event.preventDefault()
  event.stopPropagation()
}

export default {
  name: 'FhtPreview',
  components: {
    draggable
  },
  props: {
    // 图片列表
    picList: {
      type: Array,
      default: function () {
        return []
      }
    },
    // 删除
    deleteIcon: {
      type: Boolean,
      default: false
    },
    // 图片尺寸
    itemSize: {
      type: Object,
      default () {
        return {}
      }
    },
    showImageName: {
      type: Boolean,
      default () {
        return false
      }
    },
    // 是否启用拖拽
    disabled: {
      type: Boolean,
      default: true
    }
  },
  data () {
    return {
      list: [],
      isDragging: false,
      delayedDragging: false
    }
  },
  computed: {
    itemStyle () {
      return {
        width: this.itemSize.width + 'px',
        height: this.itemSize.height + 'px'
      }
    },
    dragOptions () {
      return {
        animation: 150,
        group: 'description',
        ghostClass: 'ghost',
        disabled: this.disabled
      }
    }
  },
  watch: {
    picList: {
      immediate: true,
      handler: function (val) {
        this.list = (val || []).slice()
        this.list.map((item, index) => {
          item.sortNum = item.sortNum ? item.sortNum : Math.random()
          item.type = item.type || 1
          item.title = item.picTag || item.title || ''
        })
      }
    },
    isDragging (newValue) {
      if (newValue) {
        this.delayedDragging = true
        return
      }
      this.$nextTick(() => {
        this.delayedDragging = false
      })
    }
  },
  mounted () {

  },
  methods: {
    async handlePreview (index) {
      if (this.list.length === 1 && this.list[0].isnoPic) {
        this.$message.warning('友情提示：暂无图片')
        return false
      }
      const imgloadAsync = item => new Promise(resolve => {
        let _img = new Image()
        _img.src = item.src
        _img.onload = e => {
          item.w = _img.width || 800
          item.h = _img.height || 600
          resolve(item)
        }
      })
      const previewList = []
      for (let i = 0; i < this.list.length; i++) {
        previewList.push(await imgloadAsync(this.list[i]))
      }
      this.$preview.open(index, previewList)
    },
    handleDelete (index, item) {
      this.$confirm('是否删除?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.list.splice(index, 1)
        this.handleEmit(item)
      }).catch(() => {

      })
    },
    handleEmit (item) {
      item && this.$emit('emitDelete', [this.list, item])
    },
    startDrag () {
      this.isDragging = true
    },
    endDrag (e) {
      this.isDragging = false
      this.handleEmit()
    }
  }
}
</script>
<style lang="scss" scoped>
.pswp__caption__center {
  text-align: center
}

.pswp {
  z-index: 9999999;
}

.pswp__img {
  background: #fff;
}

.flip-list-move {
  transition: transform 0.5s;
}
.no-move {
  transition: transform 0s;
}

.list-group {
  display: initial;
  min-height: 0;
}

.previewItems {
  margin: 0;
  vertical-align: top;
  display: inline-block;
  .preview-item {
    overflow: hidden;
    background-color: #fff;
    border: 1px solid #c0ccda;
    border-radius: 6px;
    box-sizing: border-box;
    display: inline-block;
    width: 122px;
    height: 92px;
    margin: 0 8px 8px 0;
    transition: all .5s cubic-bezier(.55, 0, .1, 1);
    position: relative;
    &.ghost {
      opacity: .5;
    }
    .pic-imageName {
      font-size: 12px;
      line-height: 24px;
      overflow: hidden;
      white-space: nowrap;
      text-overflow: ellipsis;
      padding: 0 5px;
    }
    img {
      width: 100%;
      height: 100%;
    }
    .preview-tags {
      position: absolute;
      overflow: hidden;
      white-space: nowrap;
      text-overflow: ellipsis;
      bottom: 0;
      text-align: center;
      width: 100%;
      height: 30px;
      line-height: 30px;
      background: rgba(0, 0, 0, 0.5);
      color: #fff;
      font-size: 12px;
    }
    .preview-item-actions {
      position: absolute;
      width: 100%;
      height: 100%;
      left: 0;
      top: 0;
      cursor: move;
      text-align: center;
      color: #fff;
      opacity: 0;
      font-size: 20px;
      background-color: rgba(0, 0, 0, .5);
      transition: opacity .3s;
      display: flex;
      justify-content: center;
      align-items: center;
      span {
        cursor: pointer;
        &+span {
          margin-left: 15px;
        }
      }
      .preview-item__item-delete {
        position: static;
        font-size: inherit;
        color: inherit;
      }
    }
    &:hover {
      .preview-item-actions {
        opacity: 1;
      }
    }
  }
}

</style>
