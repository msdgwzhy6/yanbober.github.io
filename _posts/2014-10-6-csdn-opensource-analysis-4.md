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

<hr>

##**硬着头皮开始吧**

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
    } catch (PackageManager.NameNotFoundException e) {
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
        NetworkDispatcher networkDispatcher = new NetworkDispatcher(mNetworkQueue, mNetwork, mCache, mDelivery);
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

首先通过Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);设置线程优先级，然后通过mCache.initialize();
初始化缓存块，其中mCache是由Volley.java中的newRequestQueue(Context context, HttpStack stack)方法中实例化传入的，其
Cache接口的实现为new DiskBasedCache(cacheDir)，其中cacheDir默认在Volley.java中不设置为/data/data/app-package/cache/volley/。
接下来由while (true)可以发现缓存线程是一直在执行，其中通过mQuit标记进制是否结束线程的操作。
mCacheQueue.take()从阻塞队列获取最前面的一个request，没有request就阻塞等待。
接着通过mCache.get(request.getCacheKey());尝试从缓存中取出响应结果，
如何为空的话则把这条请求加入到网络请求队列中，如果不为空的话再判断该缓存是否已过期，
如果已经过期了则同样把这条请求加入到网络请求队列中，否则就认为不需要重发网络请求，
直接使用缓存中的数据即可。
在这个过程中调运了parseNetworkResponse()方法来对数据进行解析，再往后就是将解析出来的数据进行回调了。现在先来看下
Request抽象基类的这部分代码：

{% highlight ruby %}
/**
 * Subclasses must implement this to parse the raw network response
 * and return an appropriate response type. This method will be
 * called from a worker thread.  The response will not be delivered
 * if you return null.
 * @param response Response from the network
 * @return The parsed response, or null in the case of an error
 */
abstract protected Response<T> parseNetworkResponse(NetworkResponse response);
{% endhighlight %}

通过注释可以看到他就是一个解析模块的功能。

上面说了，当调用了Volley.newRequestQueue(context)之后，就会有五个线程一直在后台运行，不断等待网络请求的到来，
其中一个CacheDispatcher是缓存线程，四个NetworkDispatcher是网络请求线程。CacheDispatcher的run方法刚才已经大致分析了，解析来看下
NetworkDispatcher中是怎么处理网络请求队列的，具体代码如下所示：

{% highlight ruby %}
@Override
public void run() {
	Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
	while (true) {
		long startTimeMs = SystemClock.elapsedRealtime();
		Request<?> request;
		try {
			// Take a request from the queue.
			request = mQueue.take();
		} catch (InterruptedException e) {
			// We may have been interrupted because it was time to quit.
			if (mQuit) {
				return;
			}
			continue;
		}

		try {
			request.addMarker("network-queue-take");

			// If the request was cancelled already, do not perform the
			// network request.
			if (request.isCanceled()) {
				request.finish("network-discard-cancelled");
				continue;
			}

			addTrafficStatsTag(request);

			// Perform the network request.
			NetworkResponse networkResponse = mNetwork.performRequest(request);
			request.addMarker("network-http-complete");

			// If the server returned 304 AND we delivered a response already,
			// we're done -- don't deliver a second identical response.
			if (networkResponse.notModified && request.hasHadResponseDelivered()) {
				request.finish("not-modified");
				continue;
			}

			// Parse the response here on the worker thread.
			Response<?> response = request.parseNetworkResponse(networkResponse);
			request.addMarker("network-parse-complete");

			// Write to cache if applicable.
			// TODO: Only update cache metadata instead of entire record for 304s.
			if (request.shouldCache() && response.cacheEntry != null) {
				mCache.put(request.getCacheKey(), response.cacheEntry);
				request.addMarker("network-cache-written");
			}

			// Post the response back.
			request.markDelivered();
			mDelivery.postResponse(request, response);
		} catch (VolleyError volleyError) {
			volleyError.setNetworkTimeMs(SystemClock.elapsedRealtime() - startTimeMs);
			parseAndDeliverNetworkError(request, volleyError);
		} catch (Exception e) {
			VolleyLog.e(e, "Unhandled exception %s", e.toString());
			VolleyError volleyError = new VolleyError(e);
			volleyError.setNetworkTimeMs(SystemClock.elapsedRealtime() - startTimeMs);
			mDelivery.postError(request, volleyError);
		}
	}
}
{% endhighlight %}

和CacheDispatcher差不多，如上可以看见一个类似的while(true)循环，说明网络请求线程也是在不断运行的。
如上通过mNetwork.performRequest(request);代码来发送网络请求，而Network是一个接口，这里具体的实现之前已经分析是BasicNetwork，
所以先看下它的performRequest()方法，如下所示：

NetWork接口的代码：

