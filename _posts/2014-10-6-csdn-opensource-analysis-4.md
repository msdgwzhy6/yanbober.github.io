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

还是按照前边的顺序分析吧，使用Volley的第一步首先是通过Volley.newRequestQueue(context)得到RequestQueue队列，
那么先看下toolbox下的Volley.java中的这个方法吧。

{% highlight ruby %}
/**
 * Creates a default instance of the worker pool and calls {@link RequestQueue#start()} on it.
 *
 * @param context A {@link Context} to use for creating the cache dir.
 * @return A started {@link RequestQueue} instance.
 */
public static RequestQueue newRequestQueue(Context context) {
	return newRequestQueue(context, null);
}
{% endhighlight %}

先看如上注释所示，创建一个默认的worker pool，并且调运RequestQueue的start方法。在这个方法里又调运了该类的另一个
连个参数的重载方法，如下所示。

{% highlight ruby %}
/**
 * Creates a default instance of the worker pool and calls {@link RequestQueue#start()} on it.
 *
 * @param context A {@link Context} to use for creating the cache dir.
 * @param stack An {@link HttpStack} to use for the network, or null for default.
 * @return A started {@link RequestQueue} instance.
 */
public static RequestQueue newRequestQueue(Context context, HttpStack stack) {
	File cacheDir = new File(context.getCacheDir(), DEFAULT_CACHE_DIR);

	String userAgent = "volley/0";
	try {
		String packageName = context.getPackageName();
		PackageInfo info = context.getPackageManager().getPackageInfo(packageName, 0);
		userAgent = packageName + "/" + info.versionCode;
	} catch (NameNotFoundException e) {
	}

	if (stack == null) {
		if (Build.VERSION.SDK_INT >= 9) {
			stack = new HurlStack();
		} else {
			// Prior to Gingerbread, HttpUrlConnection was unreliable.
			// See: http://android-developers.blogspot.com/2011/09/androids-http-clients.html
			stack = new HttpClientStack(AndroidHttpClient.newInstance(userAgent));
		}
	}

	Network network = new BasicNetwork(stack);

	RequestQueue queue = new RequestQueue(new DiskBasedCache(cacheDir), network);
	queue.start();

	return queue;
}
{% endhighlight %}

