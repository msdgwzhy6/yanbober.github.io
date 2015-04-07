---
layout: post
keywords: Volley, 框架, Android HTTP
description: Google Android Volley网络库
title: "Google Volley框架源码走读"
categories: [开源框架库笔记]
tags: [Volley, Android网络请求]
group: archive
icon: file-alt
---
{% include site/setup %}

<hr>
##**开源项目链接**

[Volley自定义 Android Developer文档](http://developer.android.com/training/volley/request-custom.html)

Volley主页：https://android.googlesource.com/platform/frameworks/volley

Volley仓库：git clone https://android.googlesource.com/platform/frameworks/volley

Volley GitHub Demo：在GitHub主页搜索Volley会有很多，不过建议阅读Android Developer文档。

<hr>

##**背景知识**

在Volley使用基础那一篇最后一个知识点说到了Volley的请求架构，这里再搬过来说说。

在Android Developer上看到的这幅图：

<img src="http://yanbober.github.io/image/open/volley_1.png" />

RequestQueue会维护一个缓存调度线程（cache线程）和一个网络调度线程池（net线程），
当一个Request被加到队列中的时候，cache线程会把这个请求进行筛选：如果这个请求的内容可以在缓存中找到，
cache线程会亲自解析相应内容，并分发到主线程（UI）。
如果缓存中没有，这个request就会被加入到另一个NetworkQueue，所有真正准备进行网络通信的request都在这里，
第一个可用的net线程会从NetworkQueue中拿出一个request扔向服务器。
当响应数据到的时候，这个net线程会解析原始响应数据，写入缓存，并把解析后的结果返回给主线程。

直接这么看好空洞，所以直接把clone的工程导入IDE边看代码边看这个图吧。

{% highlight ruby %}

{% endhighlight %}



{% highlight ruby %}

{% endhighlight %}



{% highlight ruby %}

{% endhighlight %}



{% highlight ruby %}

{% endhighlight %}



{% highlight ruby %}

{% endhighlight %}