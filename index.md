---
layout: index
title: 工匠若水
keywords: 晏博, 工匠若水, android, java, C语言, PHP, 脚本, android developer, android开发, android技术分享, 极客
description: 工匠若水的主页-开发者干货分享
---
{% include site/setup %}

## 置顶爆料头条

[安卓5.1隐秘功能曝光：谷歌VPN](http://www.ithome.com/html/android/135181.htm)

## Android开发

[主页建设中...](http://www.baidu.com)

[主页建设中...](http://www.baidu.com)

## 其他开发

[主页建设中...](http://www.baidu.com)

[主页建设中...](http://www.baidu.com)

## 大牛博客直通车

[友情链接 《第一行代码》郭林老师CSDN博客主页](http://blog.csdn.net/guolin_blog?viewmode=contents)

[友情链接 《Android系统源代码情景分析》罗升阳老师CSDN博客主页 ](http://blog.csdn.net/luoshengyang?viewmode=contents)




## 移动设备浏览我的主页请扫描如下二维码:

<img src="./image/zhuye_erweima.png" />


<div class="bshare-custom"><a title="分享到QQ空间" class="bshare-qzone"></a>
<a title="分享到新浪微博" class="bshare-sinaminiblog"></a>
<a title="分享到人人网" class="bshare-renren"></a>
<a title="分享到腾讯微博" class="bshare-qqmb"></a>
<a title="分享到有道笔记" class="bshare-youdaonote"></a>
<a title="更多平台" class="bshare-more bshare-more-icon more-style-addthis">
</a><span class="BSHARE_COUNT bshare-share-count">0</span>
</div><script type="text/javascript" charset="utf-8" src="http://static.bshare.cn/b/buttonLite.js#style=-1&amp;uuid=&amp;pophcol=2&amp;lang=zh">
</script><script type="text/javascript" charset="utf-8" src="http://static.bshare.cn/b/bshareC0.js">
</script>


<div id="post-pagination">
<ul class="pagination pagination-centered pull-right">

{% if paginator.previous_page %}
<li>
{% if paginator.previous_page == 1 %}
<a href="/"><i class="fa fa-chevron-left"></i></a>
{% else %}
<a href="/page{{ paginator.previous_page }}"><i class="fa fa-chevron-left"></i></a>
{% endif %}
</li>
{% else %}
<li class="disabled">
<span><i class="fa fa-chevron-left"></i></span>
</li>
{% endif %}

{% if paginator.page == 1 %}
<li class="active">
<a href="#">1</a>
</li>
{% else %}
<li>
<a href="/">1</a>
</li>
{% endif %}

{% for count in (2..paginator.total_pages) %}
{% if count == paginator.page %}
<li class="active">
<a href="#">{{ count }}</a>
</li>
{% else %}
<li>
<a href="/page{{ count }}">{{ count }}</a>
</li>
{% endif %}
{% endfor %}

{% if paginator.next_page %}
<li>
<a href="/page{{ paginator.next_page }}"><i class="fa fa-chevron-right"></i></a>
</li>
{% else %}
<li class="disabled">
<a href="#"><i class="fa fa-chevron-right"></i></a>
</li>
{% endif %}
</ul>
</div>

