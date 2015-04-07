---
layout: post
keywords: jni, Android Studio, ndk
description: Android Studio NDK, Android Studio JNI, JNI, NDK
title: "NDK-JNI实战教程（三） 从一个比Hello World稍微复杂一丁点儿的例子说说模板"
categories: [NDK-JNI开发]
tags: [NDK, JNI]
group: archive
icon: file-alt
---
{% include site/setup %}

#**第一部分**

<hr>

##概述

学习JNI NDK你需要有java与C或者C++基础。因为NDK几乎就是java与C或者C++的混合编程互调，JNI在其中只是扮演了一个不同语种间对接握手调运的规则而已。
就像C语言嵌入调运执行汇编程序一样，需要一种规则来约束沟通。这个例子是我在闲时继续使用Android Studio撸的，不难，适合入门。
不要一下子被这么几个文件吓着了。重点是为了通过这个例子引出来几个Android NDK开发的重要基础模板知识点。
所以内在代码逻辑看上去可能十分僵硬不合理，代码风格可能也不是十分规范，还请多多指点交流，然后撸的更多。

需要知识点：C语言基础，C语言动态参数宏，Java基础，JNI基本概念

<hr>

##代码及工程文件介绍

这个例子是一个简单的场景模拟实现；我们通过在app java层传入一个name到c库中，c库通过app传入的name经过保密的自定义加密算法（本代码没实现，只是模拟）
处理生成一个客户化定制的key反馈给app层使用。这样至于通过name得到key的具体加密机制被编译成了so文件，很难被破解。而如果使用java则很容易被破解。

###这是这篇文章要介绍的代码工程的几个主要文件夹文件分布情况：

<img src="http://yanbober.github.io/image/2015-2-14-android_studio_jni_1/4.png" />

浅析：正常NDK工程目录结构，其中jni目录下只是多包涵了两个文件夹而已。在这里在jni根目录下的两个文件就是jni核心文件，起到C与Java的互联互通作用；utils
目录是我自己加入的一个常用工具目录，里面放置一些通用代码，譬如这里的android_log_print.h用来打印log；local_logic_c目录是我放置的用C语言实现的加密逻辑
代码，其中包含实现和头文件。你的jni目录结构也可以随意组织，符合自己习惯效率就行。在这里需要注意的一点是Android JNI下面c代码使用printf打印是不显示的，
所以才需要像我加入的宏，使用android提供的log打印函数，不过在编译时请记得加入log依赖的官方lib。

###io.github.yanbober.ndkapplication包中MainActivity主Activity代码：

{% highlight ruby %}

package io.github.yanbober.ndkapplication;

import android.os.Bundle;
import android.support.v7.app.ActionBarActivity;
import android.widget.TextView;

public class MainActivity extends ActionBarActivity {
    private TextView mTextView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mTextView = (TextView) this.findViewById(R.id.test);

        NdkJniUtils jni = new NdkJniUtils();
        //传入name="vip"到jni代码模拟拿到加密后的key
        mTextView.setText(jni.generateKey("vip"));
    }
}
	
{% endhighlight %}

浅析：这就是App的传统界面了，一个UI传入name="vip"，调运native方法取到转换好的key显示在TextView里，没啥技术难度。

###io.github.yanbober.ndkapplication包中NdkJniUtils类代码：

{% highlight ruby %}

package io.github.yanbober.ndkapplication;

public class NdkJniUtils {
    public native String generateKey(String name);

    static {
        System.loadLibrary("YanboberJniLibName");
    }
}
		
{% endhighlight %}

浅析：这个类就是定义本地native方法，编译以后通过javah生成这个文件的h头文件，如下文。其中static块作用就不说了吧。
System.loadLibrary("YanboberJniLibName");就是加载你编译生成的库文件，注意库生成在lib目下默认会添加lib前缀，
形如：libXxx.so，我们在load函数里传入的名字只需要Xxx就行。