{% highlight ruby %}
public interface Network {
	/**
	* Performs the specified request.
	* @param request Request to process
	* @return A {@link NetworkResponse} with data and caching metadata; will never be null
	* @throws VolleyError on errors
	*/
	public NetworkResponse performRequest(Request<?> request) throws VolleyError;
}
{% endhighlight %}

上面说了，就是执行指定的请求。他的BasicNetwork实现子类如下：

{% highlight ruby %}
@Override
public NetworkResponse performRequest(Request<?> request) throws VolleyError {
	long requestStart = SystemClock.elapsedRealtime();
	while (true) {
		HttpResponse httpResponse = null;
		byte[] responseContents = null;
		Map<String, String> responseHeaders = Collections.emptyMap();
		try {
			// Gather headers.
			Map<String, String> headers = new HashMap<String, String>();
			addCacheHeaders(headers, request.getCacheEntry());
			httpResponse = mHttpStack.performRequest(request, headers);
			StatusLine statusLine = httpResponse.getStatusLine();
			int statusCode = statusLine.getStatusCode();

			responseHeaders = convertHeaders(httpResponse.getAllHeaders());
			// Handle cache validation.
			if (statusCode == HttpStatus.SC_NOT_MODIFIED) {

				Entry entry = request.getCacheEntry();
				if (entry == null) {
					return new NetworkResponse(HttpStatus.SC_NOT_MODIFIED, null,
							responseHeaders, true,
							SystemClock.elapsedRealtime() - requestStart);
				}

				// A HTTP 304 response does not have all header fields. We
				// have to use the header fields from the cache entry plus
				// the new ones from the response.
				// http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.3.5
				entry.responseHeaders.putAll(responseHeaders);
				return new NetworkResponse(HttpStatus.SC_NOT_MODIFIED, entry.data,
						entry.responseHeaders, true,
						SystemClock.elapsedRealtime() - requestStart);
			}

			// Some responses such as 204s do not have content.  We must check.
			if (httpResponse.getEntity() != null) {
				responseContents = entityToBytes(httpResponse.getEntity());
			} else {
				// Add 0 byte response as a way of honestly representing a
				// no-content request.
				responseContents = new byte[0];
			}

			// if the request is slow, log it.
			long requestLifetime = SystemClock.elapsedRealtime() - requestStart;
			logSlowRequests(requestLifetime, request, responseContents, statusLine);

			if (statusCode < 200 || statusCode > 299) {
				throw new IOException();
			}
			return new NetworkResponse(statusCode, responseContents, responseHeaders, false,
				SystemClock.elapsedRealtime() - requestStart);
		} catch (SocketTimeoutException e) {
			attemptRetryOnException("socket", request, new TimeoutError());
		} catch (ConnectTimeoutException e) {
			attemptRetryOnException("connection", request, new TimeoutError());
		} catch (MalformedURLException e) {
			throw new RuntimeException("Bad URL " + request.getUrl(), e);
		} catch (IOException e) {
			int statusCode = 0;
			NetworkResponse networkResponse = null;
			if (httpResponse != null) {
				statusCode = httpResponse.getStatusLine().getStatusCode();
			} else {
				throw new NoConnectionError(e);
			}
			VolleyLog.e("Unexpected response code %d for %s", statusCode, request.getUrl());
			if (responseContents != null) {
				networkResponse = new NetworkResponse(statusCode, responseContents,
						responseHeaders, false, SystemClock.elapsedRealtime() - requestStart);
				if (statusCode == HttpStatus.SC_UNAUTHORIZED ||
						statusCode == HttpStatus.SC_FORBIDDEN) {
					attemptRetryOnException("auth",
							request, new AuthFailureError(networkResponse));
				} else {
					// TODO: Only throw ServerError for 5xx status codes.
					throw new ServerError(networkResponse);
				}
			} else {
				throw new NetworkError(networkResponse);
			}
		}
	}
}
{% endhighlight %}

这个方法是网络请求的具体实现，也是一个大while循环，其中mHttpStack.performRequest(request, headers);代码中的mHttpStack是Volley的newRequestQueue()方法中创建的实例，
前面已经说过，这两个对象的内部实际就是分别使用HttpURLConnection和HttpClient来发送网络请求的，然后把服务器返回的数据组装成一个NetworkResponse对象进行返回。
在NetworkDispatcher中收到了NetworkResponse这个返回值后又会调用Request的parseNetworkResponse()方法来解析NetworkResponse中的数据，
同时将数据写入到缓存，这个方法的实现是交给Request的子类来完成的，因为不同种类的Request解析的方式也肯定不同。

钱买你可以看到在NetWorkDispatcher的run中最后执行了mDelivery.postResponse(request, response);，也就是说在解析完了NetworkResponse中的数据之后，
又会调用ExecutorDelivery（ResponseDelivery接口的实现类）的postResponse()方法来回调解析出的数据，具体代码如下所示：

