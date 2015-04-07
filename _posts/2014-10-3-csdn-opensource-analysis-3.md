---
layout: post
keywords: Volley, 框架, Android HTTP
description: Google Android Volley网络库
title: "Google Volley使用之自定义"
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

Most requests have ready-to-use implementations in the toolbox; if your response is a string, image, or JSON,
you probably won't need to implement a custom Request.

For cases where you do need to implement a custom request, this is all you need to do:

- Extend the Request<T> class, where <T> represents the type of parsed response the request expects.
So if your parsed response is a string, for example, create your custom request by extending Request<String>.
See the Volley toolbox classes StringRequest and ImageRequest for examples of extending Request<T>.
- Implement the abstract methods parseNetworkResponse() and deliverResponse(), described in more detail below.

正如官方牛逼的说法一样：

你要是请求的是string, image, or JSON还好办，有现成的，前一篇已经详细说明了。But你要是返回的不是这些呢？那就比较蛋疼，需要自定义。
不过好的一点是Volley框架的扩展性非常好。所以如果需要customer的话你需要按照如下处理：

- 继承Request<T>类，<T>就是你的响应数据格式。你可以在写customer的时候参考StringRequest实现。
- 实现parseNetworkResponse() and deliverResponse()两个抽象方法。
在StringRequest中，deliverResponse()方法调用了mListener中的onResponse()方法，并将response内容传入。
parseNetworkResponse()方法对服务器响应的数据进行解析，数据是字节的形式放在NetworkResponse的data变量中的，
这里将数据取出然后组装成一个String，并传入Response的success()方法中。

<hr>

##**开搞一个实现**

{% highlight ruby %}
public class GsonRequest<T> extends Request<T> {
    private final Gson gson = new Gson();
    private final Class<T> clazz;
    private final Map<String, String> headers;
    private final Listener<T> listener;

    /**
     * Make a GET request and return a parsed object from JSON.
     *
     * @param url URL of the request to make
     * @param clazz Relevant class object, for Gson's reflection
     * @param headers Map of request headers
     */
    public GsonRequest(String url, Class<T> clazz, Map<String, String> headers,
            Listener<T> listener, ErrorListener errorListener) {
        super(Method.GET, url, errorListener);
        this.clazz = clazz;
        this.headers = headers;
        this.listener = listener;
    }

    @Override
    public Map<String, String> getHeaders() throws AuthFailureError {
        return headers != null ? headers : super.getHeaders();
    }

    @Override
    protected void deliverResponse(T response) {
        listener.onResponse(response);
    }

    @Override
    protected Response<T> parseNetworkResponse(NetworkResponse response) {
        try {
            String json = new String(
                    response.data,
                    HttpHeaderParser.parseCharset(response.headers));
            return Response.success(
                    gson.fromJson(json, clazz),
                    HttpHeaderParser.parseCacheHeaders(response));
        } catch (UnsupportedEncodingException e) {
            return Response.error(new ParseError(e));
        } catch (JsonSyntaxException e) {
            return Response.error(new ParseError(e));
        }
    }
}
{% endhighlight %}

这是官方的GSON的反馈解析实现。

{% highlight ruby %}
public class XMLRequest extends Request<XmlPullParser> {  
 
    private final Listener<XmlPullParser> listener;  
 
    public XMLRequest(int method, String url, Listener<XmlPullParser> listener,  
            ErrorListener errorListener) {  
        super(method, url, errorListener);  
        this.listener = listener;  
    }  
 
    public XMLRequest(String url, Listener<XmlPullParser> listener, ErrorListener errorListener) {  
        this(Method.GET, url, listener, errorListener);  
    }  
 
    @Override 
    protected Response<XmlPullParser> parseNetworkResponse(NetworkResponse response) {  
        try {  
            String xmlString = new String(response.data,  
                    HttpHeaderParser.parseCharset(response.headers));  
            XmlPullParserFactory factory = XmlPullParserFactory.newInstance();  
            XmlPullParser xmlPullParser = factory.newPullParser();  
            xmlPullParser.setInput(new StringReader(xmlString));  
            return Response.success(xmlPullParser, HttpHeaderParser.parseCacheHeaders(response));  
        } catch (UnsupportedEncodingException e) {  
            return Response.error(new ParseError(e));  
        } catch (XmlPullParserException e) {  
            return Response.error(new ParseError(e));  
        }  
    }  
 
    @Override 
    protected void deliverResponse(XmlPullParser response) {  
        listener.onResponse(response);  
    }  
}
{% endhighlight %}

这是一个XmlPullParser反馈解析的实现。

通过如上你会发现Volley框架不愧于是Google大牛搞的，连拓展自定义都这么方便，设计模式运用的出神入化，膜拜。
