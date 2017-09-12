---
title: Android Oreo 新特性适配 —— Adaptive Icons
date: 2017-09-12 21:13:09
tags: Android
---
美国日全食那天，一段很短的视频带来了 Android Oreo 的正式版本，怀着激动的心情拿出了我日日把玩的 Nexus 5X，升级到了正式版本。
流畅。
我只能这么说，虽然不至于感受到宣传中提到的提升，但流畅的操作让我非常舒服。
可是一切感觉在看到图标后变了。
![图标](https://user-images.githubusercontent.com/16117136/30329107-ec88fb50-9803-11e7-8964-1e51c6f62ea6.png)
这方方的图标看的我也方方的。7.X时代圆润的圆形图标呢！
灰溜溜刷回了Lineage OS，并在知乎上喷了一通。<a href="https://www.zhihu.com/question/64259646/answer/218605483">我的回答</a>。
可是在 Pixel 上确是华丽丽的圆形图标，好看整齐得让人心醉。
那么同样的 icon 为什么在不同的手机上会显示不同的样子呢。
那就是 Android 8.0 的新特性 Adaptive Icons。
前不久刚刚为公司的产品匹配了这一特性，踩了点坑但基本上很顺畅，写下来作为笔记。
首先第一步，绝对是先使用 Android Studio 3.0，现在的版本是 Android Studio 3.0 Beta 4。
然后就是要将项目的 targetSdkVersion 升级至 26 的版本，当然随之要变动的就是 compileSdkVersion 的版本，系统不会允许 compileSdkVersion 版本低于 targetSdkVersion。然后就是相应 support 库的升级，跳过不谈，这些准备工作做好后开始新图标的使用。
![步骤1](https://user-images.githubusercontent.com/16117136/30329538-1f430dd2-9805-11e7-8a7b-88ef5d1a6bd7.png)
点击后出现了如下的页面。
![步骤2](https://user-images.githubusercontent.com/16117136/30330043-8b79df66-9806-11e7-9859-0d9caa6d5154.png)
这里我们就可以看到官方提供的 Android 小人的 icon 示意图，各种形态，那么 Android 是怎么做到的呢？
我们可以看到它是由两个图层组成的，一个叫做 Foreground 前景，一个叫做 background 背景。
前景可以是一张 .png 图片，大小为 108 * 108，具体可以参见 Google 官方文档。
背景是一张矢量图 drawable 或者 纯色背景，这样 Android 在启动器的要求下可以选取适当的外框。
我们使用 Android Studio 导入图标后可以看到一个叫做 mipmap-anydpi-v26 的目录，里面是一个 xml 文件，代码如下：
```XML
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@color/ic_launcher_background"/>
    <foreground android:drawable="@mipmap/ic_launcher_foreground"/>
</adaptive-icon>
```
点进去我们再看两张前后景图片的代码：
背景
```XML
<resources>
  <color name="ic_launcher_background">#FFFFFF</color>
</resources>
```
前景自然就是我们叫做 ic_launcher_background 的图片了。
当然这个时候的项目里还有 ic_launcher，这个是在不能使用 Adaptive Icons 情况下使用的图标。最后我们需要的是在项目的清单文件中注明
```
android:icon="@mipmap/ic_launcher"
```
让然有必要还有 round Icon 属性。
这里不得不说 Google 的套路，我理解 Google 想要统一 Android 图标的想法，但是在 Android 7 时推出的 round Icon 仅仅一代版本就遭到了弃用，现在形成了 Android 7 以下使用原来的图标，形态各异，Android 7 使用只有Android 7可以使用的 round Icon，统一的圆形图标，当然不排除很多无良厂家拿着一个奇奇怪怪的图标也称呼其为 round Icon，而 Android 8开始使用自适应图标。看得出 Google 在不断的探索 Android 的未来方向，就是系统层级相对集中的管理，包括图标、电池、权限等元素，而不是任由各种产品随意设计的以前的状况，这对于 Android 是好事，对于开发者也是好事，对于热爱 Android 的也是好事。
