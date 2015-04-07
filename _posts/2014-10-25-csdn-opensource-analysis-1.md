---
layout: post
keywords: Acache, 框架, 缓存
description: Android ACache框架分析
title: "[Training] ASimpleCache框架使用分析"
categories: [开源框架库浅析]
tags: [ACache, 缓存]
group: archive
icon: file-alt
---
{% include site/setup %}

<hr>
##**开源项目链接**

[ASimpleCache](https://github.com/yangfuhai/ASimpleCache)

<hr>

##**源码浅析**

首先看下如下结构图，思维导图展示了ACache源码类结构：

<img src="http://yanbober.github.io/image/open/ACacheStructure.png" />

如上图所示，ACache类的构造方法为private的，所以只能通过get方式获取实例。默认情况下调运ACache.get(Context);方法得到的缓存文件放置在/data/data/app-package-name/cache/路径下，
缓存的目录默认为ACache，缓存大小和数量均由ACache中的final变量控制。

其实在获取ACache实例时最终默认调运的实例方法是get(File cacheDir, long max_zise, int max_count);

{% highlight ruby %}
public static ACache get(File cacheDir, long max_zise, int max_count) {
        ACache manager = mInstanceMap.get(cacheDir.getAbsoluteFile() + myPid());
        if (manager == null) {
            manager = new ACache(cacheDir, max_zise, max_count);
            mInstanceMap.put(cacheDir.getAbsolutePath() + myPid(), manager);
        }
        return manager;
    }
{% endhighlight %}

第一次进来由于mInstanceMap中没有任何map，所以manager == null。故通过ACache构造方法构造对象，构造方法如下所示。然后将实例引用存入map。

{% highlight ruby %}
    private ACache(File cacheDir, long max_size, int max_count) {
        if (!cacheDir.exists() && !cacheDir.mkdirs()) {
            throw new RuntimeException("can't make dirs in "
                    + cacheDir.getAbsolutePath());
        }
        mCache = new ACacheManager(cacheDir, max_size, max_count);
    }
{% endhighlight %}

目录不存在或者创建失败抛出异常，否则实例化ACacheManager内部类实例。ACacheManager内部类的构造函数如下：

{% highlight ruby %}
private ACacheManager(File cacheDir, long sizeLimit, int countLimit) {
            this.cacheDir = cacheDir;
            this.sizeLimit = sizeLimit;
            this.countLimit = countLimit;
            cacheSize = new AtomicLong();
            cacheCount = new AtomicInteger();
            calculateCacheSizeAndCacheCount();
        }
{% endhighlight %}

构造函数通过calculateCacheSizeAndCacheCount();方法计算cache size和count，其中calculateCacheSizeAndCacheCount()方法如下：

{% highlight ruby %}
private void calculateCacheSizeAndCacheCount() {
	new Thread(new Runnable() {
		@Override
		public void run() {
			int size = 0;
			int count = 0;
			File[] cachedFiles = cacheDir.listFiles();
			if (cachedFiles != null) {
				for (File cachedFile : cachedFiles) {
					size += calculateSize(cachedFile);
					count += 1;
					lastUsageDates.put(cachedFile,
							cachedFile.lastModified());
				}
				cacheSize.set(size);
				cacheCount.set(count);
			}
		}
	}).start();
}
{% endhighlight %}

calculateCacheSizeAndCacheCount方法中通过开启线程进行cache size和count的计算，放置阻塞UI线程。计算完后存入cacheSize和cacheCount，
所以cacheSize和cacheCount在内部类中定义为AtomicLong和AtomicInteger类型，也就是线程安全的，量子操作。

至此整个获取ACache实例的过程结束。

接下来就是一堆的存取操作了。

如上结构图，你可以通过ACache的各种public的put和get等方法进行key-value形式的缓存你想要缓存的数据类型。这里选取其中一种分析：

{% highlight ruby %}
public void put(String key, String value, int saveTime) {
	put(key, Utils.newStringWithDateInfo(saveTime, value));
}
{% endhighlight %}

ACache的这个put方法可以缓存指定时间长度的key-value值。在ACache的该put方法中调运了他自身的另一个如下实现方法，其中第二个参数value传入的是
Utils.newStringWithDateInfo(saveTime, value)，而newStringWithDateInfo是ACache的内部工具类的一个方法，用来拼接time为header，value为content的
字符串，然后组成新的字串返回。

{% highlight ruby %}
public void put(String key, String value) {
        File file = mCache.newFile(key);
        BufferedWriter out = null;
        try {
            out = new BufferedWriter(new FileWriter(file), 1024);
            out.write(value);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (out != null) {
                try {
                    out.flush();
                    out.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            mCache.put(file);
        }
    }
{% endhighlight %}

在put(String key, String value)方法中首先在缓存目录下调运mCache.newFile(key)新建一个文件，新建的文件名为：File(cacheDir, key.hashCode() + "")。然后将
数据存入文件即可。

获取过程可以直接获取，也可以调运Utils内部类的一些方法来判断是否缓存过期等操作。也可以删除指定缓存或者批量操作等。

至此整个ACache的源码分析完成，其他方法类比分析即可。
