/*
 * @Author: FT.FE.Bolin
 * @Date: 2018-04-11 16:50:01
 * @Last Modified by: xiangting
 * @Last Modified time: 2018-12-23 12:09:44
 */

<template>
  <div
    class="pswp"
    tabindex="-1"
    role="dialog"
    aria-hidden="true">
    <div class="pswp__bg"/>
    <div class="pswp__scroll-wrap">
      <div class="pswp__container">
        <div class="pswp__item"/>
        <div class="pswp__item"/>
        <div class="pswp__item"/>
      </div>
      <div class="pswp__ui pswp__ui--hidden">
        <div class="pswp__top-bar">
          <div class="pswp__counter"/>
          <button
            class="pswp__button pswp__button--close"
            title="关闭 (Esc)"/>
          <a
            v-if="pswpOptions.downloadEl"
            :href="downloadImageUrl"
            :download="downloadImageName"
            target="_self"
            class="fht__download el-icon-download"
            title="下载图片"/>
          <span
            class="fht__download el-icon-caret-right"
            title="旋转图片"
            @click="transformRotate()"/>
          <button
            class="pswp__button pswp__button--share"
            title="分享"/>
          <button
            class="pswp__button pswp__button--fs"
            title="全屏展示"/>
          <button
            class="pswp__button pswp__button--zoom"
            title="放大/缩小"/>
          <div class="pswp__preloader">
            <div class="pswp__preloader__icn">
              <div class="pswp__preloader__cut">
                <div class="pswp__preloader__donut"/>
              </div>
            </div>
          </div>
        </div>
        <div class="pswp__share-modal pswp__share-modal--hidden pswp__single-tap">
          <div class="pswp__share-tooltip"/>
        </div>
        <button
          class="pswp__button pswp__button--arrow--left"
          title="上一张"/>
        <button
          class="pswp__button pswp__button--arrow--right"
          title="下一张"/>
        <div class="pswp__caption">
          <div class="pswp__caption__center"/>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import PhotoSwipe from 'photoswipe/dist/photoswipe'
import UI from 'photoswipe/dist/photoswipe-ui-default'
export default {
  data () {
    return {
      defaultOptions: {
        captionEl: true,
        bgOpacity: 0.8,
        fullscreenEl: true,
        history: false,
        shareEl: false,
        tapToClose: true,
        /* 下载按钮显示 */
        downloadEl: true
      },
      pswpOptions: {},
      /* 下载图片地址、名称 */
      downloadImageUrl: '',
      downloadImageName: '',
      // 当前图片旋转次数
      rotateCount: 0
    }
  },
  methods: {
    open (index, list, params = this.defaultOptions) {
      this.rotateCount = 0
      this.pswpOptions = Object.assign({
        index: index,
        getThumbBoundsFn (index) {
          let pageYScroll = window.pageYOffset || document.documentElement.scrollTop
          let rect = document.querySelectorAll('.preview-img')[index].getBoundingClientRect()
          return {x: rect.left, y: rect.top + pageYScroll, w: rect.width}
        }
      }, params)
      this.photoswipe = new PhotoSwipe(this.$el, UI, list, this.pswpOptions)
      this.photoswipe.init()
      // 默认下载当前index图片
      this.downloadImageUrl = list[index].src
      this.downloadImageName = `银盒宝成_${list[index].title}`

      /* 监听图片变换，替换当前下载图片链接 */
      this.photoswipe.listen('afterChange', (e) => {
        this.rotateCount = 0
        this.downloadImageUrl = list[this.photoswipe.getCurrentIndex() || 0].src
        this.downloadImageName = `银盒宝成_${list[this.photoswipe.getCurrentIndex() || 0].title}`
      })
    },
    close () {
      this.rotateCount = 0
      this.photoswipe.close()
    },
    /**
       * 旋转图片
       * @ratateDeg: 旋转角度
       * @refIndex: 当前图片容器索引
       */
    transformRotate () {
      this.rotateCount = this.rotateCount > 3 ? 0 : this.rotateCount
      this.rotateCount++
      const ratateDeg = this.rotateCount * 90
      let targetImage = this.photoswipe.currItem.container.querySelector('img')
      // image 配置crossOrigin属性，配合canvas跨域
      // targetImage.crossOrigin = "Anonymous"
      targetImage.style.transform = `rotateZ(${ratateDeg}deg)`
      // canvas旋转
      // this.rotate(targetImage, ratateDeg, function(data, w, h) {
      //   targetImage.src = data
      // })
    },
    /**
       * @description
       * 目前图片地址是阿里云服务，canvas跨域，需要运维支持
       * 公司服务器上需要做阿里云图片服务代理转发，配置cors访问
       * js替换阿里云服务地址为公司服务器地址
       * 配合image的crossOrigin属性可以解决canvas跨域
       */
    rotate (image, degrees, callback) {
      let w = image.width
      let h = image.height
      const canvasWidth = Math.max(w, h)
      let canvas = this.getCanvas(canvasWidth, canvasWidth)
      let ctx = canvas.getContext('2d')
      ctx.translate(canvasWidth / 2, canvasWidth / 2)
      ctx.rotate(degrees * (Math.PI / 180))
      let x = -canvasWidth / 2
      let y = -canvasWidth / 2
      degrees = degrees % 360
      // 没有旋转角度
      if (degrees === 0) {
        return callback(image.src, w, h)
      }
      // let sx = 0
      // let sy = 0
      if ((degrees % 180) !== 0) {
        if (degrees === -90 || degrees === 270) {
          x = -w + canvasWidth / 2
        } else {
          y = canvasWidth / 2 - h
        }
        let c = w
        w = h
        h = c
      } else {
        x = canvasWidth / 2 - w
        y = canvasWidth / 2 - h
      }
      ctx.drawImage(image, x, y)
      let canvas2 = this.getCanvas(w, h)
      let ctx2 = canvas2.getContext('2d')
      ctx2.drawImage(canvas, 0, 0, w, h, 0, 0, w, h)
      let data = canvas2.toDataURL('image/jpeg')
      callback(data, w, h)
    },
    getCanvas (width, height) {
      let canvas = document.createElement('canvas')
      canvas.width = width
      canvas.height = height
      return canvas
    }
  }
}
</script>

<style lang="scss">
  @import './css/dist/default-skin/default-skin.css';
  @import './css/dist/photoswipe.css';
  .fht__download {
    text-align: center;
    width: 44px;
    height: 44px;
    line-height: 44px !important;
    font-size: 24px;
    position: relative;
    background: none;
    cursor: pointer;
    overflow: visible;
    -webkit-appearance: none;
    display: block;
    border: 0;
    padding: 0;
    margin: 0;
    float: right;
    -webkit-transition: opacity 0.2s;
    transition: opacity 0.2s;
    -webkit-box-shadow: none;
    box-shadow: none;
    color: #fff;
    opacity: 0.85;
    &:focus,
    &:hover{
      color: #fff;
      opacity: 1;
    }
  }
</style>
