---
layout: post
keywords: async-http, 框架, android-async-http
description: android-async-http框架
title: "android-async-http框架库源码走读"
categories: [开源框架库笔记]
tags: [android-async-http, Android网络请求]
group: archive
icon: file-alt
---
{% include site/setup %}

<hr>
##**开源项目链接**

android-async-http仓库：git clone https://github.com/loopj/android-async-http

android-async-http主页：http://loopj.com/android-async-http/

<hr>

##**开始走读分析**

依据前一篇的基础使用教程可以发现，首先得到的是AsyncHttpClient实例，所以从这里入手分析一下：

{% highlight ruby %}
    /**
     * Creates a new AsyncHttpClient with default constructor arguments values
     */
    public AsyncHttpClient() {
        this(false, 80, 443);
    }
{% endhighlight %}

AsyncHttpClient类的这个默认构造函数最终调运了如下AsyncHttpClient(boolean fixNoHttpResponseException, int httpPort, int httpsPort)构造函数，
对于默认值设置了HTTP协议的默认端口为80，HTTPS协议的默认端口为443。同时发现可以通过其他构造函数来实例化AsyncHttpClient对象。

{% highlight ruby %}
    /**
     * Creates new AsyncHttpClient using given params
     *
     * @param fixNoHttpResponseException Whether to fix issue or not, by omitting SSL verification
     * @param httpPort                   HTTP port to be used, must be greater than 0
     * @param httpsPort                  HTTPS port to be used, must be greater than 0
     */
    public AsyncHttpClient(boolean fixNoHttpResponseException, int httpPort, int httpsPort) {
        this(getDefaultSchemeRegistry(fixNoHttpResponseException, httpPort, httpsPort));
    }
{% endhighlight %}

在该函数中调用了AsyncHttpClient(SchemeRegistry schemeRegistry)构造函数，而真正的实例化获取
逻辑过程就在AsyncHttpClient(SchemeRegistry schemeRegistry)方法中，如下所示：

{% highlight ruby %}
    /**
     * Creates a new AsyncHttpClient.
     *
     * @param schemeRegistry SchemeRegistry to be used
     */
    public AsyncHttpClient(SchemeRegistry schemeRegistry) {

        BasicHttpParams httpParams = new BasicHttpParams();

        ConnManagerParams.setTimeout(httpParams, connectTimeout);
        ConnManagerParams.setMaxConnectionsPerRoute(httpParams, new ConnPerRouteBean(maxConnections));
        ConnManagerParams.setMaxTotalConnections(httpParams, DEFAULT_MAX_CONNECTIONS);

        HttpConnectionParams.setSoTimeout(httpParams, responseTimeout);
        HttpConnectionParams.setConnectionTimeout(httpParams, connectTimeout);
        HttpConnectionParams.setTcpNoDelay(httpParams, true);
        HttpConnectionParams.setSocketBufferSize(httpParams, DEFAULT_SOCKET_BUFFER_SIZE);

        HttpProtocolParams.setVersion(httpParams, HttpVersion.HTTP_1_1);

        ClientConnectionManager cm = createConnectionManager(schemeRegistry, httpParams);
        Utils.asserts(cm != null, "Custom implementation of #createConnectionManager(SchemeRegistry, BasicHttpParams) returned null");

        threadPool = getDefaultThreadPool();
        requestMap = Collections.synchronizedMap(new WeakHashMap<Context, List<RequestHandle>>());
        clientHeaderMap = new HashMap<String, String>();

        httpContext = new SyncBasicHttpContext(new BasicHttpContext());
        httpClient = new DefaultHttpClient(cm, httpParams);
        httpClient.addRequestInterceptor(new HttpRequestInterceptor() {
            @Override
            public void process(HttpRequest request, HttpContext context) {
                if (!request.containsHeader(HEADER_ACCEPT_ENCODING)) {
                    request.addHeader(HEADER_ACCEPT_ENCODING, ENCODING_GZIP);
                }
                for (String header : clientHeaderMap.keySet()) {
                    if (request.containsHeader(header)) {
                        Header overwritten = request.getFirstHeader(header);
                        Log.d(LOG_TAG,
                                String.format("Headers were overwritten! (%s | %s) overwrites (%s | %s)",
                                        header, clientHeaderMap.get(header),
                                        overwritten.getName(), overwritten.getValue())
                        );

                        //remove the overwritten header
                        request.removeHeader(overwritten);
                    }
                    request.addHeader(header, clientHeaderMap.get(header));
                }
            }
        });

        httpClient.addResponseInterceptor(new HttpResponseInterceptor() {
            @Override
            public void process(HttpResponse response, HttpContext context) {
                final HttpEntity entity = response.getEntity();
                if (entity == null) {
                    return;
                }
                final Header encoding = entity.getContentEncoding();
                if (encoding != null) {
                    for (HeaderElement element : encoding.getElements()) {
                        if (element.getName().equalsIgnoreCase(ENCODING_GZIP)) {
                            response.setEntity(new InflatingEntity(entity));
                            break;
                        }
                    }
                }
            }
        });

        httpClient.addRequestInterceptor(new HttpRequestInterceptor() {
            @Override
            public void process(final HttpRequest request, final HttpContext context) throws HttpException, IOException {
                AuthState authState = (AuthState) context.getAttribute(ClientContext.TARGET_AUTH_STATE);
                CredentialsProvider credsProvider = (CredentialsProvider) context.getAttribute(
                        ClientContext.CREDS_PROVIDER);
                HttpHost targetHost = (HttpHost) context.getAttribute(ExecutionContext.HTTP_TARGET_HOST);

                if (authState.getAuthScheme() == null) {
                    AuthScope authScope = new AuthScope(targetHost.getHostName(), targetHost.getPort());
                    Credentials creds = credsProvider.getCredentials(authScope);
                    if (creds != null) {
                        authState.setAuthScheme(new BasicScheme());
                        authState.setCredentials(creds);
                    }
                }
            }
        }, 0);

        httpClient.setHttpRequestRetryHandler(new RetryHandler(DEFAULT_MAX_RETRIES, DEFAULT_RETRY_SLEEP_TIME_MILLIS));
    }
{% endhighlight %}