如上所示，该方法有两个参数，第一个Context是为了拿到当前App的Activity的一些信息，第二个参数在默认的
RequestQueue newRequestQueue(Context context)方法中传递为null，也就是说在这段代码中的if (stack == null)会被
执行。在这个if中对版本号进行了判断，如果版本号大于等于9就使得HttpStack对象的实例为HurlStack，如果小于9则实例
为HttpClientStack。至于这里为何进行版本号判断，实际代码中的[See](See: http://android-developers.blogspot.com/2011/09/androids-http-clients.html)
注释已经说明了。实际上HurlStack类也在toolbox中，他实现了toolbox的HttpStack接口中的HttpResponse performRequest(Request<?> request, Map<String, String> additionalHeaders)
方法，其实现过程使用的是HttpURLConnection。而HttpClientStack也在toolbox中，他也实现了toolbox的HttpStack接口的HttpResponse performRequest(Request<?> request, Map<String, String> additionalHeaders)
方法，不过其实现过程使用的是HttpClient而已。

其中的userAgent就是App的包名加版本号而已，传入new HttpClientStack(AndroidHttpClient.newInstance(userAgent));作为name TAG使用。

如上HttpStack创建完成之后创建了Network实例。BasicNetwork是Network接口的实现，他们都在toolbox中，BasicNetwork实现了
public NetworkResponse performRequest(Request<?> request)方法，其作用是根据传入的HttpStack对象来处理网络请求。
紧接着new出一个RequestQueue对象，并调用它的start()方法进行启动，然后将RequestQueue返回。RequestQueue是根目录下的一个类，
其作用是一个请求调度队列调度程序的线程池。这样newRequestQueue()的方法就执行结束了。

现在再来看下根目录下RequestQueue队列的start方法，如下所示：

{% highlight ruby %}
/**
 * Starts the dispatchers in this queue.
 */
public void start() {
	stop();  // Make sure any currently running dispatchers are stopped.
	// Create the cache dispatcher and start it.
	mCacheDispatcher = new CacheDispatcher(mCacheQueue, mNetworkQueue, mCache, mDelivery);
	mCacheDispatcher.start();

	// Create network dispatchers (and corresponding threads) up to the pool size.
	for (int i = 0; i < mDispatchers.length; i++) {
		NetworkDispatcher networkDispatcher = new NetworkDispatcher(mNetworkQueue, mNetwork,
				mCache, mDelivery);
		mDispatchers[i] = networkDispatcher;
		networkDispatcher.start();
	}
}
{% endhighlight %}

通过注释可以看出来这里在派发队列的事务。先是创建了一个CacheDispatcher的实例，然后调用了它的start()方法，
接着在一个for循环里去创建NetworkDispatcher的实例，并分别调用它们的start()方法。
这里的CacheDispatcher和NetworkDispatcher都是继承自Thread的，
而默认情况下for循环会执行（DEFAULT_NETWORK_THREAD_POOL_SIZE）四次，
也就是说当调用了Volley.newRequestQueue(context)之后，就会有五个线程一直在后台运行，不断等待网络请求的到来，
其中一个CacheDispatcher是缓存线程，四个NetworkDispatcher是网络请求线程。

按照之前使用Volley可以知道，得到了RequestQueue之后，我们只需要构建出相应的Request，
然后调用RequestQueue的add()方法将Request传入就可以完成网络请求操作了。也就是说add()方法的内部是核心代码了。
现在看下RequestQueue的add方法，具体如下：

{% highlight ruby %}
/**
 * Adds a Request to the dispatch queue.
 * @param request The request to service
 * @return The passed-in request
 */
public <T> Request<T> add(Request<T> request) {
	// Tag the request as belonging to this queue and add it to the set of current requests.
	request.setRequestQueue(this);
	synchronized (mCurrentRequests) {
		mCurrentRequests.add(request);
	}

	// Process requests in the order they are added.
	request.setSequence(getSequenceNumber());
	request.addMarker("add-to-queue");

	// If the request is uncacheable, skip the cache queue and go straight to the network.
	if (!request.shouldCache()) {
		mNetworkQueue.add(request);
		return request;
	}

	// Insert request into stage if there's already a request with the same cache key in flight.
	synchronized (mWaitingRequests) {
		String cacheKey = request.getCacheKey();
		if (mWaitingRequests.containsKey(cacheKey)) {
			// There is already a request in flight. Queue up.
			Queue<Request<?>> stagedRequests = mWaitingRequests.get(cacheKey);
			if (stagedRequests == null) {
				stagedRequests = new LinkedList<Request<?>>();
			}
			stagedRequests.add(request);
			mWaitingRequests.put(cacheKey, stagedRequests);
			if (VolleyLog.DEBUG) {
				VolleyLog.v("Request for cacheKey=%s is in flight, putting on hold.", cacheKey);
			}
		} else {
			// Insert 'null' queue for this cacheKey, indicating there is now a request in
			// flight.
			mWaitingRequests.put(cacheKey, null);
			mCacheQueue.add(request);
		}
		return request;
	}
}
{% endhighlight %}

可以看到注释所示，添加一个Request到派发队列。Request<T>是所有请求的基类，是一个抽象类。
request.setRequestQueue(this);的作用就是将请求Request关联到当前RequestQueue。
然后同步操作将当前Request添加到RequestQueue对象的mCurrentRequests HashSet中做记录。
通过request.setSequence(getSequenceNumber());得到当前RequestQueue中请求的个数，然后关联到当前Request。
request.addMarker("add-to-queue");添加调试的Debug标记。
if (!request.shouldCache())判断当前的请求是否可以缓存，如果不能缓存则直接通过mNetworkQueue.add(request);
将这条请求加入网络请求队列，然后返回request；如果可以缓存的话则在通过同步操作将这条请求加入缓存队列。
在默认情况下，每条请求都是可以缓存的，当然我们也可以调用Request的setShouldCache(false)方法来改变这一默认行为。
OK，那么既然默认每条请求都是可以缓存的（shouldCache返回为true），自然就被添加到了缓存队列中，
于是一直在后台等待的缓存线程就要开始运行起来了。现在来看下CacheDispatcher中的run()方法，代码如下所示：

{% highlight ruby %}
@Override
public void run() {
	if (DEBUG) VolleyLog.v("start new dispatcher");
	Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);

	// Make a blocking call to initialize the cache.
	mCache.initialize();

	while (true) {
		try {
			// Get a request from the cache triage queue, blocking until
			// at least one is available.
			final Request<?> request = mCacheQueue.take();
			request.addMarker("cache-queue-take");

			// If the request has been canceled, don't bother dispatching it.
			if (request.isCanceled()) {
				request.finish("cache-discard-canceled");
				continue;
			}

			// Attempt to retrieve this item from cache.
			Cache.Entry entry = mCache.get(request.getCacheKey());
			if (entry == null) {
				request.addMarker("cache-miss");
				// Cache miss; send off to the network dispatcher.
				mNetworkQueue.put(request);
				continue;
			}

			// If it is completely expired, just send it to the network.
			if (entry.isExpired()) {
				request.addMarker("cache-hit-expired");
				request.setCacheEntry(entry);
				mNetworkQueue.put(request);
				continue;
			}

			// We have a cache hit; parse its data for delivery back to the request.
			request.addMarker("cache-hit");
			Response<?> response = request.parseNetworkResponse(
					new NetworkResponse(entry.data, entry.responseHeaders));
			request.addMarker("cache-hit-parsed");

			if (!entry.refreshNeeded()) {
				// Completely unexpired cache hit. Just deliver the response.
				mDelivery.postResponse(request, response);
			} else {
				// Soft-expired cache hit. We can deliver the cached response,
				// but we need to also send the request to the network for
				// refreshing.
				request.addMarker("cache-hit-refresh-needed");
				request.setCacheEntry(entry);

				// Mark the response as intermediate.
				response.intermediate = true;

				// Post the intermediate response back to the user and have
				// the delivery then forward the request along to the network.
				mDelivery.postResponse(request, response, new Runnable() {
					@Override
					public void run() {
						try {
							mNetworkQueue.put(request);
						} catch (InterruptedException e) {
							// Not much we can do about this.
						}
					}
				});
			}

		} catch (InterruptedException e) {
			// We may have been interrupted because it was time to quit.
			if (mQuit) {
				return;
			}
			continue;
		}
	}
}
{% endhighlight %}

{% highlight ruby %}

{% endhighlight %}

{% highlight ruby %}

{% endhighlight %}