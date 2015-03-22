---
layout: post
keywords: jni, Android Studio, ndk
description: Android Studio NDK, Android Studio JNI, JNI, NDK
title: "NDK-JNI实战教程（二） JNI官方中文资料"
categories: [NDK-JNI开发]
tags: [NDK, JNI]
group: archive
icon: file-alt
---
{% include site/setup %}

<hr>

##声明

该篇文章完全引用自《JNI完全手册》完整版，用来方便查询查阅，同时作为该系列教程的基础知识。感谢原文档作者。

<hr>

##正文  >>>

<hr>

##设计概述

###JNI接口函数和指针

平台相关代码是通过调用JNI函数来访问Java虚拟机功能的。JNI函数可通过接口指针来获得。接口指针是指针的指针，它指向
一个指针数组，而指针数组中的每个元素又指向一个接口函数。每个接口函数都处在数组的某个预定偏移量中。下图说明了接
口指针的组织结构。

<img src="http://yanbober.github.io/image/2015-2-14-android_studio_jni_1/2.png" />

JNI接口的组织类似于C++虚拟函数表或COM接口。使用接口表而不使用硬性编入的函数表的好处是使JNI名字空间与平台相关代码
分开。虚拟机可以很容易地提供多个版本的JNI函数表。例如，虚拟机可支持以下两个JNI函数表：

	一个表对非法参数进行全面检查，适用于调试程序。
	另一个表只进行JNI规范所要求的最小程度的检查，因此效率较高。
	
JNI接口指针只在当前线程中有效。因此，本地方法不能将接口指针从一个线程传递到另一个线程中。实现JNI的虚拟机可将本地
线程的数据分配和储存在JNI接口指针所指向的区域中。本地方法将JNI接口指针当作参数来接受。虚拟机在从相同的Java线程中
对本地方法进行多次调用时，保证传递给该本地方法的接口指针是相同的。但是，一个本地方法可被不同的Java线程所调用，因
此可以接受不同的JNI接口指针。

###加载和链接本地方法

对本地方法的加载通过System.loadLibrary方法实现。下例中，类初始化方法加载了一个与平台有关的本地库，在该本地库中
给出了本地方法f的定义：
{% highlight ruby %}
package pkg;

class Cls {
	native double f(int i, String s);
	
	static {
		System.loadLibrary("pkg_Cls");
	}
}
{% endhighlight %}

System.loadLibrary的参数是程序员任意选取的库名。系统按照标准的但与平台有关的处理方法将该库名转换为本地库名。例如，
Solaris系统将名称pkg_Cls转换为libpkg_Cls.so，而Win32系统将相同的名称pkg_Cls转换为pkg_Cls.dll。程序员可用单个库来
存放任意数量的类所需的所有本地方法，只要这些类将被相同的类加载器所加载。虚拟机在其内部为每个类加载器保护其所加载
的本地库清单。提供者应该尽量选择能够避免名称冲突的本地库名。如果底层操作系统不支持动态链接，则必须事先将所有的本
地方法链接到虚拟机上。这种情况下，虚拟机实际上不需要加载库即可完成System.loadLibrary调用。程序员还可调用JNI函数
RegisterNatives()来注册与类关联的本地方法。在与静态链接的函数一起使用时，RegisterNatives()函数将特别有用。

###解析本地方法名

动态链接程序是根据项的名称来解析各项的。本地方法名由以下几部分串接而成：

1. 前缀 Java_
2. mangled 全限定的类名
3. 下划线（“_”）分隔符
4. mangled 方法名
5. 对于重载的本地方法，加上两个下划线（“__”），后跟mangled参数签名

虚拟机将为本地库中的方法查找匹配的方法名。它首先查找短名（没有参数签名的名称），然后再查找带参数签名的长名称。
只有当某个本地方法被另一个本地方法重载时程序员才有必要使用长名。但如果本地方法的名称与非本地方法的名称相同，则
不会有问题。因为非本地方法（Java方法）并不放在本地库中。下例中，不必用长名来链接本地方法g，因为另一个方法g不是
本地方法，因而它并不在本地库中。

{% highlight ruby %}
class Cls1 {
	int g(int i);
	native int g(double d);
}
{% endhighlight %}

我们采取简单的名字搅乱方案，以保证所有的Unicode字符都能被转换为有效的C函数名。我们用下划线（“_”）字符来代替全限定的
类名中的斜杠（“/”）。由于名称或类型描述符从来不会以数字打头，我们用 _0、...、_9 来代替转义字符序列。

本地方法和接口API都要遵守给定平台上的库调用标准约定。例如，UNIX系统使用C调用约定，而Win32系统使用 __stdcall。

###本地方法的参数

JNI接口指针是本地方法的第一个参数。其类型是JNIEnv。第二个参数随本地方法是静态还是非静态而有所不同。非静态本地方法的
第二个参数是对对象的引用，而静态本地方法的第二个参数是对其Java类的引用。其余的参数对应于通常Java方法的参数。本地方法
调用利用返回值将结果传回调用程序中。下一章 “JNI的类型和数据结构” 将描述Java类型和C类型之间的映射。

代码示例说明了如何用C函数来实现本地方法f。对本地方法f的声明如下：

{% highlight ruby %}
package pkg;

class Cls {
	native double f(int i, String s);
	...
}
{% endhighlight %}

具有长mangled名称Java_pkg_Cls_f_ILjava_lang_String_2的C函数实现本地方法f：

{% highlight ruby %}
jdouble Java_pkg_Cls_f__ILjava_lang_String_2 (
	JNIEnv *env,
	/* 接口指针 */
	jobject obj,
	/* “this”指针 */
	jint i,
	/* 第一个参数 */
	jstring s)
	/* 第二个参数 */
	{
	/* 取得Java字符串的C版本 */
	const char *str = (*env)->GetStringUTFChars(env, s, 0);
	/* 处理该字符串 */
	...
	/* 至此完成对 str 的处理 */
	(*env)->ReleaseStringUTFChars(env, s, str);
	return ...
}
{% endhighlight %}

注意，我们总是用接口指针env来操作Java对象。可用C++将此代码写得稍微简洁一些，如代码示例所示：

