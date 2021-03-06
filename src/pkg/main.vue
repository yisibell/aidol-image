<template>
  <div class="el-image">
    <slot v-if="loading" name="placeholder">
      <div class="el-image__placeholder"></div>
    </slot>
    <slot v-else-if="error" name="error">
      <div class="el-image__error"> 加载失败 </div>
    </slot>
    <img
      v-else
      class="el-image__inner"
      v-bind="$attrs"
      v-on="$listeners"
      @click="clickHandler"
      :src="src"
      :style="imageStyle"
      :class="{ 'el-image__inner--center': alignCenter, 'el-image__preview': preview }">
    <template v-if="preview">
      <image-viewer ref="el-image-viewer" :z-index="zIndex" :initial-index="imageIndex" v-if="showViewer" :on-close="closeViewer" :on-switch="switchHandler" :url-list="previewSrcList"/>
    </template>
  </div>
</template>

<script>
  import ImageViewer from 'element-ui/packages/image/src/image-viewer';
  import { on, off, getScrollContainer, isInContainer, setStyle } from 'element-ui/src/utils/dom';
  import { isString, isHtmlElement } from 'element-ui/src/utils/types';
  import throttle from 'throttle-debounce/throttle';
  const isSupportObjectFit = () => document.documentElement.style.objectFit !== undefined;
  const ObjectFit = {
    NONE: 'none',
    CONTAIN: 'contain',
    COVER: 'cover',
    FILL: 'fill',
    SCALE_DOWN: 'scale-down'
  };
  let prevOverflow = '';
  export default {
    name: 'AiImage',
    inheritAttrs: false,
    components: {
      ImageViewer
    },
    props: {
      src: String,
      fit: String,
      lazy: Boolean,
      scrollContainer: {},
      previewSrcList: {
        type: Array,
        default: () => []
      },
      zIndex: {
        type: Number,
        default: 2000
      },
      // current image viewer tips
      viewerTips: {
        type: [Function, String, undefined],
        default: undefined
      },
      // viewer tips div class name
      viewerTipsClassName: {
        type: String,
        default: 'ai-image-viewer__tips'
      },
      // viewer tips div style object
      viewerTipsStyle: {
        type: Object,
        default: () => ({
          height: '40px',
          width: '100%',
          display: 'flex',
          justifyContent: 'center',
          alignItems: 'center',
          fontSize: '14px',
          color: '#fff',
          position: 'absolute',
          left: '0px',
          top: '0px'
        })
      }
    },
    data() {
      return {
        loading: true,
        error: false,
        show: !this.lazy,
        imageWidth: 0,
        imageHeight: 0,
        showViewer: false
      };
    },
    computed: {
      imageStyle() {
        const { fit } = this;
        if (!this.$isServer && fit) {
          return isSupportObjectFit()
            ? { 'object-fit': fit }
            : this.getImageStyle(fit);
        }
        return {};
      },
      alignCenter() {
        return !this.$isServer && !isSupportObjectFit() && this.fit !== ObjectFit.FILL;
      },
      preview() {
        const { previewSrcList } = this;
        return Array.isArray(previewSrcList) && previewSrcList.length > 0;
      },
      imageIndex() {
        let previewIndex = 0;
        const srcIndex = this.previewSrcList.indexOf(this.src);
        if (srcIndex >= 0) {
          previewIndex = srcIndex;
        }
        return previewIndex;
      }
    },
    watch: {
      src() {
        this.show && this.loadImage();
      },
      show(val) {
        val && this.loadImage();
      }
    },
    mounted() {
      if (this.lazy) {
        this.addLazyLoadListener();
      } else {
        this.loadImage();
      }
    },
    beforeDestroy() {
      this.lazy && this.removeLazyLoadListener();
    },
    methods: {
      loadImage() {
        if (this.$isServer) return;
        // reset status
        this.loading = true;
        this.error = false;
        const img = new Image();
        img.onload = e => this.handleLoad(e, img);
        img.onerror = this.handleError.bind(this);
        // bind html attrs
        // so it can behave consistently
        Object.keys(this.$attrs)
          .forEach((key) => {
            const value = this.$attrs[key];
            img.setAttribute(key, value);
          });
        img.src = this.src;
      },
      handleLoad(e, img) {
        this.imageWidth = img.width;
        this.imageHeight = img.height;
        this.loading = false;
        this.error = false;
      },
      handleError(e) {
        this.loading = false;
        this.error = true;
        this.$emit('error', e);
      },
      handleLazyLoad() {
        if (isInContainer(this.$el, this._scrollContainer)) {
          this.show = true;
          this.removeLazyLoadListener();
        }
      },
      addLazyLoadListener() {
        if (this.$isServer) return;
        const { scrollContainer } = this;
        let _scrollContainer = null;
        if (isHtmlElement(scrollContainer)) {
          _scrollContainer = scrollContainer;
        } else if (isString(scrollContainer)) {
          _scrollContainer = document.querySelector(scrollContainer);
        } else {
          _scrollContainer = getScrollContainer(this.$el);
        }
        if (_scrollContainer) {
          this._scrollContainer = _scrollContainer;
          this._lazyLoadHandler = throttle(200, this.handleLazyLoad);
          on(_scrollContainer, 'scroll', this._lazyLoadHandler);
          this.handleLazyLoad();
        }
      },
      removeLazyLoadListener() {
        const { _scrollContainer, _lazyLoadHandler } = this;
        if (this.$isServer || !_scrollContainer || !_lazyLoadHandler) return;
        off(_scrollContainer, 'scroll', _lazyLoadHandler);
        this._scrollContainer = null;
        this._lazyLoadHandler = null;
      },
      /**
       * simulate object-fit behavior to compatible with IE11 and other browsers which not support object-fit
       */
      getImageStyle(fit) {
        const { imageWidth, imageHeight } = this;
        const {
          clientWidth: containerWidth,
          clientHeight: containerHeight
        } = this.$el;
        if (!imageWidth || !imageHeight || !containerWidth || !containerHeight) return {};
        const vertical = imageWidth / imageHeight < 1;
        if (fit === ObjectFit.SCALE_DOWN) {
          const isSmaller = imageWidth < containerWidth && imageHeight < containerHeight;
          fit = isSmaller ? ObjectFit.NONE : ObjectFit.CONTAIN;
        }
        switch (fit) {
          case ObjectFit.NONE:
            return { width: 'auto', height: 'auto' };
          case ObjectFit.CONTAIN:
            return vertical ? { width: 'auto' } : { height: 'auto' };
          case ObjectFit.COVER:
            return vertical ? { height: 'auto' } : { width: 'auto' };
          default:
            return {};
        }
      },
      clickHandler() {
        // don't show viewer when preview is false
        if (!this.preview) {
          return;
        }
        // prevent body scroll
        prevOverflow = document.body.style.overflow;
        document.body.style.overflow = 'hidden';
        this.showViewer = true;
        this.switchHandler(0)
      },
      closeViewer() {
        document.body.style.overflow = prevOverflow;
        this.showViewer = false;
      },
      // emit a event when switch the image.
      switchHandler(index) {
        this.$nextTick(() => {
          const el_image_viewer = this.$refs['el-image-viewer']
          this.$emit('switch', index, el_image_viewer)
          this.createViewerTips(index, el_image_viewer)
        })
      },
      // create image viewer tips
      createViewerTips(index, viewer) {
        const tips = this.viewerTips 
        let str = ''

        if(isString(tips) && tips) {
          str = tips
        }else if (tips) {
          str = tips(index, viewer)
        }

        if(str) {
          const el = viewer.$el
          const div = document.createElement('div')
          const className = this.viewerTipsClassName
          const id = `${this.viewerTipsClassName}-${this._uid}`
          const isExitDiv = document.querySelector(`#${id}`)

          div.innerHTML = str
          div.setAttribute('id', id)
          div.classList.add(className)
          setStyle(div, this.viewerTipsStyle)
          
          if (isExitDiv) {
            el.removeChild(isExitDiv)
          }
      
          el.appendChild(div)

        }
      }
    }
  };
</script>