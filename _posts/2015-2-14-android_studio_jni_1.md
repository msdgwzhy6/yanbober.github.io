---
layout: post
keywords: jni, Android Studio, ndk
description: Android Studio NDK, Android Studio JNI, JNI, NDK
title: "NDK-JNI实战教程（一） 在Android Studio运行第一个NDK程序"
categories: [NDK-JNI开发]
tags: [NDK, JNI]
group: archive
icon: file-alt
---
{% include site/setup %}

NDK开发，其实是为了项目需要调用底层的一些C/C++的一些东西；另外就是为了效率更加高些。如果你在Eclipse+ADT下开发过NDK就能体会到要么是配置NDK还要下载Cygwin，
配置Cygwin ,然后需要编译生成，相当的蛋疼。要么是直接用Eclipse开发，但是前期配置也是一堆；真心蛋疼。但是现在在AS上Eclipse能做到的这边都OK，这边有的Eclipse
上没有的，而且Google亲生的支持下只会越来越比Eclipse下开发NDK更加牛逼，所以你还不准备上手吗？

在AS开发NDK JNI也需要配置，不过相当Easy。第一步就是去官方下载个NDK包就可以了，像我的直接放在D盘就行了。关于怎么下载安装看这里 [AD NDK](http://developer.android.com/tools/sdk/ndk/index.html#Revisions)会有介绍。

第二步就是就是直接写代码了。哈哈，你没听错，是这样的，
方便吧？至于下载下来的NDK怎么和AS工程关联，也就是一行配置的问题，后文有说明带你一步一步体验。

##Let's Go！！！

在AS中新建一个Project，然后再新建一个class为NdkJniUtils，在内部声明native方法（jni使用的定义，后面系列教程会细说）。

{% highlight ruby %}

package io.github.yanbober.ndkapplication;

public class NdkJniUtils {
    public native String getCLanguageString();
}

{% endhighlight %}

在工程主文件Activity中写入如下代码调运JNI的东西显示在UI上。

{% highlight ruby %}

public class MainActivity extends ActionBarActivity {
    private TextView mTextView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mTextView = (TextView) this.findViewById(R.id.test);

        NdkJniUtils jni = new NdkJniUtils();

        mTextView.setText(jni.getCLanguageString());
    }
}

{% endhighlight %}

然后build project得到其中中间文件，我们关注的是.class文件。编译OK以后生成的class文件在AS工程的如下目录：
NDKApplication\app\build\intermediates\classes\debug。然后接下来的步骤就是根据生成的class文件，利用javah
生成对应的 .h头文件。

点开AS的Terminal标签，默认进入到该项目的app文件夹下。我在windows平台下输入如下命令跳转到class中间文件生成路径：
	
	xxxxx\app> cd build\intermediates\classes\debug
	
然后执行如下javah命令生成h文件。

	xxxxx\debug> javah -jni io.github.yanbober.ndkapplication.NdkJniUtils
	
执行完之后你可以在文件夹NDKApplication\app\build\intermediates\classes\debug下看见生成的 .h头文件为：

	io_github_yanbober_ndkapplication_NdkJniUtils.h
	
其内容为：

{% highlight ruby %}

/* DO NOT EDIT THIS FILE - it is machine generated */
#include <jni.h>
/* Header for class io_github_yanbober_ndkapplication_NdkJniUtils */

#ifndef _Included_io_github_yanbober_ndkapplication_NdkJniUtils
#define _Included_io_github_yanbober_ndkapplication_NdkJniUtils
#ifdef __cplusplus
extern "C" {
#endif
/*
 * Class:     io_github_yanbober_ndkapplication_NdkJniUtils
 * Method:    getCLanguageString
 * Signature: ()Ljava/lang/String;
 */
JNIEXPORT jstring JNICALL Java_io_github_yanbober_ndkapplication_NdkJniUtils_getCLanguageString
  (JNIEnv *, jobject);

#ifdef __cplusplus
}
#endif
#endif

{% endhighlight %}

在工程的main目录下新建一个名字为jni的目录，然后将刚才的 .h文件剪切过来。
在jni目录下新建一个c文件，随意取名，我的叫jnitest.c 。然后编辑代码如下（后面会解释啥意思，这里重在工具使用）：

{% highlight ruby %}

#include "io_github_yanbober_ndkapplication_NdkJniUtils.h"
/*
 * Class:     io_github_yanbober_ndkapplication_NdkJniUtils
 * Method:    getCLanguageString
 * Signature: ()Ljava/lang/String;
 */
JNIEXPORT jstring JNICALL Java_io_github_yanbober_ndkapplication_NdkJniUtils_getCLanguageString
  (JNIEnv *env, jobject obj){
     return (*env)->NewStringUTF(env,"This just a test for Android Studio NDK JNI developer!");
  }

{% endhighlight %}

接下来在工程的local.properties文件中添加NDK路径（上面下载好的那个NDK），类似其中的SDK路径一样，我的添加后如下：

{% highlight ruby %}

sdk.dir=D\:\\AndroidStdioSDK\\sdk
#add by 工匠若水
ndk.dir=D\:\\AndroidStdioSDK\\android-ndk-r10d-64bit

{% endhighlight %}

接下来在app module目录下的build.gradle中设置库文件名（生成的so文件名）。找到gradle文件的defaultConfig这项，在里面添加如下内容：

{% highlight ruby %}
defaultConfig {
	......
	ndk{
		moduleName "YanboberJniLibName"			//生成的so名字
		abiFilters "armeabi", "armeabi-v7a", "x86"	//输出指定三种abi体系结构下的so库。目前可有可无。
    }
}
{% endhighlight %}

现在生成的so库名字也有了，那就去代码的NdkJniUtils java文件添加静态初始化load代码，添加如下：

{% highlight ruby %}
static {
        System.loadLibrary("YanboberJniLibName");	//defaultConfig.ndk.moduleName
    }
{% endhighlight %}

好了，到此AS下NDK JNI开发的代码编写和设置就OK了，接下来就是编译工程运行就可以了。

但是有些电脑好奇怪此时编译会报错，妹的，没辙，后来网上找到答案说这是NDK在Windows下一个bug，当只编译一个单一文件时出现，解决办法就是
再添加一个空的文件就行了，这个网站有介绍：[NDK在Windows的一个bug](http://ph0b.com/android-studio-gradle-and-ndk-integration/)。不过
你要是刚才能顺利编译就没必要蛋疼这个问题了。

好了，我的编译运行结果如下：

<img src="http://yanbober.github.io/image/2015-2-14-android_studio_jni_1/1.png" />

到此为止简单的体验AS下NDK开发的过程就结束了。期待下一篇再续深入。

	（烦请令尊体谅作者劳动成果，转载麻烦声明文章链接。您的声明与讨论是鄙人写作的动力。JNI NDK系列文章依据时间及个人情况持续更新中......）