{% highlight ruby %}
extern "C" /* 指定 C 调用约定 */
	jdouble Java_pkg_Cls_f__ILjava_lang_String_2 (
		JNIEnv *env,
		/* 接口指针 */
		jobject obj,
		/* “this”指针 */
		jint i,
		/* 第一个参数 */
		jstring s)
		/* 第二个参数
		*/
		{
		const char *str = env->GetStringUTFChars(s, 0);
		...
		env->ReleaseStringUTFChars(s, str);
		return ...
	}
{% endhighlight %}

使用C++后，源代码变得更为直接，且接口指针参数消失。但是，C++的内在机制与C的完全一样。在C++中，JNI函数被定义为内联
成员函数，它们将扩展为相应的C对应函数。

###引用Java对象

基本类型（如整型、字符型等）在Java和平台相关代码之间直接进行复制。而Java对象由引用来传递。虚拟机必须跟踪传到平台相关
代码中的对象，以使这些对象不会被垃圾收集器释放。反之，平台相关代码必须能用某种方式通知虚拟机它不再需要那些对象，
同时，垃圾收集器必须能够移走被平台相关代码引用过的对象。

####全局和局部引用

JNI将平台相关代码使用的对象引用分成两类：局部引用和全局引用。局部引用在本地方法调用期间有效，并在本地方法返回后被
自动释放掉。全局引用将一直有效，直到被显式释放。对象是被作为局部引用传递给本地方法的，由JNI函数返回的所有Java对象
也都是局部引用。JNI允许程序员从局部引用创建全局引用。要求Java对象的JNI函数既可接受全局引用也可接受局部引用。本地方法
将局部引用或全局引用作为结果返回。

大多数情况下，程序员应该依靠虚拟机在本地方法返回后释放所有局部引用。但是，有时程序员必须显式释放某个局部引用。
例如，考虑以下的情形：

1. 本地方法要访问一个大型Java对象，于是创建了对该Java对象的局部引用。然后，本地方法要在返回调用程序之前执行其它计算。
对这个大型Java对象的局部引用将防止该对象被当作垃圾收集，即使在剩余的运算中并不再需要该对象。

2. 本地方法创建了大量的局部引用，但这些局部引用并不是要同时使用。由于虚拟机需要一定的空间来跟踪每个局部引用，
创建太多的局部引用将可能使系统耗尽内存。 例如，本地方法要在一个大型对象数组中循环，把取回的元素作为局部引用，
并在每次迭代时对一个元素进行操作。每次迭代后，程序员不再需要对该数组元素的局部引用。

JNI允许程序员在本地方法内的任何地方对局部引用进行手工删除。为确保程序员可以手工释放局部引用，JNI函数将不能创建额外的
局部引用，除非是这些JNI函数要作为结果返回的引用。局部引用仅在创建它们的线程中有效。本地方法不能将局部引用从一个线程
传递到另一个线程中。

####实现局部引用

为了实现局部引用，Java虚拟机为每个从Java到本地方法的控制转换都创建了注册服务程序。注册服务程序将不可移动的局部引用
映射为Java对象，并防止这些对象被当作垃圾收集。所有传给本地方法的Java对象（包括那些作为JNI函数调用结果返回的对象）将
被自动添加到注册服务程序中。本地方法返回后，注册服务程序将被删除，其中的所有项都可以被当作垃圾来收集。可用各种不同的
方法来实现注册服务程序，例如，使用表、链接列表或hash表来实现。虽然引用计数可用来避免注册服务程序中有重复的项，但JNI
实现不是必须检测和消除重复的项。注意，以保守方式扫描本地堆栈并不能如实地实现局部引用。平台相关代码可将局部引用储存在
全局或堆数据结构中。

###访问Java对象

JNI提供了一大批用来访问全局引用和局部引用的函数。这意味着无论虚拟机在内部如何表示Java对象，相同的本地方法实现都能
工作。这就是为什么JNI可被各种各样的虚拟机实现所支持的关键原因。通过不透明的引用来使用访问函数的开销比直接访问C数据
结构的开销来得高。我们相信，大多数情况下，Java程序员使用本地方法是为了完成一些重要任务，此时这种接口的开销不是首要
问题。

####访问基本类型数组

对于含有大量基本数据类型（如整数数组和字符串）的Java对象来说，这种开销将高得不可接受（考虑一下用于执行矢量和矩阵运算
的本地方法的情形便知）。对Java数组进行迭代并且要通过函数调用取回数组的每个元素，其效率是非常低的。

一个解决办法是引入“钉住”概念，以使本地方法能够要求虚拟机钉住数组内容。而后，该本地方法将接受指向数值元素的直接指针。
但是，这种方法包含以下两个前提：

1. 垃圾收集器必须支持钉住。
2. 虚拟机必须在内存中连续存放基本类型数组。虽然大多数基本类型数组都是连续存放的，但布尔数组可以压缩或不压缩存储。因此，
依赖于布尔数组确切存储方式的本地方法将是不可移植的。

我们将采取折衷方法来克服上述两个问题。

首先，我们提供了一套函数，用于在Java数组的一部分和本地内存缓冲之间复制基本类型数组元素。这些函数只有在本地方法只需访问
大型数组中的一小部分元素时才使用。

其次，程序员可用另一套函数来取回数组元素的受约束版本。记住，这些函数可能要求Java虚拟机分配存储空间和进行复制。
虚拟机实现将决定这些函数是否真正复制该数组，如下所示：

1. 如果垃圾收集器支持钉住，且数组的布局符合本地方法的要求，则不需要进行复制。
2. 否则，该数组将被复制到不可移动的内存块中（例如，复制到C堆中），并进行必要的格式转换，然后返回指向该副本的指针。

最后，接口提供了一些函数，用以通知虚拟机本地方法已不再需要访问这些数组元素。当调用这些函数时，系统或者释放数组，
或者在原始数组与其不可移动副本之间进行协调并将副本释放。

这种处理方法具有灵活性。垃圾收集器的算法可对每个给定的数组分别作出复制或钉住的决定。例如，垃圾收集器可能复制小型
对象而钉住大型对象。JNI实现必须确保多个线程中运行的本地方法可同时访问同一数组。例如，JNI可以为每个被钉住的数组保留
一个内部计数器，以便某个线程不会解开同时被另一个线程钉住的数组。注意，JNI不必将基本类型数组锁住以专供某个本地方法
访问。同时从不同的线程对Java数组进行更新将导致不确定的结果。