###jni根目录下通过系列教程一中javah生成的头文件io_github_yanbober_ndkapplication_NdkJniUtils.h内容：

{% highlight ruby %}

/* DO NOT EDIT THIS FILE - it is machine generated */
#include <jni.h>

#ifndef _Included_io_github_yanbober_ndkapplication_NdkJniUtils
#define _Included_io_github_yanbober_ndkapplication_NdkJniUtils
#ifdef __cplusplus
extern "C" {
#endif

JNIEXPORT jstring JNICALL Java_io_github_yanbober_ndkapplication_NdkJniUtils_generateKey(JNIEnv *, jobject, jstring);

#ifdef __cplusplus
}
#endif
#endif
	
{% endhighlight %}

浅析：通过javah生成的头文件，不明白的参考系列教程一中。

###jni根目录下通过系列教程一中类似test生成的jni接口c文件jni_interface.c内容：

{% highlight ruby %}

#include <jni.h>
#include <string.h>
#include "io_github_yanbober_ndkapplication_NdkJniUtils.h"
#include "./utils/android_log_print.h"
#include "./local_logic_c/easy_encrypt.h"

JNIEXPORT jstring JNICALL Java_io_github_yanbober_ndkapplication_NdkJniUtils_generateKey
  (JNIEnv *env, jobject obj, jstring name){
     //声明局部量
     char key[KEY_SIZE] = {0};
     memset(key, 0, sizeof(key));

     char temp[KEY_NAME_SIZE] = {0};

     //将java传入的name转换为本地utf的char*
     const char* pName = (*env)->GetStringUTFChars(env, name, NULL);

     if (NULL != pName) {
        strcpy(temp, pName);
        strcpy(key, generateKeyRAS(temp));

        //java的name对象不需要再使用，通知虚拟机回收name
        (*env)->ReleaseStringUTFChars(env, name, pName);
     }

     return (*env)->NewStringUTF(env, key);
  }	
	
{% endhighlight %}

浅析：jni"接口封装实现"文件，我就叫这名吧，可能好理解些，别把jni想的太高大上。这里面就是实现h文件声明的函数。
一些基本参数可以查阅系列教程二文档，复制关键字在教程二里搜索查阅即可。主要流程就是通过GetStringUTFChars拿到java
传入的String的name转换后的char* utf-8指针；把name通过generateKeyRAS传入C语言实现的加密逻辑代码中处理，同时通过ReleaseStringUTFChars
告诉虚拟机不需要持有name的引用，以便Java释放String的name；完事将C语言处理生成的key通过NewStringUTF转换返回给java层使用。

###jni目录下utils子目录下的log打印工具宏android_log_print.h文件内容：

{% highlight ruby %}

/*
 * 作者：工匠若水
 * 说明：Android JNI Log打印宏定义文件
 */

#ifndef _ANDROID_LOG_PRINT_H_
#define _ANDROID_LOG_PRINT_H_

#include <android/log.h>

#define IS_DEBUG

#ifdef IS_DEBUG

#define LOG_TAG ("CUSTOMER_NDK_JNI")

#define LOGV(...) ((void)__android_log_print(ANDROID_LOG_VERBOSE, LOG_TAG, __VA_ARGS__))

#define LOGD(...) ((void)__android_log_print(ANDROID_LOG_DEBUG  , LOG_TAG, __VA_ARGS__))

#define LOGI(...) ((void)__android_log_print(ANDROID_LOG_INFO   , LOG_TAG, __VA_ARGS__))

#define LOGW(...) ((void)__android_log_print(ANDROID_LOG_WARN   , LOG_TAG, __VA_ARGS__))

#define LOGE(...) ((void)__android_log_print(ANDROID_LOG_ERROR  , LOG_TAG, __VA_ARGS__))

#else

#define LOGV(LOG_TAG, ...) NULL

#define LOGD(LOG_TAG, ...) NULL

