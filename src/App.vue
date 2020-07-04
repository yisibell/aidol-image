<template>
  <div id="app">
    <ai-image :src="url" :preview-src-list="srcList" viewer-tips="hello aidol image" @switch="handleSwitch" />
  </div>
</template>

<script>
import AiImage from './pkg/main'

export default {
  name: 'App',
  components: {
    AiImage
  },
  data() {
    return {
      url: 'https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg',
      srcList: [
        'https://fuss10.elemecdn.com/8/27/f01c15bb73e1ef3793e64e6b7bbccjpeg.jpeg',
        'https://fuss10.elemecdn.com/1/8e/aeffeb4de74e2fde4bd74fc7b4486jpeg.jpeg'
      ]
    }
  },
  methods: {
    handleSwitch(i, el) {
      const dom = el.$el
      console.log(i, el, dom);
    }
  }
}
</script>