####访问域和方法

JNI允许本地方法访问Java对象的域或调用其方法。JNI用符号名称和类型签名来识别方法和域。从名称和签名来定位域或对象的过程
可分为两步。例如，为调用类cls中的f方法，平台相关代码首先要获得方法ID，如下所示：

{% highlight ruby %}
jmethodID mid = env->GetMethodID(cls, "f", "(ILjava/lang/String;)D");
{% endhighlight %}

然后，平台相关代码可重复使用该方法ID而无须再查找该方法，如下所示：

{% highlight ruby %}
jdouble result = env->CallDoubleMethod(obj, mid, 10, str);
{% endhighlight %}

域ID或方法ID并不能防止虚拟机卸载生成该ID的类。该类被卸载之后，该方法ID或域ID亦变成无效。因此，如果平台相关代码要
长时间使用某个方法ID或域ID，则它必须确保：保留对所涉及类的活引用，或重新计算该方法ID或域ID。JNI对域ID和方法ID的
内部实现并不施加任何限制。

###报告编程错误

JNI不检查诸如传递NULL指针或非法参数类型之类的编程错误。非法的参数类型包括诸如要用Java类对象时却用了普通Java对象这样
的错误。JNI不检查这些编程错误的理由如下：

1. 强迫JNI函数去检查所有可能的错误情况将降低正常（正确）的本地方法的性能。
2. 在许多情况下，没有足够的运行时的类型信息可供这种检查使用。

大多数C库函数对编程错误不进行防范。例如，printf()函数在接到一个无效地址时通常是引起运行错而不是返回错误代码。强迫C库
函数检查所有可能的错误情况将有可能引起这种检查被重复进行--先是在用户代码中进行，然后又在库函数中再次进行。

程序员不得将非法指针或错误类型的参数传递给JNI函数。否则，可能产生意想不到的后果，包括可能使系统状态受损或使虚拟机崩溃。

###Java异常

JNI允许本地方法抛出任何Java异常。本地方法也可以处理突出的Java异常。未被处理的Java异常将被传回虚拟机中。

####异常和错误代码

一些JNI函数使用Java异常机制来报告错误情况。大多数情况下，JNI函数通过返回错误代码并抛出Java异常来报告错误情况。
错误代码通常是特殊的返回值（如 NULL），这种特殊的返回值在正常返回值范围之外。因此，程序员可以：快速检查上一个
JNI调用所返回的值以确定是否出错，并通过调用函数ExceptionOccurred()来获得异常对象，它含有对错误情况的更详细说明。

在以下两种情况中，程序员需要先查出异常，然后才能检查错误代码：

1. 调用Java方法的JNI函数返回该Java方法的结果。程序员必须调用ExceptionOccurred() 以检查在执行Java方法期间可能发生的异常。
2. 某些用于访问JNI数组的函数并不返回错误代码，但可能会抛出ArrayIndexOutOfBoundsException或ArrayStoreException。

在所有其它情况下，返回值如果不是错误代码值就可确保没有抛出异常。

####异步异常

在多个线程的情况下，当前线程以外的其它线程可能会抛出异步异常。异步异常并不立即影响当前线程中平台相关代码的执行，
直到出现下列情况：该平台相关代码调用某个有可能抛出同步异常的JNI函数，或者该平台相关代码用 ExceptionOccurred() 
显式检查同步异常或异步异常。

注意，只有那些有可能抛出同步异常的JNI函数才检查异步异常。本地方法应在必要的地方（例如，在一个没有其它异常检查的
紧密循环中）插入ExceptionOccurred() 检查以确保当前线程可在适当时间内对异步异常作出响应。

####异常的处理

可用两种方法来处理平台相关代码中的异常：

1. 本地方法可选择立即返回，使异常在启动该本地方法调用的Java代码中抛出。
2. 平台相关代码可通过调用ExceptionClear() 来清除异常，然后执行自己的异常处理代码。

抛出了某个异常之后，平台相关代码必须先清除异常，然后才能进行其它的JNI调用。当有待定异常时，只有以下这些JNI函数可被
安全地调用：ExceptionOccurred()、ExceptionDescribe()和ExceptionClear()。ExceptionDescribe()函数将打印有关待定异常的
调试消息。

<hr>

##JNI的类型和数据结构

本章讨论JNI如何将Java类型映射到本地C类型。

###基本类型

基本类型和本地等效类型表：

|| *Java类型* || *本地类型* || *说明* ||
|| boolean || jboolean || 无符号，8位 ||
|| byte || jbyte || 无符号，8位 ||
|| char || jchar || 无符号，16位 ||
|| short || jshort || 有符号，16位 ||
|| int || jint || 有符号，32位 ||
|| long || jlong || 有符号，64位 ||
|| float || jfloat || 32位 ||
|| double || jdouble || 64位 ||
|| void || void || N/A ||

为了使用方便，特提供以下定义：

{% highlight ruby %}
	#define JNI_FALSE 0
	#define JNI_TRUE 1
{% endhighlight %}

jsize整数类型用于描述主要指数和大小：

{% highlight ruby %}
	typedef jint jsize;
{% endhighlight %}

###引用类型

JNI包含了很多对应于不同Java对象的引用类型。JNI引用类型的组织层次如图所示：

<img src="http://yanbober.github.io/image/2015-2-14-android_studio_jni_1/3.png" />

在C中，所有其它JNI引用类型都被定义为与jobject一样。例如：

{% highlight ruby %}
typedef jobject jclass;
{% endhighlight %}

在C++中，JNI引入了虚构类以加强子类关系。例如：

{% highlight ruby %}
class _jobject {};
class _jclass : public _jobject {};
...
typedef _jobject *jobject;
typedef _jclass *jclass;
{% endhighlight %}

###域ID和方法ID

方法ID和域ID是常规的C指针类型：

{% highlight ruby %}
struct _jfieldID;
/*不透明结构 */
typedef struct _jfieldID *jfieldID;
/* 域 ID */
struct _jmethodID;
/* 不透明结构 */
typedef struct _jmethodID *jmethodID; /* 方法 ID */
{% endhighlight %}

###值类型