#define LOGI(LOG_TAG, ...) NULL

#define LOGW(LOG_TAG, ...) NULL

#define LOGE(LOG_TAG, ...) NULL

#endif

#endif
		
{% endhighlight %}

浅析：这个文件是我自己写JNI时每次直接使用的文件，就是一个工具文件一样。目的是因为Android的JNI使用printf函数打印的东西
是没法显示，这里这么转化其实对应的就是java层打印Log的函数Log.d(), Log.i(), Log.w(),Log.e(), Log.f()。原因是因为Android
的java层和C++ framework层都提供了Log函数，但是JNI环境下打印稍有不同，使用的是__android_log_print并且用NDK环境编译和
android源码framework环境编译选择链接Android.mk库也不同。所以你会发现Google NDK官方sample代码中也是类似处理的，这里只是
简单封装的更实用而已。需要一点C语言知识理解。如果你喜欢再往深里折腾，那我再提一点吧，那就是自己去android系统源码的
system/core/include/cutils/log.h去看看吧，如果是在完整源码编译环境下，只要include<utils/Log.h>头文件，就可以直接使用对应的LOGI、LOGD等方法了，
同时请定义LOG_TAG，LOG_NDEBUG等宏值。哈哈，又扯到系统了。不扯了。反正记得include不同情况下可以有不同的路径h文件，但是本质都是一样的，
只是封装了而已。

###jni目录下local_logic_c子目录中本地C语言实现的逻辑目录下的接口头文件easy_encrypt.h内容：

{% highlight ruby %}

#ifndef _EASY_ENCRYPT_H_
#define _EASY_ENCRYPT_H_
/*
 * 作者：晏博
 *
 * 功能：通过name获取加密后的key
 * 类型：测试代码
 */
#define KEY_NAME_SIZE  (6)
#define KEY_SIZE  (129)

char* generateKeyRAS(char* name);

#endif /* _EASY_ENCRYPT_H_ */
		
{% endhighlight %}

浅析：这就是标准的C语言模块了，这是逻辑的h文件，不解释。

###jni目录下local_logic_c子目录中本地C语言实现的逻辑目录下的接口逻辑实现文件easy_encrypt.c内容：

{% highlight ruby %}

#include <string.h>
#include "easy_encrypt.h"
#include "./../utils/android_log_print.h"

/*
 * 功能：通过传入name生成加密后唯一的key值
 *
 * name 传入小于KEY_NAME_SIZE的字符串
 * return 通过name生成的验证key值
 */
char* generateKeyRAS(char* name)
{
    //判断形参是否有效
    if (NULL == name || strlen(name) > KEY_NAME_SIZE) {
		LOGD("function generateKey must have a ok name!\n");
        return NULL;
    }

    //声明局部变量
    int index = 0;
    int loop = 0;
    char temp[KEY_SIZE] = {"\0"};
    //清空数组内存
    memset(temp, 0, sizeof(temp));
    //将传进来的name拷贝到零时空间
    strcpy(temp, name);
    //进行通过name转化生成key的逻辑，这里是模拟测试，实际算法比这复杂
    for (index=0; index<KEY_SIZE-1; index++)
    {
        temp[index] = 93;
        LOGD("---------------temp[%d]=%c", index, temp[index]);
    }

    return temp;
}

{% endhighlight %}

浅析：这就是标准的C语言模块了，这是逻辑的c文件，模拟实现了加密算法而已。

###build.gradle文件中android.defaultConfig中新加如下代码（其他使用AS编译设置参见本系列教程一）：

{% highlight ruby %}

ndk{
	moduleName "YanboberJniLibName"
	ldLibs "log", "z", "m"	//添加依赖库文件，因为有log打印等
	abiFilters "armeabi", "armeabi-v7a", "x86"
}

{% endhighlight %}

浅析：不解释。

###编译代码运行在LogCat中可以看见主要的几条Log如下：

