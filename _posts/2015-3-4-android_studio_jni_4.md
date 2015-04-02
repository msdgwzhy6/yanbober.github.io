---
layout: post
keywords: jni, Android Studio, ndk
description: Android Studio NDK, Android Studio JNI, JNI, NDK
title: "NDK-JNI实战教程（四） JNI实用总结"
categories: [NDK-JNI开发]
tags: [NDK, JNI]
group: archive
icon: file-alt
---
{% include site/setup %}

<hr>

##**官方英文原版**

[Oracle JNI官方文档](http://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/jniTOC.html)

<hr>

#**关于字符串处理**

{% highlight ruby %}
GetStringUTFChars
ReleaseStringUTFChars
NewStringUTF
GetStringUTFLength

GetStringChars
ReleaseStringChars
GetStringLength

GetStringCritical
ReleaseStringCritical

GetStringRegion
GetStringUTFRegion

{% endhighlight %}

- 对于小字符串，GetStringRegion和GetStringUTFRegion这两对函数是最佳选择，因为缓冲区可以被编译器提前分配，而且永远不会产生内存溢出的异常。
当你需要处理一个字符串的一部分时，使用这对函数也是不错。因为它们提供了一个开始索引和子字符串的长度值。另外，复制少量字符串的消耗 也是非常小的。
- 使用GetStringCritical和ReleaseStringCritical这对函数时，必须非常小心。一定要确保在持有一个由 GetStringCritical 获取到的指针时，
本地代码不会在 JVM 内部分配新对象，或者做任何其它可能导致系统死锁的阻塞性调用。
- 获取Unicode字符串和长度，使用GetStringChars和GetStringLength函数
- 获取UTF-8字符串的长度，使用GetStringUTFLength函数
- 创建Unicode字符串，使用NewStringUTF函数
- 从Java字符串转换成C/C++字符串，使用GetStringUTFChars函数
- 通过GetStringUTFChars、GetStringChars、GetStringCritical获取字符串，这些函数内部会分配内存，必须调用相对应的ReleaseXXXX函数释放内存


{% highlight ruby %}

{% endhighlight %}



	（烦请令尊体谅作者劳动成果，转载麻烦声明文章链接。您的声明与讨论是鄙人写作的动力。JNI NDK系列文章依据时间及个人情况持续更新中......）