jvalue联合类型在参数数组中用作单元类型。其声明方式如下：

{% highlight ruby %}
typedef union jvalue {
	jboolean	z;
	jbyte	b;
	jchar	c;
	jshort	s;
	jint	i;
	jlong	j;
	jfloat	f;
	jdouble	d;
	jobject	l;
} jvalue;
{% endhighlight %}

###类型签名

JNI使用Java虚拟机的类型签名表述。下表列出了这些类型签名：

|| *类型签名* || *Java 类型* ||
|| Z || boolean ||
|| B || byte ||
|| C || char ||
|| S || short ||
|| I || int ||
|| J || long ||
|| F || float ||
|| D || double ||
|| L fully-qualified-class ; || 全限定的类 ||
|| [ type || type[] ||
|| ( arg-types ) ret-type || 方法类型 ||

例如，Java方法：

{% highlight ruby %}
long f (int n, String s, int[] arr);
{% endhighlight %}

具有以下类型签名：

{% highlight ruby %}
(ILjava/lang/String;[I)J
{% endhighlight %}

###UTF-8字符串

JNI用UTF-8字符串来表示各种字符串类型。UTF-8字符串和Java虚拟机所使用的一样。UTF-8字符串的编码方式使得仅包含非空ASCII
字符的字符序列能够按每字符一个字节表示，但是最多只能表示16位的字符。所有在\u0001到\u007F范围内的字符都用单字节表示，
如下所示：
	
	|0|0-6位|
	
字节中的七位数据确定了所表示字符的值。空字符 (\u000)和\u0080到\u07FF范围内的字符用一对字节表示， 即x和y，如下所示：

	x：|1|1|0|6-10位|				
	y：|1|0|0-5位|
	
值为((x&0x1f)<<6)+(y&0x3f)的字符需用两个字节表示。\u0800到\uFFFF范围内的字符用三个字节表示，即x，y和z：

	x：|1|1|1|0|12-15位|
	y：|1|0|6-11位|
	z：|1|0|0-5位|
	
值为((x&0xf)<<12)+(y&0x3f)<<6)+(z&0x3f)的字符需用三个字节表示。

此格式与“标准” UTF-8格式之间有两个区别。第一，空字节(byte)0使用双字节格式进行编码，而不是单字节格式。
这意味着Java虚拟机的UTF-8字符串不可能有嵌入的空值。第二，只使用单字节、双字节和三字节格式。Java虚拟
机不能识别更长的UTF-8格式。

<hr>

##JNI函数

本章为JNI函数提供参考信息。其中列出了全部JNI函数，同时也给出了JNI函数表的准确布局。注意：“必须”一词用于约束JNI编程
人员。例如，当说明某个JNI函数必须接收非空对象时，就应确保不要向该JNI函数传递NULL。这时，JNI实现将无需在该JNI函数中
执行NULL指针检查。本章的部分资料改编自Netscape的JRI文档。该参考资料按用法对函数进行组织。

###接口函数表

每个函数均可通过JNIEnv参数以固定偏移量进行访问。JNIEnv的类型是一个指针，指向存储全部JNI函数指针的结构。注意：前三项
留作将来与COM兼容。此外，我们在函数表开头部分也留出来多个NULL项，从而可将将来与类有关的JNI操作添加到FindClass后面，
而非函数表的末尾。注意，函数表可在所有JNI接口指针间共享。

{% highlight ruby %}
const struct JNINativeInterface ... = {
	NULL,
	NULL,
	NULL,
	NULL,
	GetVersion,

	DefineClass,
	FindClass,
	NULL,
	NULL,
	NULL,
	GetSuperclass,
	IsAssignableFrom,
	NULL,

	Throw,
	ThrowNew,
	ExceptionOccurred,
	ExceptionDescribe,
	ExceptionClear,
	FatalError,
	NULL,
	NULL,

	NewGlobalRef,
	DeleteGlobalRef,
	DeleteLocalRef,
	IsSameObject,
	NULL,
	NULL,
	AllocObject,

	NewObject,
	NewObjectV,
	NewObjectA,
	GetObjectClass,

	IsInstanceOf,

	GetMethodID,

	CallObjectMethod,
	CallObjectMethodV,
	CallObjectMethodA,
	CallBooleanMethod,
	CallBooleanMethodV,
	CallBooleanMethodA,
	CallByteMethod,
	CallByteMethodV,
	CallByteMethodA,
	CallCharMethod,
	CallCharMethodV,
	CallCharMethodA,
	CallShortMethod,
	CallShortMethodV,
	CallShortMethodA,
	CallIntMethod,
	CallIntMethodV,
	CallIntMethodA,
	CallLongMethod,
	CallLongMethodV,
	CallLongMethodA,
	CallFloatMethod,
	CallFloatMethodV,
	CallFloatMethodA,
	CallDoubleMethod,
	CallDoubleMethodV,
	CallDoubleMethodA,
	CallVoidMethod,
	CallVoidMethodV,
	CallVoidMethodA,

	CallNonvirtualObjectMethod,
	CallNonvirtualObjectMethodV,
	CallNonvirtualObjectMethodA,
	CallNonvirtualBooleanMethod,
	CallNonvirtualBooleanMethodV,
	CallNonvirtualBooleanMethodA,
	CallNonvirtualByteMethod,
	CallNonvirtualByteMethodV,
	CallNonvirtualByteMethodA,
	CallNonvirtualCharMethod,
	CallNonvirtualCharMethodV,
	CallNonvirtualCharMethodA,
	CallNonvirtualShortMethod,
	CallNonvirtualShortMethodV,
	CallNonvirtualShortMethodA,
	CallNonvirtualIntMethod,
	CallNonvirtualIntMethodV,
	CallNonvirtualIntMethodA,
	CallNonvirtualLongMethod,
	CallNonvirtualLongMethodV,
	CallNonvirtualLongMethodA,
	CallNonvirtualFloatMethod,
	CallNonvirtualFloatMethodV,
	CallNonvirtualFloatMethodA,
	CallNonvirtualDoubleMethod,
	CallNonvirtualDoubleMethodV,
	CallNonvirtualDoubleMethodA,
	CallNonvirtualVoidMethod,
	CallNonvirtualVoidMethodV,
	CallNonvirtualVoidMethodA,

	GetFieldID,

	GetObjectField,
	GetBooleanField,
	GetByteField,
	GetCharField,
	GetShortField,
	GetIntField,
	GetLongField,
	GetFloatField,
	GetDoubleField,
	SetObjectField,
	SetBooleanField,
	SetByteField,
	SetCharField,
	SetShortField,
	SetIntField,
	SetLongField,
	SetFloatField,
	SetDoubleField,

	GetStaticMethodID,

	CallStaticObjectMethod,
	CallStaticObjectMethodV,
	CallStaticObjectMethodA,
	CallStaticBooleanMethod,
	CallStaticBooleanMethodV,
	CallStaticBooleanMethodA,
	CallStaticByteMethod,
	CallStaticByteMethodV,
	CallStaticByteMethodA,
	CallStaticCharMethod,
	CallStaticCharMethodV,
	CallStaticCharMethodA,
	CallStaticShortMethod,
	CallStaticShortMethodV,
	CallStaticShortMethodA,
	CallStaticIntMethod,
	CallStaticIntMethodV,
	CallStaticIntMethodA,
	CallStaticLongMethod,
	CallStaticLongMethodV,
	CallStaticLongMethodA,
	CallStaticFloatMethod,
	CallStaticFloatMethodV,
	CallStaticFloatMethodA,
	CallStaticDoubleMethod,
	CallStaticDoubleMethodV,
	CallStaticDoubleMethodA,
	CallStaticVoidMethod,
	CallStaticVoidMethodV,
	CallStaticVoidMethodA,

	GetStaticFieldID,

	GetStaticObjectField,
	GetStaticBooleanField,
	GetStaticByteField,
	GetStaticCharField,
	GetStaticShortField,
	GetStaticIntField,
	GetStaticLongField,
	GetStaticFloatField,
	GetStaticDoubleField,

	SetStaticObjectField,
	SetStaticBooleanField,
	SetStaticByteField,
	SetStaticCharField,
	SetStaticShortField,
	SetStaticIntField,
	SetStaticLongField,
	SetStaticFloatField,
	SetStaticDoubleField,

	NewString,
	GetStringLength,
	GetStringChars,
	ReleaseStringChars,
	NewStringUTF,
	GetStringUTFLength,
	GetStringUTFChars,
	ReleaseStringUTFChars,

	GetArrayLength,

	NewObjectArray,
	GetObjectArrayElement,
	SetObjectArrayElement,

	NewBooleanArray,
	NewByteArray,
	NewCharArray,
	NewShortArray,
	NewIntArray,
	NewLongArray,
	NewFloatArray,
	NewDoubleArray,

	GetBooleanArrayElements,
	GetByteArrayElements,
	GetCharArrayElements,
	GetShortArrayElements,
	GetIntArrayElements,
	GetLongArrayElements,
	GetFloatArrayElements,
	GetDoubleArrayElements,

	ReleaseBooleanArrayElements,
	ReleaseByteArrayElements,
	ReleaseCharArrayElements,
	ReleaseShortArrayElements,
	ReleaseIntArrayElements,
	ReleaseLongArrayElements,
	ReleaseFloatArrayElements,
	ReleaseDoubleArrayElements,

	GetBooleanArrayRegion,
	GetByteArrayRegion,
	GetCharArrayRegion,
	GetShortArrayRegion,
	GetIntArrayRegion,
	GetLongArrayRegion,
	GetFloatArrayRegion,
	GetDoubleArrayRegion,
	SetBooleanArrayRegion,
	SetByteArrayRegion,
	SetCharArrayRegion,
	SetShortArrayRegion,
	SetIntArrayRegion,
	SetLongArrayRegion,
	SetFloatArrayRegion,
	SetDoubleArrayRegion,

	RegisterNatives,
	UnregisterNatives,

	MonitorEnter,
	MonitorExit,

	GetJavaVM,
};
{% endhighlight %}

###版本信息

####GetVersion 返回本地方法接口的版本。

{% highlight ruby %}
jint GetVersion(JNIEnv *env);
{% endhighlight %}

参数
	env：JNI接口指针。
	
返回值：
	高16位返回主版本号，低16位返回次版本号。
	在JDK1.1 中，GetVersion()返回0x00010001。

###类操作

####DefineClass 从原始类数据的缓冲区中加载类。

{% highlight ruby %}
jclass DefineClass(JNIEnv *env, jobject loader, const jbyte *buf, jsize bufLen);
{% endhighlight %}

参数：
	env：JNI 接口指针。
	loader：分派给所定义的类的类加载器。
	buf：包含.class文件数据的缓冲区。
	bufLen：缓冲区长度。
	
返回值：
	返回Java类对象。如果出错则返回NULL。
	
抛出：
	ClassFormatError：如果类数据指定的类无效。
	ClassCircularityError：如果类或接口是自身的超类或超接口。
	OutOfMemoryError：如果系统内存不足。

####FindClass 该函数用于加载本地定义的类。它将搜索由CLASSPATH环境变量为具有指定名称的类所指定的目录和zip文件。

{% highlight ruby %}
jclass FindClass(JNIEnv *env, const char *name);
{% endhighlight %}

参数：
	env：JNI接口指针。
	name：类全名（即包名后跟类名，之间由“/”分隔）。如果该名称以“[”（数组签名字符）打头，则返回一个数组类。

返回值：
	返回类对象全名。如果找不到该类，则返回NULL。

抛出：
	ClassFormatError：如果类数据指定的类无效。
	ClassCircularityError：如果类或接口是自身的超类或超接口。
	NoClassDefFoundError：如果找不到所请求的类或接口的定义。
	OutOfMemoryError：如果系统内存不足。

####GetSuperclass 如果clazz代表类而非类object，则该函数返回由clazz所指定的类的超类。如果clazz指定类object或代表某个接口，则该函数返回NULL。

{% highlight ruby %}
jclass GetSuperclass(JNIEnv *env, jclass clazz);
{% endhighlight %}

参数：
	env：JNI接口指针。
	clazz：Java类对象。
	
返回值：
	由clazz所代表的类的超类或NULL。
	
####IsAssignableFrom 确定clazz1的对象是否可安全地强制转换为clazz2。

{% highlight ruby %}
jboolean IsAssignableFrom(JNIEnv *env, jclass clazz1, jclass clazz2);
{% endhighlight %}

参数：
	env：JNI接口指针。
	clazz1：第一个类参数。
	clazz2：第二个类参数。
	
返回值：
	下列某个情况为真时返回JNI_TRUE：
		第一及第二个类参数引用同一个Java类。
		第一个类是第二个类的子类。
		第二个类是第一个类的某个接口。
		
###异常

####Throw 抛出java.lang.Throwable对象。

{% highlight ruby %}
jint Throw(JNIEnv *env, jthrowable obj);
{% endhighlight %}

参数：
	env：JNI接口指针。
	obj：java.lang.Throwable对象。
	
返回值：
	成功时返回0，失败时返回负数。
	
抛出：
	java.lang.Throwable对象obj。
	
####ThrowNew利用指定类的消息（由message指定）构造异常对象并抛出该异常。

{% highlight ruby %}
jint ThrowNew(JNIEnv *env, jclass clazz, const char *message);
{% endhighlight %}

参数：
	env：JNI接口指针。
	clazz：java.lang.Throwable的子类。
	message：用于构造java.lang.Throwable对象的消息。

返回值：
	成功时返回0，失败时返回负数。
	
抛出：
	新构造的java.lang.Throwable对象。
	
####ExceptionOccurred 确定是否某个异常正被抛出。在平台相关代码调用ExceptionClear()或Java代码处理该异常前，异常将始终保持抛出状态。

{% highlight ruby %}
jthrowable ExceptionOccurred(JNIEnv *env);
{% endhighlight %}

参数：
	env：JNI接口指针。
	
返回值：
	返回正被抛出的异常对象，如果当前无异常被抛出，则返回NULL。
	
####ExceptionDescribe 将异常及堆栈的回溯输出到系统错误报告信道（例如 stderr）。该例程可便利调试操作。

{% highlight ruby %}
void ExceptionDescribe(JNIEnv *env);
{% endhighlight %}

参数：
	env：JNI接口指针。
	
####ExceptionClear 清除当前抛出的任何异常。如果当前无异常，则此例程不产生任何效果。

{% highlight ruby %}
void ExceptionClear(JNIEnv *env);
{% endhighlight %}

参数：
	env：JNI接口指针。
	
####FatalError 抛出致命错误并且不希望虚拟机进行修复。该函数无返回值。

{% highlight ruby %}
void FatalError(JNIEnv *env, const char *msg);
{% endhighlight %}

参数：
	env：JNI接口指针。
	msg：错误消息。
	
###全局及局部引用

####NewGlobalRef 创建obj参数所引用对象的新全局引用。obj参数既可以是全局引用，也可以是局部引用。全局引用通过调用DeleteGlobalRef()来显式撤消。

{% highlight ruby %}
jobject NewGlobalRef(JNIEnv *env, jobject obj);
{% endhighlight %}

参数：
	env：JNI接口指针。
	obj：全局或局部引用。

返回值：
	返回全局引用。如果系统内存不足则返回NULL。
	
####DeleteGlobalRef 删除globalRef所指向的全局引用。

{% highlight ruby %}
void DeleteGlobalRef(JNIEnv *env, jobject globalRef);
{% endhighlight %}

参数：
	env：JNI接口指针。
	globalRef：全局引用。
	
####DeleteLocalRef 删除localRef所指向的局部引用。

{% highlight ruby %}
void DeleteLocalRef(JNIEnv *env, jobject localRef);
{% endhighlight %}

参数：
	env：JNI接口指针。
	localRef：局部引用。
	
###对象操作

####AllocObject 分配新Java对象而不调用该对象的任何构造函数。返回该对象的引用。clazz参数务必不要引用数组类。

{% highlight ruby %}
jobject AllocObject(JNIEnv *env, jclass clazz);
{% endhighlight %}

参数：
	env：JNI接口指针。
	clazz：Java类对象。
	
返回值：
	返回Java对象。如果无法构造该对象，则返回NULL。
	
抛出：
	InstantiationException：如果该类为一个接口或抽象类。
	OutOfMemoryError：如果系统内存不足。
	
####NewObject NewObjectA NewObjectV 构造新Java对象。

{% highlight ruby %}
jobject NewObject(JNIEnv *env, jclass clazz, jmethodID methodID, ...);
jobject NewObjectA(JNIEnv *env, jclass clazz, jmethodID methodID, jvalue *args);
jobject NewObjectV(JNIEnv *env, jclass clazz, jmethodID methodID, va_list args);
{% endhighlight %}

方法ID指示应调用的构造函数方法。该ID必须通过调用GetMethodID()获得，且调用时的方法名必须为<init>，
而返回类型必须为void(V)。clazz参数务必不要引用数组类。

#####NewObject

编程人员应将传递给构造函数的所有参数紧跟着放在methodID参数的后面。NewObject()收到这些参数后，将把它们传给编程人员
所要调用的Java方法。

#####NewObjectA

编程人员应将传递给构造函数的所有参数放在jvalues类型的数组args中，该数组紧跟着放在methodID参数的后面。NewObject()
收到数组中的这些参数后，将把它们传给编程人员所要调用的Java方法。

#####NewObjectV

编程人员应将传递给构造函数的所有参数放在va_list类型的参数args中，该参数紧跟着放在methodID参数的后面。NewObject()收到
这些参数后，将把它们传给编程人员所要调用的Java方法。

参数：
	env：JNI接口指针。
	clazz：Java类对象。
	methodID：构造函数的方法ID。

NewObject 的其它参数：
	传给构造函数的参数。

NewObjectA的其它参数：
	args：传给构造函数的参数数组。

NewObjectV的其它参数：
	args：传给构造函数的参数va_list。

返回值：
返回Java对象，如果无法构造该对象，则返回NULL。

抛出：
	InstantiationException：如果该类为接口或抽象类。
	OutOfMemoryError：如果系统内存不足。
	构造函数抛出的任何异常。

####GetObjectClass 返回对象的类。

{% highlight ruby %}
jclass GetObjectClass(JNIEnv *env, jobject obj);
{% endhighlight %}

参数：
	env：JNI接口指针。
	obj：Java对象（不能为NULL）。

返回值：
	返回Java类对象。

####IsInstanceOf 测试对象是否为某个类的实例。

{% highlight ruby %}
jboolean IsInstanceOf(JNIEnv *env, jobject obj, jclass clazz);
{% endhighlight %}

参数：
	env：JNI接口指针。
	obj：Java对象。
	clazz：Java类对象。
	
返回值：
	如果可将obj强制转换为clazz，则返回JNI_TRUE。否则返回JNI_FALSE。NULL对象可强制转换为任何类。
	
####IsSameObject 测试两个引用是否引用同一Java对象。

{% highlight ruby %}
jboolean IsSameObject(JNIEnv *env, jobject ref1, jobject ref2);
{% endhighlight %}

参数：
	env：JNI接口指针。
	ref1：Java对象。
	ref2：Java对象。
	
返回值：
	如果ref1和ref2引用同一Java对象或均为NULL，则返回JNI_TRUE。否则返回JNI_FALSE。

###访问对象的域

####GetFieldID

{% highlight ruby %}
jfieldID GetFieldID(JNIEnv *env, jclass clazz, const char *name, const char *sig);
{% endhighlight %}

返回类的实例（非静态）域的域ID。该域由其名称及签名指定。访问器函数的Get<type>Field及Set<type>Field系列使用域ID检索
对象域。GetFieldID()将未初始化的类初始化。GetFieldID()不能用于获取数组的长度域。应使用GetArrayLength()。

参数：
	env：JNI接口指针。
	clazz：Java类对象。
	name: 0终结的UTF-8字符串中的域名。
	sig：0终结的UTF-8字符串中的域签名。
	
返回值：
	域ID。如果操作失败，则返回NULL。
	
抛出：
	NoSuchFieldError：如果找不到指定的域。
	ExceptionInInitializerError：如果由于异常而导致类初始化程序失败。
	OutOfMemoryError：如果系统内存不足。

####Get<type>Field例程

{% highlight ruby %}
NativeType Get<type>Field(JNIEnv *env, jobject obj, jfieldID fieldID);
{% endhighlight %}

该访问器例程系列返回对象的实例（非静态）域的值。要访问的域由通过调用GetFieldID()而得到的域ID指定。下表说明了
Get<type>Field例程名及结果类型。应将Get<type>Field中的type替换为域的Java类型（或使用表中的某个实际例程名），
然后将NativeType替换为该例程对应的本地类型。

|| *Get<type>Field例程名* || *本地类型* ||
|| GetObjectField() || jobject ||
|| GetBooleanField() || jboolean ||
|| GetByteField() || jbyte ||
|| GetCharField() || jchar ||
|| GetShortField() || jshort ||
|| GetIntField() || jint ||
|| GetLongField() || jlong ||
|| GetFloatField() || jfloat ||
|| GetDoubleField() || jdouble ||

参数：
	env：JNI接口指针。
	obj：Java对象（不能为 NULL）。
	fieldID：有效的域ID。
	
返回值：
	域的内容。
	
####Set<type>Field例程

{% highlight ruby %}
void Set<type>Field(JNIEnv *env, jobject obj, jfieldID fieldID, NativeType value);
{% endhighlight %}

该访问器例程系列设置对象的实例（非静态）域的值。要访问的域由通过调用SetFieldID()而得到的域ID指定。下表说明了
Set<type>Field例程名及结果类型。应将Set<type>Field中的type替换为域的Java类型（或使用表中的某个实际例程名），
然后将NativeType替换为该例程对应的本地类型。

|| *Set<type>Field例程名* || *本地类型* ||
|| SetObjectField() || jobject ||
|| SetBooleanField() || jboolean ||
|| SetByteField() || jbyte ||
|| SetCharField() || jchar ||
|| SetShortField() || jshort ||
|| SetIntField() || jint ||
|| SetLongField() || jlong ||
|| SetFloatField() || jfloat ||
|| SetDoubleField() || jdouble ||

参数：
	env：JNI接口指针。
	obj：Java对象（不能为NULL）。
	fieldID：有效的域ID。
	value：域的新值。

###调用实例方法

####GetMethodID

{% highlight ruby %}
jmethodID GetMethodID(JNIEnv *env, jclass clazz, const char *name, const char *sig);
{% endhighlight %}

返回类或接口实例（非静态）方法的方法ID。方法可在某个clazz的超类中定义，也可从clazz继承。该方法由其名称和签名决定。
GetMethodID()可使未初始化的类初始化。要获得构造函数的方法ID，应将<init>作为方法名，同时将void (V)作为返回类型。

参数：
	env：JNI接口指针。
	clazz：Java类对象。
	name：0终结的UTF-8字符串中的方法名。
	sig：0终结的UTF-8字符串中的方法签名。

返回值：
	方法ID，如果找不到指定的方法，则为NULL。

抛出：
	NoSuchMethodError：如果找不到指定方法。
	ExceptionInInitializerError：如果由于异常而导致类初始化程序失败。
	OutOfMemoryError：如果系统内存不足。

####Call<type>Method，Call<type>MethodA，Call<type>MethodV例程

{% highlight ruby %}
NativeType Call<type>Method(JNIEnv *env, jobject obj, jmethodID methodID, ...);
NativeType Call<type>MethodA(JNIEnv *env, jobject obj, jmethodID methodID, jvalue *args);
NativeType Call<type>MethodV(JNIEnv *env, jobject obj, jmethodID methodID, va_list args);
{% endhighlight %}

这三个操作的方法用于从本地方法调用Java实例方法。它们的差别仅在于向其所调用的方法传递参数时所用的机制。这三个操作将
根据所指定的方法ID调用Java对象的实例（非静态）方法。参数methodID必须通过调用GetMethodID()来获得。当这些函数用于调用
私有方法和构造函数时，方法ID必须从obj的真实类派生而来，而不应从其某个超类派生。

#####Call<type>Method例程

编程人员应将要传给方法的所有参数紧跟着放在methodID参数之后。Call<type>Method例程接受这些参数并将其传给编程人员所要
调用的Java方法。

#####Call<type>MethodA例程

编程人员应将要传给方法的所有参数放在紧跟在methodID参数之后的jvalues类型数组args中。Call<type>MethodA routine接受这些
数组中的参数并将其传给编程人员所要调用的Java方法。

#####Call<type>MethodV 例程

编程人员将方法的所有参数放在紧跟着在methodID参数之后的va_list类型参数变量中。Call<type>MethodV routine接受这些参数
并将其传给编程人员所要调用的Java方法。

下表根据结果类型说明了各个方法调用例程。用户应将Call<type>Method中的type替换为所调用方法的Java类型（或使用表中的实际
方法调用例程名），同时将NativeType替换为该例程相应的本地类型。

|| *Call<type>Method例程名* || *本地类型* ||
|| CallVoidMethod() CallVoidMethodA() CallVoidMethodV() || void ||
|| CallObjectMethod() CallObjectMethodA() CallObjectMethodV() || jobject ||
|| CallBooleanMethod() CallBooleanMethodA() CallBooleanMethodV() || jboolean ||
|| CallByteMethod() CallByteMethodA() CallByteMethodV() || jbyte ||
|| CallCharMethod() CallCharMethodA() CallCharMethodV() || jchar ||
|| CallShortMethod() CallShortMethodA() CallShortMethodV() || jshort ||
|| CallIntMethod() CallIntMethodA() CallIntMethodV() || jint ||
|| CallLongMethod() CallLongMethodA() CallLongMethodV() || jlong ||
|| CallFloatMethod() CallFloatMethodA() CallFloatMethodV() || jfloat ||
|| CallDoubleMethod() CallDoubleMethodA() CallDoubleMethodV() || jdouble ||

参数：
	env：JNI接口指针。
	obj：Java对象。
	methodID：方法ID。
	
Call<type>Method例程的其它参数：
	要传给Java方法的参数。
	
Call<type>MethodA例程的其它参数：
	args：参数数组。
	
Call<type>MethodV例程的其它参数：
	args：参数的va_list。

返回值：
	返回调用Java方法的结果。

抛出：
	执行Java方法时抛出的异常。
	
####CallNonvirtual<type>Method，CallNonvirtual<type>MethodA，CallNonvirtual<type>MethodV例程

{% highlight ruby %}
NativeType CallNonvirtual<type>Method(JNIEnv *env, jobject obj, jclass clazz, jmethodID methodID, ...);
NativeType CallNonvirtual<type>MethodA(JNIEnv *env, jobject obj, jclass clazz, jmethodID methodID, jvalue *args);
NativeType CallNonvirtual<type>MethodV(JNIEnv *env, jobject obj, jclass clazz, jmethodID methodID, va_list args);
{% endhighlight %}

这些操作根据指定的类和方法ID调用某Java对象的实例（非静态）方法。参数methodID必须通过调用clazz类的GetMethodID()获得。
CallNonvirtual<type>Method和Call<type>Method例程系列并不相同。Call<type>Method例程根据对象的类调用方法，而
CallNonvirtual<type>Method例程则根据获得方法ID的（由clazz参数指定）类调用方法。方法ID必须从对象的真实类或其某个超类获得。

#####CallNonvirtual<type>Method例程

编程人员应将要传给方法的所有参数紧跟着放在methodID参数之后。CallNonvirtual<type>Method routine接受这些参数并将其传给编
程人员所要调用的Java方法。

#####CallNonvirtual<type>MethodA例程

编程人员应将要传给方法的所有参数放在紧跟在methodID参数之后的jvalues类型数组args中。
CallNonvirtual<type>MethodA routine接受这些数组中的参数并将其传给编程人员所要调用的Java方法。

#####CallNonvirtual<type>MethodV例程

编程人员应将要传给方法的所有参数放在紧跟在methodID参数之后的va_list类型参数args中。CallNonvirtualMethodV routine接受
这些参数并将其传给编程人员所要调用的Java方法。

下表根据结果类型说明了各个方法调用例程。用户应将CallNonvirtual<type>Method中的type替换为所调用方法的Java类型（或使用
表中的实际方法调用例程名），同时将NativeType替换为该例程相应的本地类型。

|| *CallNonvirtual<type>Method例程名* || *本地类型* ||
|| CallNonvirtualVoidMethod() CallNonvirtualVoidMethodA() CallNonvirtualVoidMethodV() || void ||
|| CallNonvirtualObjectMethod() CallNonvirtualObjectMethodA() CallNonvirtualObjectMethodV() || jobject ||
|| CallNonvirtualBooleanMethod() CallNonvirtualBooleanMethodA() CallNonvirtualBooleanMethodV() || jboolean ||
|| CallNonvirtualByteMethod() CallNonvirtualByteMethodA() CallNonvirtualByteMethodV() || jbyte ||
|| CallNonvirtualCharMethod() CallNonvirtualCharMethodA() CallNonvirtualCharMethodV() || jchar ||
|| CallNonvirtualShortMethod() CallNonvirtualShortMethodA() CallNonvirtualShortMethodV() || jshort ||
|| CallNonvirtualIntMethod() CallNonvirtualIntMethodA() CallNonvirtualIntMethodV() || jint ||
|| CallNonvirtualLongMethod() CallNonvirtualLongMethodA() CallNonvirtualLongMethodV() || jlong ||
|| CallNonvirtualFloatMethod() CallNonvirtualFloatMethodA() CallNonvirtualFloatMethodV() || jfloat ||
|| CallNonvirtualDoubleMethod() CallNonvirtualDoubleMethodA() CallNonvirtualDoubleMethodV() || jdouble ||
	
参数：
	env：JNI接口指针。
	clazz：Java类。
	obj: Java对象。
	methodID：方法ID。
	
CallNonvirtual<type>Method例程的其它参数：
	要传给Java方法的参数。
	
CallNonvirtual<type>MethodA例程的其它参数：
	args：参数数组。
	
CallNonvirtual<type>MethodV例程的其它参数：
	args：参数的va_list。
	
返回值：
	调用Java方法的结果。

抛出：
	执行Java方法时所抛出的异常。
	
###访问静态域

####
<img src="http://yanbober.github.io/image/2015-2-14-android_studio_jni_1/2.png" />


{% highlight ruby %}

{% endhighlight %}

<img src="http://yanbober.github.io/image/2015-2-14-android_studio_jni_1/2.png" />

	（烦请令尊体谅作者劳动成果，转载麻烦声明文章链接。您的声明与讨论是鄙人写作的动力。JNI NDK系列文章依据时间及个人情况持续更新中......）