<img src="http://yanbober.github.io/image/2015-2-14-android_studio_jni_1/5.png" />

浅析：这里你会看到在运行app时：
	
- 尝试加载so文件	Trying to load lib /data/app-lib/io.github.yanbober.ndkapplication-2/libYanboberJniLibName.so 0xa6a4e120
- 加载了so文件	Added shared lib /data/app-lib/io.github.yanbober.ndkapplication-2/libYanboberJniLibName.so 0xa6a4e120
- 先不解释这句话	No JNI_OnLoad found in /data/app-lib/io.github.yanbober.ndkapplication-2/libYanboberJniLibName.so 0xa6a4e120, skipping init

上面说“先不解释这句话”的No JNI_OnLoad found......skipping init其实透露出了一个新的知识点，下文会介绍的。

###运行程序结果如下：

<img src="http://yanbober.github.io/image/2015-2-14-android_studio_jni_1/6.png" />

浅析：传入name加密后得到的key显示。

<hr>

##总结

以上第一部分就是JNI开发常见的基本结构模板，实际开发代码量和文件和目录结构都会比这复杂，这只是一个雏形用来领悟重点。

#**第二部分**

<hr>

##概述

如果你已经大致理解掌握了第一部分内容，那基本OK了。接下来要扯蛋的就是第一部分遗留的历史问题和其他提升技能。

首先，不知道还记不记得第一部分编译代码运行在LogCat中可以看见主要的几条Log。“No JNI_OnLoad found......skipping init”这句话是不是还是依旧耿耿于怀呢？
那么接下来咱们放大招来kill它。

<hr>

##从Load这个蛋疼的词说起

###Android OS加载JNI Lib的方法有两种：

- 通过JNI_OnLoad。
- 如果JNI Lib实现中没有定义JNI_OnLoad，则dvm调用dvmResolveNativeMethod进行动态解析。

PS：咱们上面第一部分就是dvm调用dvmResolveNativeMethod进行动态解析，所以log打印No JNI_OnLoad found。

###从网上查到的深入解析（此解析模块代码引用自网络）

####JNI_OnLoad机制分析

System.loadLibrary调用流程如下所示：

System.loadLibrary->Runtime.loadLibrary->(Java)nativeLoad->(C: java_lang_Runtime.cpp)Dalvik_java_lang_Runtime_nativeLoad->dvmLoadNativeCode->(dalvik/vm/Native.cpp)

接着如下：

- dlopen(pathName, RTLD_LAZY) (把.so mmap到进程空间，并把func等相关信息填充到soinfo中)
- dlsym(handle, "JNI_OnLoad")
- JNI_OnLoad->RegisterNatives->dvmRegisterJNIMethod(ClassObject* clazz, const char* methodName, const char* signature, void* fnPtr)->dvmUseJNIBridge(method, fnPtr)->(method->nativeFunc = func)

JNI函数在进程空间中的起始地址被保存在ClassObject->directMethods中。

{% highlight ruby %}

struct ClassObject : Object {  
    /* static, private, and <init> methods */  
    int             directMethodCount;  
    Method*         directMethods;  
  
    /* virtual methods defined in this class; invoked through vtable */  
    int             virtualMethodCount;  
    Method*         virtualMethods;  
}

{% endhighlight %}
    
此ClassObject通过gDvm.jniGlobalRefTable或gDvm.jniWeakGlobalRefLock获取。



####dvmResolveNativeMethod延迟解析机制

如果JNI Lib中没有JNI_OnLoad，即在执行System.loadLibrary时，无法把此JNI Lib实现的函数在进程中的地址增加到ClassObject->directMethods。则直到需要调用的时候才会解析这些javah风格的函数 。这样的函数dvmResolveNativeMethod(dalvik/vm/Native.cpp)来进行解析，其执行流程如下所示：

