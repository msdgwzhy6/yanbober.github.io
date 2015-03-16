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

下载下来以后安装的过程可以忽略了吧，能安装的都是程序猿吧，所以安装这点就不说了，注意已经正确安装配置了JDK。

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

<img src="http://yanbober.github.io/image/2015-1-28-android_studio_guide_4.png" />

这个创建过程可比Eclipse上长的多。主要是因为从gradle上下载。gradle也可以手动离线下载好放在对应目录下。工程的结构和Eclipse上的不同，src下分为java和res。
AS是基于idea，而idea和eclipse有大的区别，有好处也有不好的地方，在一段时间里，idea被认为是开发java最好用强大的ide工具，所以AS新建的时候有new application和new module
开发。idea没有工作空间这样的说法。这就是Eclipse用户切换过来第一个比较不适应的地方。

具体说就是：

	1. android studio是单工程的开发模式
	2. android studio中的application相当于eclipse里的workspace概念
	3. android studio中的module相当于eclipse里的project概念

有了如上三条概念自己手动创建摸索下，相信聪明的你自然就明白咋回事了吧。

接下来看一些工欲善其事必先利其器的基本高频率实用设置：

	1. 中文乱码
	
	在窗口中，找到IDE Settings->Appearance，在右侧勾选上“Override default fonts by”，然后在第一个下拉框中选择字体为“simsun”，然后apply，重启IDE，就好了。

	2. 设置快捷键
	
	在settings窗口中，找到IDE Settings->keymap，右侧打开的就是快捷键了。
	右键单击要修改的快捷键，会弹出一个菜单，选择“Add keyboard shortcut”就可以修改快捷键了。删除的话，在弹出的菜单中选择remove XXX即可。
	特别说明，在AS的快捷键设置里可以直接设置使用Eclipse快捷键还是别的IDE快捷键。如果你热衷Eclipse那么也可设置成Eclipse的快捷键。	

	3. 修改主题
	
	在IDE Settings->Appearance，右侧的Theme选择自己喜欢的主题即可。个人比较喜欢Darcula主题，也就是如上截图样式。

	4. 如何将Eclipse工程导入AS使用
	
	选择File->Import Project，在弹出的菜单中选择要导入的工程即可，选择好以后就直接next，在第二个窗口中也选择默认的第一个选项就可以。
	需要注意的是，在AS中，有两种工程，一个是Project，一个是Module，上面已经细说过了。

	5. 导入jar包
	
	选择File->Projcet Structure，在弹出的窗口中左侧找到Libraries并选中，然后点击“+”，并选择Java就能导入Jar包了。

	6. 删除项目
	
	AS对工程删除做了保护机制，默认你在项目右键发现没有删除选项。你会发现你的module上面会有一个小手机，这是保护机制。删除的第一步就是去掉保护机制，也就是
	让手机不见，具体做法就是鼠标放在工程上右键->open module setting，或者F4进入如图界面：
	
<img src="http://yanbober.github.io/image/2015-1-28-android_studio_guide_5.png" />

	选中你要删除的module，然后点击减号，这样就取消了保护机制，然后回到项目工程右键就可发现删除选项。注意：删除会将源文件删除。
	
	7. 修改工程目录
	
	在创建项目的时候，在Project Location中选好工程目录后，要自己输入一个文件夹的名字用来保存工程，然后就能使用自己的工程目录了。
	
## 入门总结

到此为止AS的基本情况相信你已经有个大致了解了。具体比Eclipse的优势体现在如下几点：

相信目前国内用Eclipse的还是大多数，那么首先就来说一下Studio的一些优点，比较才能更有说服力，才能说明为什么我们要从Eclipse迁移到Studio。

	1. AS是Google专门为Android基于IntelliJ IDEA打造的利器。亲生的永远是最好的，只是现在还在成长中而已。

	2. AS在速度上不管哪一个方面都比Eclipse快。

	3. Darcula主题UI简直就是极客范，帅爆了。

	4. 强大的智能提示补全功能在写代码时简直比Eclipse高效率N倍。
	
	5. 智能保存，不需要Ctrl + S。效率会大大提升。

	6. 整合Gradle构建工具，Gradle集合了Ant和Maven的优点，不管是配置、编译、打包都非常牛逼。

	7. UI编辑器简直比Eclipse高效N倍，自带了多设备的实时预览，简直是神器。多语言适配点击地球直接输入，再也不用比较那个string没有翻译了。

	8. 内置终端直接替代cmd命令行，一个IDE全部搞定。

	9. 完善的插件系统，如Git、Markdown、Gradle等，直接搜索下载。

	10. 版本控制系统，安装的时候就自带GitHub, Git, SVN等流行的版本控制系统，可以直接check out你的项目，边写代码边右键可以直接具备BCompare功能与其他版本进行对比修改。

总之就一句话，相信我，若果你和我一样是Eclipse用户切换过来，那么你绝对不会再切换回去，你会爱上AS的。

##Android Studio目录结构及相关操作


##Android Studio开发调试使用

##总结



	（烦请令尊体谅作者劳动成果，转载麻烦声明文章链接。您的声明与讨论是鄙人写作的动力。本篇文章依据时间及个人情况持续跟新中......）
