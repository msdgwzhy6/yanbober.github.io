---
layout: post
keywords: Fresco, 框架, Android图片缓存
description: facebook Fresco框架
title: "facebook Fresco框架库源使用基础"
categories: [开源框架库笔记]
tags: [Fresco, Android图片缓存]
group: archive
icon: file-alt
---
{% include site/setup %}

<hr>
##**开源项目链接**

facebook Fresco仓库：git clone https://github.com/facebook/fresco

facebook Fresco主页：http://fresco-cn.org/docs/index.html#_

Fresco Demo：https://github.com/yanbober/Android-Blog-Source/tree/master/Fresco-Android-CN-Demo

<hr>

##**背景介绍**

最近微博和论坛火了一个facebook的lib，其实没啥介绍的，以下文字摘自Fresco主页，只能说他牛逼高大上duang duang duang的。

Fresco是一个强大的图片加载组件。

Fresco中设计有一个叫做image pipeline的模块。它负责从网络，从本地文件系统，本地资源加载图片。
为了最大限度节省空间和CPU时间，它含有3级缓存设计（2级内存，1级文件）。

Fresco中设计有一个叫做Drawees模块，
方便地显示loading图，当图片不再显示在屏幕上时，及时地释放内存和空间占用。

Fresco支持Android2.3(API level 9)及其以上系统。

####**最牛逼的特点**

**内存管理**

一个没有未压缩的图片，即Android中的Bitmap，占用大量的内存。大的内存占用势必引发更加频繁的GC。
在5.0以下，GC将会显著地引发界面卡顿。

在5.0以下系统，Fresco将图片放到一个特别的内存区域。当然，在图片不显示的时候，占用的内存会自动被释放。
这会使得APP更加流畅，减少因图片内存占用而引发的OOM。

Fresco在低端机器上表现一样出色，你再也不用因图片内存占用而思前想后。

**图片的渐进式呈现**

渐进式的JPEG图片格式已经流行数年了，渐进式图片格式先呈现大致的图片轮廓，然后随着图片下载的继续，
呈现逐渐清晰的图片，这对于移动设备，尤其是慢网络有极大的利好，可带来更好的用户体验。

Android本身的图片库不支持此格式，但是Fresco支持。
使用时，和往常一样，仅仅需要提供一个图片的URI即可，剩下的事情，Fresco会处理。

**Gif图和WebP格式**

是的，支持加载Gif图，支持WebP格式。

**图像的呈现**

Fresco的Drawees设计，带来一些有用的特性：

- 自定义居中焦点(对人脸等图片显示非常有帮助)。
- 圆角图，当然圆圈也行。
- 下载失败之后，点击重现下载。
- 自定义占位图，自定义overlay, 或者进度条。
- 指定用户按压时的overlay。

**图像的加载**

Fresco的image pipeline设计，允许用户在多方面控制图片的加载：

- 为同一个图片指定不同的远程路径，或者使用已经存在本地缓存中的图片。
- 先显示一个低解析度的图片，等高清图下载完之后再显示高清图。
- 加载完成回调通知。
- 对于本地图，如有EXIF缩略图，在大图加载完成之前，可先显示缩略图。
- 缩放或者旋转图片。
- 处理已下载的图片。
- WebP支持。

<hr>

##**实战一把屌爆天的功能**

我经过实战发现使用时有几步需要留意：

1. jcenter和mavenCentral都已经有这个库了。
2. 在Application里init切记在Android管理文件里声明name，否则容易跳坑。

目前截图只是简单的测试体验了下这个屌爆天的库基本功能，日后还会持续补充：

<img src="http://yanbober.github.io/image/open/fresco1.png" />

该实例代码[点我下载](https://github.com/yanbober/Android-Blog-Source/tree/master/Fresco-Android-CN-Demo)

##**IMAGE PIPELINE指南**

[参见这里](http://fresco-cn.org/docs/intro-image-pipeline.html#_)

关于这个屌爆天的库会依据日后使用持续更新修改该文章补充。