void dvmResolveNativeMethod(const u4* args, JValue* pResult, const Method* method, Thread* self)->(Resolve a native method and invoke it.)

接着如下：

- void* func = lookupSharedLibMethod(method)(根据signature在所有已经打开的.so中寻找此函数实现)dvmHashForeach(gDvm.nativeLibs, findMethodInLib,(void\*) method)->findMethodInLib(void\* vlib, void\* vmethod)->dlsym(pLib->handle, mangleCM)
- dvmUseJNIBridge((Method*) method, func)
- (*method->nativeFunc)(args, pResult, method, self);(调用执行)

##说完蛋疼Load基础后该准么办？

答案其实就是推荐Android OS加载JNI Lib的方法的通过JNI_OnLoad。因为通过它你可以干许多自定义的事，譬如实现自己的本地注册等。
因为在上面的解析中已经看到了JNI_OnLoad->RegisterNatives->...这两个关键方法。具体细节咱们现在再说说。

###先来看JNI_OnLoad函数

JNI_OnLoad()函数主要的用途有两点：
 
- 通知VM此C组件使用的JNI版本。如果你的.so文件没有提供JNI_OnLoad()函数，VM会默认该.so使用最老的JNI 1.1版本。
而新版的JNI做了许多扩充，如果需要使用JNI的新版功能，例如JNI 1.4的java.nio.ByteBuffer, 就必须藉由JNI_OnLoad()函数来告知VM。 
- 因为VM执行到System.loadLibrary()函数时，会立即先调运JNI_OnLoad()，所以C组件的开发者可以由JNI_OnLoad()来进行C组件内的初期值之设定(Initialization)。

既然有JNI_OnLoad()，那就有相呼应的函数，那就是JNI_OnUnload()，当VM释放JNI组件时会呼叫它，因此在该方法中进行善后清理，资源释放的动作最为合适。

###再来看RegisterNatives函数

在上面第一部分时我们看见通过javah命令生成的io_github_yanbober_ndkapplication_NdkJniUtils.h里函数的名字好长，看着就蛋疼。你肯定也想过怎么这么长，
而且当有时候项目需求原因导致类名变了的时候，函数名必须一个一个的改，更加蛋疼。我第一次接触时那时候自己经验不足，就遇上了这个蛋疼问题。泪奔啊！

既然这样那就有解决办法的，那就是RegisterNatives大招。接下来来看下这个大招：

App的Java程序寻找c本地方法的过程一般是依赖VM去寻找*.so里的本地函数，如果需要连续调运很多次，每次都要寻找一遍，
会多花许多时间。因此为了解决这个问题我们可以自行将本地函数向VM进行登记，然后让VM自行调registerNativeMethods()函数。

VM自行调registerNativeMethods()函数的作用主要有两点：　　

- 更加有效率去找到C语言的函数　　
- 可以在执行期间进行抽换，因为自定义的JNINativeMethod类型的methods[]数组是一个名称-函数指针对照表，在程序执行时，
可以多次调运registerNativeMethods()函数来更换本地函数指针，从而达到弹性抽换本地函数的效果。

上面提到的JNINativeMethod结构是c/c++方法和Java方法之间映射关系的关键结构，该结构定义在jni.h中，具体定义如下：

{% highlight ruby %}
typedef struct {   
	const char* name;//java方法名称   
	const char* signature; //java方法签名  
	void*       fnPtr;//c/c++的函数指针  
} JNINativeMethod; 
{% endhighlight %}

所谓自定义的JNINativeMethod类型的methods[]数组自然也就类似长下面这样了：

{% highlight ruby %}
static JNINativeMethod methods[] = {  
		{"generateKey", "(Ljava/lang/String;)Ljava/lang/String;", (void*)generateKey},  
}; 
{% endhighlight %}

以上也就是所谓的动态注册JNI了。

好了，该补脑的也差不多了，很空洞很枯燥，空虚寂寞冷啊；接下来进入实战吧，通过对第一部分代码的改变来轻松理解这部分扯淡的内容。

