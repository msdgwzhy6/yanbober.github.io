---
layout: post
keywords: Android Studio, Android开发工具, AS, Android Studio使用
description: 关于Google发布的最新Android开发IDE，取代Eclipse的大势所在。
title: "Google利器Android Studio从入门到精通"
categories: [开发工具]
tags: [Android Studio]
group: archive
icon: file-alt
---
{% include site/setup %}

##AS简介

经过2年时间的研发，Google终于正式发布了面向Android开发者的集成开发环境Android Studio 1.0（稳定版）。Android Studio是Google开发的一款面向Android开发者的IDE，
支持Windows、Mac、Linux等操作系统，基于流行的Java语言集成开发环境IntelliJ搭建而成。该IDE在2013年5月的Google I/O开发者大会上首次露面，当时的测试版各种莫名其
妙的Bug，但是14年12月8日发布的版本是稳定版。Android Studio 1.0推出后，Google官方将逐步放弃对原来主要的Eclipse ADT的支持，并为Eclipse用户提供了工程迁移的解
决办法。不过相信作为Developer的你上手AS 1.0以后你再也不愿意使用原来苦逼的Eclipse+ADT了，你会被AS的各种强大所吸引。

##下载安装

下载AS前先说下，AS安装包分为含SDK版本和不含SDK版本下载，如果你有SDK，那么完全可以下载不含SDK版本；不过下载了含SDK版本也没事，安装时选择自定义SDK也可以，安装
后重新指定SDK路径也可以，总之看个人爱好喽。
先吐槽下天朝的强大吧，不得不拜服天朝的墙。如果你有梯子请去Android Developer下载最新版的AS安装包，如果你没有梯子那也有个办法，就是去[Android Studio中文社区官
网](http://www.android-studio.org/)下载你的平台需要的安装包。

下载下来以后安装的过程可以忽略了吧，能安装的都是程序猿吧，所以安装这点就不说了。

安装好了以后首次运行AS可能一直停在Fetching Android SDK component information。如下界面：

<img src="http://yanbober.github.io/image/2015-1-28-android_studio_guide_1.png" />

这是因为天朝的墙真的太高太厚把首次运行更新SDK给墙了。解决办法就是关闭安装向导，如果无法关闭可以在任务管理器中手动关掉进程（Ctrl+Alt+Del启动任务管理器），然后
打开AS安装目录下的bin目录里面的idea.properties文件，添加一条禁用开始运行向导的配置项：

	disable.android.first.run=true
	
然后再启动程序就会打开项目向导界面，这个时候如果点击Start a new Android Studio project是没有反应的，并且在Configure下面的SDK Manager是灰色的，这是因为没有安装
Android SDK的缘故。这时候一般有两种做法：

1. 自己没有SDK，需要从网络下载；打开向导的Configure-Settings，在查找框里面输入proxy，找到下面的HTTP Proxy，设置代理服务器，并且将Force https://... sources to 
be fetched using http://选中，然后退出将上面在idea.properties配置文件中添加的那条配置项注释掉重新打开Android Studio等刚开始的向导把Android SDK下载安装完成就可
以了。

2. 自己有SDK，重新指定SDK路径；打开向导的Configure->Project Defaults->Project Structure，在此填入你已有的SDK路径。

此时重启AS就可以在向导里新建Android工程喽。至此整个安装过程结束。

##基本使用介绍

首先新建工程，输入工程名和主包名和存储路径；点击next到如图步骤：

<img src="http://yanbober.github.io/image/2015-1-28-android_studio_guide_2.png" />

上图中首先你可选择你的App要适配的设备是Wear还是Mobile还是TV。在你新建App选择最低适配版本时，强大的AS会给你一些有用的统计提示，如图描述了当前版本的用户情况，点
击Help me choose后弹出如下更加形象的分布图表描述：

<img src="http://yanbober.github.io/image/2015-1-28-android_studio_guide_3.png" />

爱不释手的亮点就是这么一步一步比Eclipse强大的，这只是一些不值得一提的小点而已，强大的功能还在后面。继续点击Next选择形象友好的GUI模板，点击完成进入工程初始化过程。

第一次安装工程初始化时由于需要联网下载gradle会比较慢，不过有时候不是第一也会慢，工程依赖的gradle版本不匹配时也会自动重新下载；我的初始化很快，原因是我本地的
gradle-2.2-all.zip之前已经下载OK的。至于啥时gradle后文会有说明。这儿只是告诉你若果你看到卡一会儿时正常的。

接下来进入到了工程界面下：



	（烦请令尊体谅作者劳动成果，转载麻烦声明文章链接。您的声明与讨论是鄙人写作的动力。本篇文章依据时间及个人情况持续跟新中......）