首先通过ConnManagerParams和HttpConnectionParams及HttpProtocolParams设置一些基本参数，譬如版本，timeout时间，max connect count等。
接着通过createConnectionManager(schemeRegistry, httpParams);方法创建了一个ClientConnectionManager，其实现类
ThreadSafeClientConnManager是一个复杂的实现来管理客户端连接池，它也可以从多个执行线程中服务连接请求。
对每个基本的路由，连接都是池管理的。
接着通过threadPool = getDefaultThreadPool();初始化网络请求的线程池。
接着初始化requestMap，用来与Android Context对应的请求map。初始化clientHeaderMap，用来与放置客户端的请求header map。
接着也是一对初始化，完事通过httpClient.setHttpRequestRetryHandler(new RetryHandler(DEFAULT_MAX_RETRIES, DEFAULT_RETRY_SLEEP_TIME_MILLIS));
设置重试Handler，会在合适的情况下自动重试。

接下来我们调运的就是AsyncHttpClient里面的各种get、post、delete等方法，通过看代码可以发现它们最终调用的都是sendRequest方法，如下：

{% highlight ruby %}
    /**
     * Puts a new request in queue as a new thread in pool to be executed
     *
     * @param client          HttpClient to be used for request, can differ in single requests
     * @param contentType     MIME body type, for POST and PUT requests, may be null
     * @param context         Context of Android application, to hold the reference of request
     * @param httpContext     HttpContext in which the request will be executed
     * @param responseHandler ResponseHandler or its subclass to put the response into
     * @param uriRequest      instance of HttpUriRequest, which means it must be of HttpDelete,
     *                        HttpPost, HttpGet, HttpPut, etc.
     * @return RequestHandle of future request process
     */
    protected RequestHandle sendRequest(DefaultHttpClient client, HttpContext httpContext, HttpUriRequest uriRequest, String contentType, ResponseHandlerInterface responseHandler, Context context) {
        if (uriRequest == null) {
            throw new IllegalArgumentException("HttpUriRequest must not be null");
        }

        if (responseHandler == null) {
            throw new IllegalArgumentException("ResponseHandler must not be null");
        }

        if (responseHandler.getUseSynchronousMode() && !responseHandler.getUsePoolThread()) {
            throw new IllegalArgumentException("Synchronous ResponseHandler used in AsyncHttpClient. You should create your response handler in a looper thread or use SyncHttpClient instead.");
        }

        if (contentType != null) {
            if (uriRequest instanceof HttpEntityEnclosingRequestBase && ((HttpEntityEnclosingRequestBase) uriRequest).getEntity() != null) {
                Log.w(LOG_TAG, "Passed contentType will be ignored because HttpEntity sets content type");
            } else {
                uriRequest.setHeader(HEADER_CONTENT_TYPE, contentType);
            }
        }

        responseHandler.setRequestHeaders(uriRequest.getAllHeaders());
        responseHandler.setRequestURI(uriRequest.getURI());

        AsyncHttpRequest request = newAsyncHttpRequest(client, httpContext, uriRequest, contentType, responseHandler, context);
        threadPool.submit(request);
        RequestHandle requestHandle = new RequestHandle(request);

        if (context != null) {
            // Add request to request map
            List<RequestHandle> requestList = requestMap.get(context);
            synchronized (requestMap) {
                if (requestList == null) {
                    requestList = Collections.synchronizedList(new LinkedList<RequestHandle>());
                    requestMap.put(context, requestList);
                }
            }

            requestList.add(requestHandle);

            Iterator<RequestHandle> iterator = requestList.iterator();
            while (iterator.hasNext()) {
                if (iterator.next().shouldBeGarbageCollected()) {
                    iterator.remove();
                }
            }
        }

        return requestHandle;
    }
{% endhighlight %}

