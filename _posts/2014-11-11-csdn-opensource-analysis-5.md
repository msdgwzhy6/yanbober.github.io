---
layout: post
keywords: async-http, 框架, android-async-http
description: android-async-http框架
title: "android-async-http框架库使用基础"
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

##**背景知识**

开始使用分析前还是先了解下Android的HTTP一些过往趣事：

[关于Android HTTP推荐的Google官方链接](http://android-developers.blogspot.com/2011/09/androids-http-clients.html) 

HttpClient拥有众多的API，实现稳定，bug很少。
HttpURLConnection是一种多用途、轻量的HTTP客户端，使用它来进行HTTP操作可以适用于大多数的应用程序。
HttpURLConnection的API比较简单、扩展容易。不过在Android 2.2版本之前，HttpURLConnection一直存在着一些bug。
比如说对一个可读的InputStream调用close()方法时，就有可能会导致连接池失效了。
所以说2.2之前推荐使用HttpClient，2.2之后推荐HttpURLConnection。

好了，那现在话又说回来，在android-async-http中使用的是HttpClient。哎...好像在Volley中分析过Volley对不同版本进行了判断，
所以针对不同版本分别使用了HttpClient和HttpURLConnection。还是google牛逼啊！

回过神继续android-async-http吧，不瞎扯了。

android-async-http是专门针对Android在Apache的HttpClient基础上构建的异步http连接。所有的请求全在UI（主）线程之外执行，
而callback使用了Android的Handler发送消息机制在创建它的线程中执行。

类似Volley一样，使用一个优秀框架之前就是必须得先知道他的特性，如下就是android-async-http的特性：

1. 发送异步http请求，在匿名callback对象中处理response信息；
2. http请求发生在UI（主）线程之外的异步线程中；
3. 内部采用线程池来处理并发请求；
4. 通过RequestParams类构造GET/POST；
5. 内置多部分文件上传，不需要第三方库支持；
6. 流式Json上传，不需要额外的库；
7. 能处理环行和相对重定向；
8. 和你的app大小相比来说，库的size很小，所有的一切只有90kb；
9. 在各种各样的移动连接环境中具备自动智能请求重试机制；
10. 自动的gzip响应解码；
11. 内置多种形式的响应解析，有原生的字节流，string，json对象，甚至可以将response写到文件中；
12. 永久的cookie保存，内部实现用的是Android的SharedPreferences；
13. 通过BaseJsonHttpResponseHandler和各种json库集成；
14. 支持SAX解析器；
15. 支持各种语言和content编码，不仅仅是UTF-8；

<hr>

##**整体操作流程**

android-async-http最简单基础的使用只需如下步骤：

1. 创建一个AsyncHttpClient；
2. （可选的）通过RequestParams对象设置请求参数；
3. 调用AsyncHttpClient的某个get方法，传递你需要的（成功和失败时）callback接口实现，一般都是匿名内部类
，实现了AsyncHttpResponseHandler，类库自己也提供许多现成的response handler，你一般不需要自己创建。

<hr>

##**AsyncHttpClient与AsyncHttpResponseHandler基础GET体验**

AsyncHttpClient类通常用在android应用程序中创建异步GET, POST, PUT和DELETE HTTP请求，
请求参数通过RequestParams实例创建，响应通过重写匿名内部类ResponseHandlerInterface方法处理。

如下代码展示了使用AsyncHttpClient与AsyncHttpResponseHandler的基础操作：

{% highlight ruby %}
AsyncHttpClient client = new AsyncHttpClient();
client.get("www.baidu.com", new AsyncHttpResponseHandler() {
    @Override
    public void onSuccess(int statusCode, Header[] headers, byte[] responseBody) {

    }

    @Override
    public void onFailure(int statusCode, Header[] headers, byte[] responseBody, Throwable error) {

    }

    @Override
    public void onStart() {
        super.onStart();
    }

    @Override
    public void onFinish() {
        super.onFinish();
    }

    @Override
    public void onRetry(int retryNo) {
        super.onRetry(retryNo);
    }

    @Override
    public void onCancel() {
        super.onCancel();
    }

    @Override
    public void onProgress(int bytesWritten, int totalSize) {
        super.onProgress(bytesWritten, totalSize);
    }
});
{% endhighlight %}

<hr>

##**官方推荐AsyncHttpClient静态实例化的封装**

**注意**：官方推荐使用一个静态的AsyncHttpClient，官方示例代码如下：

{% highlight ruby %}
public class TwitterRestClient {
    private static final String BASE_URL = "http://api.twitter.com/1/";

    private static AsyncHttpClient client = new AsyncHttpClient();

    public static void get(String url, RequestParams params, AsyncHttpResponseHandler responseHandler) {
        client.get(getAbsoluteUrl(url), params, responseHandler);
    }

    public static void post(String url, RequestParams params, AsyncHttpResponseHandler responseHandler) {
        client.post(getAbsoluteUrl(url), params, responseHandler);
    }

    private static String getAbsoluteUrl(String relativeUrl) {
        return BASE_URL + relativeUrl;
    }
}
{% endhighlight %}

通过官方这个推荐例子可以发现，我们在用时可以直接通过类名调用需要的请求方法。所以我们可以自己多封装一些不同的请求方法，比如参数不同的方法，下载方法，
上传方法等。

<hr>

##**RequestParams的基础使用**

{% highlight ruby %}
RequestParams params = new RequestParams();
params.put("username", "yanbober");
params.put("password", "123456");
params.put("email", "yanbobersky@email.com");

/*
* Upload a File
*/
params.put("file_pic", new File("test.jpg"));
params.put("file_inputStream", inputStream);
params.put("file_bytes", new ByteArrayInputStream(bytes));

/*
* url params: "user[first_name]=jesse&user[last_name]=yan"
*/
Map<String, String> map = new HashMap<String, String>();
map.put("first_name", "jesse");
map.put("last_name", "yan");
params.put("user", map);

/*
* url params: "what=haha&like=wowo"
*/
Set<String> set = new HashSet<String>();
set.add("haha");
set.add("wowo");
params.put("what", set);

/*
* url params: "languages[]=Java&languages[]=C"
*/
List<String> list = new ArrayList<String>();
list.add("Java");
list.add("C");
params.put("languages", list);

/*
* url params: "colors[]=blue&colors[]=yellow"
*/
String[] colors = { "blue", "yellow" };
params.put("colors", colors);

/*
* url params: "users[][age]=30&users[][gender]=male&users[][age]=25&users[][gender]=female"
*/
List<Map<String, String>> listOfMaps = new ArrayList<Map<String, String>>();
Map<String, String> user1 = new HashMap<String, String>();
user1.put("age", "30");
user1.put("gender", "male");
Map<String, String> user2 = new HashMap<String, String>();
user2.put("age", "25");
user2.put("gender", "female");
listOfMaps.add(user1);
listOfMaps.add(user2);
params.put("users", listOfMaps);

/*
* 使用实例
*/
AsyncHttpClient client = new AsyncHttpClient();
client.post("http://localhost:8080/androidtest/", params, responseHandler);
{% endhighlight %}

<hr>

##**JsonHttpResponseHandler带Json参数的POST**

{% highlight ruby %}
try {
    JSONObject jsonObject = new JSONObject();
    jsonObject.put("username", "ryantang");
    StringEntity stringEntity = new StringEntity(jsonObject.toString());
    client.post(mContext, "http://api.com/login", stringEntity, "application/json", new JsonHttpResponseHandler(){
        @Override
        public void onSuccess(JSONObject jsonObject) {
            super.onSuccess(jsonObject);
        }
    });
} catch (JSONException e) {
    e.printStackTrace();
} catch (UnsupportedEncodingException e) {
    e.printStackTrace();
}
{% endhighlight %}

<hr>

##**BinaryHttpResponseHandler下载文件**

{% highlight ruby %}
client.get("http://download/file/test.java", new BinaryHttpResponseHandler() {
    @Override
    public void onSuccess(byte[] arg0) {
        super.onSuccess(arg0);
        File file = Environment.getExternalStorageDirectory();
        File file2 = new File(file, "down");
        file2.mkdir();
        file2 = new File(file2, "down_file.jpg");
        try {
            FileOutputStream oStream = new FileOutputStream(file2);
            oStream.write(arg0);
            oStream.flush();
            oStream.close();
        } catch (Exception e) {
            e.printStackTrace();
            Log.i(null, e.toString());
        }
    }
});
{% endhighlight %}

<hr>

##**RequestParams上传文件**

{% highlight ruby %}
File myFile = new File("/sdcard/test.java");
RequestParams params = new RequestParams();
try {
    params.put("filename", myFile);
    AsyncHttpClient client = new AsyncHttpClient();
    client.post("http://update/server/location/", params, new AsyncHttpResponseHandler(){
        @Override
        public void onSuccess(int statusCode, String content) {
            super.onSuccess(statusCode, content);
        }
    });
} catch(FileNotFoundException e) {
    e.printStackTrace();
}
{% endhighlight %}

<hr>

##**PersistentCookieStore持久化存储cookie**

官方文档里说PersistentCookieStore类用于实现Apache HttpClient的CookieStore接口，
可自动将cookie保存到Android设备的SharedPreferences中，
如果你打算使用cookie来管理验证会话，这个非常有用，因为用户可以保持登录状态，不管关闭还是重新打开你的app。

文档里介绍了持久化Cookie的步骤：

1. 创建 AsyncHttpClient实例对象；
2. 将客户端的cookie保存到PersistentCookieStore实例对象，带有activity或者应用程序context的构造方法；
3. 任何从服务器端获取的cookie都会持久化存储到myCookieStore中，添加一个cookie到存储中，只需要构造一个新的cookie对象，并且调用addCookie方法；

下面这个例子就是铁证：

{% highlight ruby %}
AsyncHttpClient client = new AsyncHttpClient(); 

PersistentCookieStore cookieStore = new PersistentCookieStore(this);  
client.setCookieStore(cookieStore); 

BasicClientCookie newCookie = new BasicClientCookie("name", "value");  
newCookie.setVersion(1);  
newCookie.setDomain("mycompany.com");  
newCookie.setPath("/");  
cookieStore.addCookie(newCookie);
{% endhighlight %}

<hr>

##**总结性的唠叨几句**

AsyncHttpResponseHandler是一个请求返回处理、成功、失败、开始、完成等自定义的消息的类，如上第一个基础例子中所示。

BinaryHttpResponseHandler是继承AsyncHttpResponseHandler的子类，这是一个字节流返回处理的类，用于处理图片等类。

JsonHttpResponseHandler是继承AsyncHttpResponseHandler的子类，这是一个json请求返回处理服务器与客户端用json交流时使用的类。

AsyncHttpRequest继承自Runnable，是基于线程的子类，用于异步请求类， 通过AsyncHttpResponseHandler回调。

PersistentCookieStore继承自CookieStore，是一个基于CookieStore的子类， 使用HttpClient处理数据，并且使用cookie持久性存储接口。

**PS：例子用的在牛逼还不如阅读开头列出的官方文档和源码吧。**