<hr>

##代码实例分析

我们对第一部分的jni根目录下的c代码修改如下：

{% highlight ruby %}

#include <jni.h>
#include <string.h>
#include <assert.h>
#include "io_github_yanbober_ndkapplication_NdkJniUtils.h"
#include "./utils/android_log_print.h"
#include "./local_logic_c/easy_encrypt.h"

JNIEXPORT jstring JNICALL native_generate_key(JNIEnv *env, jobject obj, jstring name)
{
     //声明局部量
     char key[KEY_SIZE] = {0};
     memset(key, 0, sizeof(key));

     char temp[KEY_NAME_SIZE] = {0};

     //将java传入的name转换为本地utf的char*
     const char* pName = (*env)->GetStringUTFChars(env, name, NULL);

     if (NULL != pName)
     {
        strcpy(temp, pName);
        strcpy(key, generateKeyRAS(temp));

        //java的name对象不需要再使用，通知虚拟机回收name
        (*env)->ReleaseStringUTFChars(env, name, pName);
     }

     return (*env)->NewStringUTF(env, key);
  }

//参数映射表
static JNINativeMethod methods[] = {
    {"nativeGenerateKey", "(Ljava/lang/String;)Ljava/lang/String;", (void*)native_generate_key},
    //这里可以有很多其他映射函数
};

//自定义函数，为某一个类注册本地方法，调运JNI注册方法
static int registerNativeMethods(JNIEnv* env , const char* className , JNINativeMethod* gMethods, int numMethods)
{
    jclass clazz;
    clazz = (*env)->FindClass(env, className);
    if (clazz == NULL)
    {
        return JNI_FALSE;
    }
    //JNI函数，参见系列教程2
    if ((*env)->RegisterNatives(env, clazz, gMethods, numMethods) < 0)
    {
        return JNI_FALSE;
    }

    return JNI_TRUE;
}

//自定义函数
static int registerNatives(JNIEnv* env)
{
    const char* kClassName = "io/github/yanbober/ndkapplication/NdkJniUtils";//指定要注册的类
    return registerNativeMethods(env, kClassName, methods,  sizeof(methods) / sizeof(methods[0]));
}

JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void* reserved)
{
    LOGD("customer---------------------------JNI_OnLoad-----into.\n");
    JNIEnv* env = NULL;
    jint result = -1;

    if ((*vm)->GetEnv(vm, (void**) &env, JNI_VERSION_1_4) != JNI_OK)
    {
        return -1;
    }
    assert(env != NULL);

    //动态注册，自定义函数
    if (!registerNatives(env))
    {
        return -1;
    }

    return JNI_VERSION_1_4;
}

{% endhighlight %}

相应的h头文件修改如下：

{% highlight ruby %}

/* DO NOT EDIT THIS FILE - it is machine generated */
#include <jni.h>

#ifndef _Included_io_github_yanbober_ndkapplication_NdkJniUtils
#define _Included_io_github_yanbober_ndkapplication_NdkJniUtils
#ifdef __cplusplus
extern "C" {
#endif

JNIEXPORT jstring JNICALL native_generate_key(JNIEnv *env, jobject obj, jstring name);

#ifdef __cplusplus
}
#endif
#endif

{% endhighlight %}

对应的java文件中native方法名字换为映射表中的nativeGenerateKey即可。

以上代码不做详细解释，代码中有注释，同时可以参考该系列第二篇博客字典。

##总结

至此一个比Hello World稍微复杂一丁点儿的例子就分析的差不多了。整个JNI的基本雏形也就差不多这样子。下一篇会从其他角度来啃。T_T!!!

	（烦请令尊体谅作者劳动成果，转载麻烦声明文章链接。您的声明与讨论是鄙人写作的动力。JNI NDK系列文章依据时间及个人情况持续更新中......）