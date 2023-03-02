> WARNING: 该组件已不再维护，且不推荐这样使用，请改用 <a href="https://github.com/yisibell/vue-previewable-image">vue-previewable-image</a> 替代。




# aidol-image

A image Component for vue.

> `aidol-image` 是对 `element-ui@2.x` 中 `el-image` 功能增强的 **Hack**。将其内部原本没有暴露出来的 `switch` 事件丢了出来。从而让你有能力控制 `viewer` 切换时的某些操作。比如：给当前切换的大图预览加上个标题显示。

> 注意：请不要过度依赖于该组件。最好的方式是 `el-image` 能够原生提供该 `api`。或许可以给她提交个 **PR** 来解决该问题。

# Installation

``` bash
$ npm i @aidol/image --save
```

# Usage

``` html
<!-- demo.vue -->

<template>
<div>
  <ai-image :src="url" :preview-src-list="srcList" viewer-tips="hello aidol image" @switch="handleSwitch" />
</div>
</template>
```

``` js
import AiImage from '@aidol/image'
export default {
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
    handleSwitch(index, viewer) {
      console.log(index) // 当前预览图索引
      console.log(viewer) // 大图预览组件实例
    }
  }
}
```

# Attributes

| 参数 | 说明 | 类型 | 可选值 | 默认值 |
| :------------------------------- | :------------------------------- | :------------------------------- | :------------------------------- | :------------------------------- |
| src | 图片源，同原生 | string | - | - |
| fit | 确定图片如何适应容器框，同原生 `object-fit` | string | fill / contain / cover / none / scale-down | - |
| alt | 原生 alt | string | - | - |
| referrer-policy | 原生 referrerPolicy | string | - | - |
| lazy | 是否开启懒加载 | boolean | - | false |
| scroll-container | 开启懒加载后，监听 scroll 事件的容器 | string / HTMLElement | - | 最近一个 `overflow` 值为 `auto` 或 `scroll` 的父元素 |
| z-index | 设置图片预览的 z-index | Number | - | 2000 |
| viewer-tips | 为当前大图预览提供的提示文本内容，如果提供了，则会显示在预览弹出层的上方，默认文本居中。如果是 `string` 类型，则每张预览大图显示同样的提示文本，如果是 `function` 类型，则该函数会被依次传入 `index` 当前预览大图索引值，`viewer` 大图预览组件实例。 | string / function | - | undefined |
| viewer-tips-class-name | 为 `viewer-tips` 元素添加类名 | string | - | ai-image-viewer__tips |
| viewer-tips-style | 为 `viewer-tips` 元素添加样式 | object | - | 见下方注释 |

**viewer-tips-style默认值：**

``` js
{
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
}
```

# Events

| 事件名称 | 说明 | 回调参数 |
| :----------- | :----------- | :----------- |
| load | 图片加载成功触发	 | (e: Event) |
| error | 图片加载失败触发	 | (e: Event) |
| switch | 预览图切换时触发，首次点开预览图时也会触发。`index` 当前预览图索引值，`viewer` 大图预览弹出层组件实例。 | (index: number, viewer: VNode) |


# Slots

| 名称 | 说明 |
| :------- | :------- |
| placeholder | 图片未加载的占位内容 |
| error | 加载失败的内容 |

# Logs