这个方法的主要作用是将一个新的请求添加到队列线程池中执行。
AsyncHttpRequest request = newAsyncHttpRequest(client, httpContext, uriRequest, contentType, responseHandler, context);这行开始是主要的逻辑，
其创建了请求，接着通过threadPool.submit(request);把请求提交到线程池，接着通过RequestHandle requestHandle = new RequestHandle(request);把请求
包装到RequestHandle用于之后的取消、管理等操作。

现在来看，发送请求的过程其实重点是创建请求，然后submit到线程池，剩下的事情就交给线程池自己处理了，我们只需要坐等被调用。

现在来看下newAsyncHttpRequest这个逻辑实现：

{% highlight ruby %}
    /**
     * Instantiate a new asynchronous HTTP request for the passed parameters.
     *
     * @param client          HttpClient to be used for request, can differ in single requests
     * @param contentType     MIME body type, for POST and PUT requests, may be null
     * @param context         Context of Android application, to hold the reference of request
     * @param httpContext     HttpContext in which the request will be executed
     * @param responseHandler ResponseHandler or its subclass to put the response into
     * @param uriRequest      instance of HttpUriRequest, which means it must be of HttpDelete,
     *                        HttpPost, HttpGet, HttpPut, etc.
     * @return AsyncHttpRequest ready to be dispatched
     */
    protected AsyncHttpRequest newAsyncHttpRequest(DefaultHttpClient client, HttpContext httpContext, HttpUriRequest uriRequest, String contentType, ResponseHandlerInterface responseHandler, Context context) {
        return new AsyncHttpRequest(client, httpContext, uriRequest, responseHandler);
    }
{% endhighlight %}

实质就是得到了一个AsyncHttpRequest的实例，继续看下会发现AsyncHttpRequest implements Runnable，这就是submit到线程池的Runnable了。

至此发送请求过程就结束了。

接收过程更容易，所以不做分析。

##**总结几句**

回过头会发现在我们的请求中最好都加上Context参数，因为这样可以在Activity pause或stop时取消掉没用的请求。

再来整理下整个类功能：

AsyncHttpClient 核心类，使用HttpClient执行网络请求，提供了get，put，post，delete，head等请求方法，
使用起来很简单，只需以url及RequestParams调用相应的方法即可，还可以选择性地传入Context，
用于取消Content相关的请求，同时必须提供ResponseHandlerInterface（AsyncHttpResponseHandler继承自ResponseHandlerInterface）的实现类，
一般为AsyncHttpResponseHandler的子类，AsyncHttpClient内部有一个线程池，当使用AsyncHttpClient执行网络请求时，
最终都会调用sendRequest方法，在这个方法内部将请求参数封装成AsyncHttpRequest（继承自Runnable）交由内部的线程池执行。

SyncHttpClient 继承自AsyncHttpClient，同步执行网络请求，AsyncHttpClient把请求封装成AsyncHttpRequest后提交至线程池，
SyncHttpClient把请求封装成AsyncHttpRequest后直接调用它的run方法。

AsyncHttpRequest 继承自Runnabler，被submit至线程池执行网络请求并发送start，success等消息。

AsyncHttpResponseHandler 接收请求结果，一般重写onSuccess及onFailure接收请求成功或失败的消息，还有onStart，onFinish等消息。

TextHttpResponseHandler、JsonHttpResponseHandler、BaseJsonHttpResponseHandler这些类都继承自AsyncHttpResponseHandler，
只是重写了AsyncHttpResponseHandler的onSuccess和onFailure方法，将请求结果进行了转换而已。

RequestParams 请求参数，可以添加普通的字符串参数，并可添加File，InputStream上传文件。

**最近不在状态，深入分析待日后补充。。。**