{% highlight ruby %}
@Override
public void postResponse(Request<?> request, Response<?> response, Runnable runnable) {
	request.markDelivered();
	request.addMarker("post-response");
	mResponsePoster.execute(new ResponseDeliveryRunnable(request, response, runnable));
}
{% endhighlight %}

这里可以看见在mResponsePoster的execute()方法中传入了一个ResponseDeliveryRunnable对象，
就可以保证该对象中的run()方法就是在主线程当中运行的了，我们看下run()方法中的代码是什么样的：

{% highlight ruby %}
/**
 * A Runnable used for delivering network responses to a listener on the
 * main thread.
 */
@SuppressWarnings("rawtypes")
private class ResponseDeliveryRunnable implements Runnable {
	private final Request mRequest;
	private final Response mResponse;
	private final Runnable mRunnable;

	public ResponseDeliveryRunnable(Request request, Response response, Runnable runnable) {
		mRequest = request;
		mResponse = response;
		mRunnable = runnable;
	}

	@SuppressWarnings("unchecked")
	@Override
	public void run() {
		// If this request has canceled, finish it and don't deliver.
		if (mRequest.isCanceled()) {
			mRequest.finish("canceled-at-delivery");
			return;
		}

		// Deliver a normal response or error, depending.
		if (mResponse.isSuccess()) {
			mRequest.deliverResponse(mResponse.result);
		} else {
			mRequest.deliverError(mResponse.error);
		}

		// If this is an intermediate response, add a marker, otherwise we're done
		// and the request can be finished.
		if (mResponse.intermediate) {
			mRequest.addMarker("intermediate-response");
		} else {
			mRequest.finish("done");
		}

		// If we have been provided a post-delivery runnable, run it.
		if (mRunnable != null) {
			mRunnable.run();
		}
	}
}
{% endhighlight %}

这段代码里的run方法中可以看到如下一部分细节：

{% highlight ruby %}
// Deliver a normal response or error, depending.
if (mResponse.isSuccess()) {
	mRequest.deliverResponse(mResponse.result);
} else {
	mRequest.deliverError(mResponse.error);
}
{% endhighlight %}

这段代码是最核心的，明显可以看到通过mRequest的deliverResponse或者deliverError将反馈发送到回调到UI线程。这也是你重写实现的接口方法。

<hr>

##**再来整体看下**

现在是该回过头去看背景知识模块了，再看下那幅官方图，对比就明白咋回事了。结合上图和如上分析可以知道：

1. 当一个RequestQueue被成功申请后会开启一个CacheDispatcher和4个默认的NetworkDispatcher。

2. CacheDispatcher缓存调度器最为第一层缓冲，开始工作后阻塞的从缓存序列mCacheQueue中取得请求；
对于已经取消的请求，标记为跳过并结束这个请求；新的或者过期的请求，直接放入mNetworkQueue中由N个NetworkDispatcher进行处理；
已获得缓存信息（网络应答）却没有过期的请求，由Request的parseNetworkResponse进行解析，从而确定此应答是否成功。
然后将请求和应答交由Delivery分发者进行处理，如果需要更新缓存那么该请求还会被放入mNetworkQueue中。

3. 将请求Request add到RequestQueue后对于不需要缓存的请求（需要额外设置，默认是需要缓存）直接丢入mNetworkQueue交给N个NetworkDispatcher处理；
对于需要缓存的，新的请求加到mCacheQueue中给CacheDispatcher处理；需要缓存，但是缓存列表中已经存在了相同URL的请求，
放在mWaitingQueue中做暂时处理，等待之前请求完毕后，再重新添加到mCacheQueue中。

4. 网络请求调度器NetworkDispatcher作为网络请求真实发生的地方，对消息交给BasicNetwork进行处理，同样的，请求和结果都交由Delivery分发者进行处理。

5. Delivery分发者实际上已经是对网络请求处理的最后一层了，在Delivery对请求处理之前，Request已经对网络应答进行过解析，此时应答成功与否已经设定；
而后Delivery根据请求所获得的应答情况做不同处理；若应答成功，则触发deliverResponse方法，最终会触发开发者为Request设定的Listener；
若应答失败，则触发deliverError方法，最终会触发开发者为Request设定的ErrorListener；
处理完后，一个Request的生命周期就结束了，Delivery会调用Request的finish操作，将其从mRequestQueue中移除，
与此同时，如果等待列表中存在相同URL的请求，则会将剩余的层级请求全部丢入mCacheQueue交由CacheDispatcher进行处理。

至此所有搞定。

**PPPS一句：通过上面原理分析之后总结发现，推荐整个App全局持有一个RequestQueue的做法，这样会有相对比较高的性能效率。**

<hr>

[走读代码时参考博客链接](http://www.cnblogs.com/cpacm/p/4211719.html)
