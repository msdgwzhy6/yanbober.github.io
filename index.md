---
layout: index
title: 工匠若水
keywords: 晏博, 工匠若水, android, java, C语言, PHP, 脚本, android developer, android开发, android技术分享, 极客
description: Android开发者, 嵌入式开发者, 极客
---
{% include site/setup %}

<a href="http://www3.clustrmaps.com/user/6b811baec">
<img src="http://www3.clustrmaps.com/stats/maps-no_clusters/yanbober.github.io-thumb.jpg" alt="Locations of visitors to this page" />
</a>


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

