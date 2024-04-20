---
title: 自定义在线播放器
date: 2024-01-13 
description: 自定义在线播放器|视频与canvas
categories: web
tags: 
  - canvas
  - HTML5
---
# 自定义在线播放器|视频与canvas

![Untitled](/pic/%E8%87%AA%E5%AE%9A%E4%B9%89%E5%9C%A8%E7%BA%BF%E6%92%AD%E6%94%BE%E5%99%A8%20%E8%A7%86%E9%A2%91%E4%B8%8Ecanvas%20aad12929f85d4f919ddaf1149ec6b4ac/Untitled.png)

在使用网页去观看视频时，大家可能会注意到主流网站的**在线播放器**功能非常丰富，弹幕，热点跳转等。如果你在网页看过虎牙LOL直播，可能会体验到更强大的交互功能，比如实时查看选手经济，符文等，那么这些功能到底是如何实现的呢？

一般来说，普通的 `<video>` 标签只提供了基础的视频播放功能，例如播放、暂停、音量控制、全屏等。要实现更加丰富的功能，网站通常会借助其他技术和组件来实现，下图是拥有简单功能的播放器

![屏幕截图 2024-04-18 143506.png](/pic/%E8%87%AA%E5%AE%9A%E4%B9%89%E5%9C%A8%E7%BA%BF%E6%92%AD%E6%94%BE%E5%99%A8%20%E8%A7%86%E9%A2%91%E4%B8%8Ecanvas%20aad12929f85d4f919ddaf1149ec6b4ac/%25E5%25B1%258F%25E5%25B9%2595%25E6%2588%25AA%25E5%259B%25BE_2024-04-18_143506.png)


根据我的搜索调研呢，发现当前主流在线视频播放网站处理播放器的方案是用`<canvas>`和`<video>`标签配合使用来实现自定义的在线播放器。

video标签可以通过controls属性来控制默认的控制条是否显示，但是传统的控制条只具有基本的控制功能，同时canvas比video具有更强大的灵活性和控制性。以下是一些使用Canvas来改造视频的优势：

1. 自定义渲染：使用Canvas可以完全自定义视频的渲染过程。你可以通过使用Canvas API绘制视频帧，实现各种特效、滤镜、变换等。这样可以为视频添加独特的风格和视觉效果，提升用户体验。
2. 跨平台支持：Canvas是HTML5的一部分，几乎所有现代浏览器都支持Canvas标签。这意味着你可以在不同的设备上（包括桌面、移动设备等）使用Canvas来播放视频，而无需依赖特定的视频解码器或插件。
3. 性能优化：使用Canvas可以更好地控制视频的渲染过程，从而实现更好的性能优化。例如，你可以通过手动控制视频的渲染帧率、降低分辨率或调整渲染算法等方式，以更好地适应设备性能和网络条件。
4. 动态交互：与视频元素相比，Canvas提供了更多的交互性。你可以将其他图形、文本或用户界面元素与视频一起绘制在Canvas上，并通过JavaScript代码来实现与视频的交互效果，例如添加交互式标记、图形叠加等。
5. 兼容性：一些特定的功能，如实时弹幕、画中画等，可能在使用原生视频元素时受到限制。但是使用Canvas可以更方便地实现这些功能，以及更灵活和可定制的用户体验。

## canvas渲染video方案

把视频按帧绘制在画布上

1. 设置画布大小
2. 在画布上绘制视频帧
3. 清空画布
4. 递归调用步骤2和步骤3，以实现连续播放

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <title>video2canvas</title>
</head>
<body>
      <canvas id="myCanvas"></canvas>
      <video id="myVideo" width="600" height="360" preload controls>
        <source src="http://nettuts.s3.amazonaws.com/763_sammyJSIntro/trailer_test.mp4" type='video/mp4'>
        <source src="http://nettuts.s3.amazonaws.com/763_sammyJSIntro/trailer_test.ogg" type='video/ogg'>
      </video>
</body>

</html>
```

按照上述渲染方案完成渲染，script代码如下

```jsx
let canvas = document.getElementById("myCanvas");
let video = document.getElementById("myVideo");
let ctx = canvas.getContext("2d");

//设置事件绑定函数 
function bindEvent(ele, eventName, func) {
      if (window.addEventListener) {
        ele.addEventListener(eventName, func);
      }
      else {
        ele.attachEvent('on' + eventName, func);
      }
}

//video加载完成后设置画布大小
bindEvent(video, "loadeddata", setCanvasAndDraw);

function setCanvasAndDraw() {
      // 设置画布大小与视频大小相同
      canvas.width = video.videoWidth
      canvas.height = video.videoHeight;

      // 开始绘制画面
      drawFrame();
 }

// 在每一帧更新画面
function drawFrame() {
  // 清空画布
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // 在画布上绘制视频帧
  ctx.drawImage(video, 0, 0, canvas.width, canvas.height);

  // 递归调用，以实现连续播放
  requestAnimationFrame(drawFrame);
}
```

完成后打开页面可以看到有两个画面，通过video控制条可以同时控制两个画面的播放效果，看起来就像copy了一个video

这时我们可以把video的display设置为none，将video隐藏掉，在canvas上创建自己的视频控制条，以及各种自定义功能。因为canvas是以画布整体来使用，如果想在canvas上操作元素需要设置遮罩层，在遮罩层上添加元素。

NOTE**：**

- 进度条实现原理：video标签duration属性可以获取到视频总时长，currentTime可以获取当前播放时间
- duration是只读属性如果数据未加载完成时读取会出现NAN提示
- 设置控制条交互需要处理好重叠元素的事件触发问题，这里很容易出现BUG
- `[<video>](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/video)` MDN web doc 详细记录的video标签的属性，实际场景中出现的问题基本都可以用这些属性解决

> Web 的迷人之处在于你可以结合各种技术创造出新的形式。拥有浏览器中的原生音频和视频意味着我们可在像 `[<canvas>](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/canvas)`、[WebGL](https://developer.mozilla.org/zh-CN/docs/Web/API/WebGL_API) 或 [Web Audio API](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Audio_API) 等技术的辅助下使用这类数据流，例如：为音频添加混响和压缩效果，或为视频添加灰度/暗色滤镜。`[<video>](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/video)` MDN web doc
>