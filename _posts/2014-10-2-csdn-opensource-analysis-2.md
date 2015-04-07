---
layout: post
keywords: Volley, 框架, Android HTTP
description: Google Android Volley网络库
title: "Google Volley使用之基础"
categories: [开源框架库笔记]
tags: [Volley, Android网络请求]
group: archive
icon: file-alt
---
{% include site/setup %}

<hr>
##**开源项目链接**

[Volley Android Developer文档](http://developer.android.com/training/volley/index.html)

Volley主页：https://android.googlesource.com/platform/frameworks/volley

Volley仓库：git clone https://android.googlesource.com/platform/frameworks/volley

Volley GitHub Demo：在GitHub主页搜索Volley会有很多，不过建议阅读Android Developer文档。

<hr>

##**背景知识**

啥是Google Android Volley？

一句简单的回答：Volley就是集AsyncHttpClient和Universal-Image-Loader优点于一身的一个Google亲儿子开源框架。

Google在I/O 2013大会上发布了Volley。它是Android平台上的网络通信库，能使网络通信更快，更简单，更健壮。其最明显的一个优点就是特别适合数据量不大但是通信频繁的场景，
最明显的缺点就是大数据传输表现的很糟糕。

Volley提供了如下的便利功能：

- JSON数据和图像等的异步下载；
- 网络请求排序（scheduling）；
- 网络请求优先级处理；
- 缓存；
- 多级别取消请求；
- 与Activity生命周期联动（Activity结束时同时取消所有网络请求）；

Volley对APP的版本要求是android:minSdkVersion为8。

<hr>

##**使用newRequestQueue和StringRequest**

Volley的用法非常简单，这里使用官方例子发起一条HTTP GET请求，然后接收HTTP响应。首先需要获取到一个RequestQueue对象，可以调用如下方法获取到：

{% highlight ruby %}
RequestQueue mQueue = Volley.newRequestQueue(context);
{% endhighlight %}

这里拿到的RequestQueue是一个请求队列对象，它可以缓存所有的HTTP请求，然后按照一定的算法并发地发出这些请求。
RequestQueue内部的设计就是非常合适高并发的，因此我们不必为每一次HTTP请求都创建一个RequestQueue对象，这是非常浪费资源的，
基本上在每一个需要和网络交互的Activity中创建一个RequestQueue对象就足够了。
 
StringRequest的构造函数需要传入四个参数（有一个三个参数的构造函数，默认是GET方式），第一个参数就是目标服务器的URL地址，
第二个参数是服务器响应成功的回调，第三个参数是服务器响应失败的回调。

将这个StringRequest对象添加到RequestQueue里面就可以了。

使用Volley时您可以从任何线程开始请求，但响应始终传递到了主线程上。

{% highlight ruby %}
final TextView mTextView = (TextView) findViewById(R.id.text);
...

// Instantiate the RequestQueue.
RequestQueue queue = Volley.newRequestQueue(this);
String url ="http://www.google.com";

// Request a string response from the provided URL.
StringRequest stringRequest = new StringRequest(Request.Method.GET, url,
            new Response.Listener<String>() {
    @Override
    public void onResponse(String response) {
        // Display the first 500 characters of the response string.
        mTextView.setText("Response is: "+ response.substring(0,500));
    }
}, new Response.ErrorListener() {
    @Override
    public void onErrorResponse(VolleyError error) {
        mTextView.setText("That didn't work!");
    }
});
// Add the request to the RequestQueue.
queue.add(stringRequest);
{% endhighlight %}

如上代码就示范了一个简单的HTTP GET请求。

既然HTTP GET的例子这么easy搞定，是不是该再整一个POST的例子呢？

POST有点曲线救国，因为StringRequest中并没有提供设置POST参数的方法，但是当发出POST请求的时候，Volley会尝试调用StringRequest父类Request的getParams()方法来获取POST参数，
所以我们只需要重写StringRequest的getParams()方法，在这里设置POST参数就可以了，代码如下所示：

{% highlight ruby %}
RequestQueue queue = Volley.newRequestQueue(this);

StringRequest stringRequest = new StringRequest(Method.POST, url,  listener, errorListener) {  
    @Override 
    protected Map<String, String> getParams() throws AuthFailureError {  
        Map<String, String> map = new HashMap<String, String>();  
        map.put("key1", "value1");  
        map.put("key2", "value2");  
        return map;  
    }  
}; 

queue.add(stringRequest);
{% endhighlight %}

如上代码就示范了一个简单的HTTP POST请求。

<hr>

##**使用newRequestQueue和JsonRequest**

和StringRequest基本上差不多，JsonRequest也是继承自Request类的，不过JsonRequest是一个抽象类，我们无法直接创建它的实例。JsonRequest有两个直接的子类，JsonObjectRequest和JsonArrayRequest。
所以对于JSON格式的数据基本没啥压力了，他都帮你搞完了，如下展示一个简单例子：

{% highlight ruby %}
RequestQueue queue = Volley.newRequestQueue(this);

JsonObjectRequest jsonObjectRequest = new JsonObjectRequest("http://json_test.php", null,  
        new Response.Listener<JSONObject>() {  
            @Override 
            public void onResponse(JSONObject response) {  
                Log.d("TAG", response.toString());  
            }  
        }, new Response.ErrorListener() {  
            @Override 
            public void onErrorResponse(VolleyError error) {  
                Log.e("TAG", error.getMessage(), error);  
            }  
        });  
		
queue.add(stringRequest);
{% endhighlight %}

如上就是一个Volley的JSON HTTP的简单例子咯！

<hr>

##**取消Request**

要取消一个请求，调用cancel()即可。一旦取消，Volley保证你的响应处理程序将永远不会被调用。
这意味着在实践中，你可以取消所有待定的请求在你Activity的onStop()方法中，
你不必乱抛垃圾的响应处理程序或者检查getActivity() == NULL，无论onSaveInstanceState()是否已经被调用。

你发出去的请求必须要自己保证是可控的，在这里还有一个更简单的方法：你可以标记发送的每个请求对象，然后你可以使用这个标签来提供请求取消的范围。

这里是一个使用字符串值标签的例子：

{% highlight ruby %}
public static final String TAG = "MyTag";
StringRequest stringRequest; // Assume this exists.
RequestQueue mRequestQueue;  // Assume this exists.

// Set the tag on the request.
stringRequest.setTag(TAG);

// Add the request to the RequestQueue.
mRequestQueue.add(stringRequest);


@Override
protected void onStop () {
    super.onStop();
    if (mRequestQueue != null) {
        mRequestQueue.cancelAll(TAG);
    }
}
{% endhighlight %}

现在是不是发现Volley灰常简单了呢！就是举一反三的操作。

好了，不扯了，网络HTTP的Volley简单使用先说到这里。

<hr>

##**使用newRequestQueue和ImageRequest**

ImageRequest也是继承自Request的，因此它的用法和上面的也是基本相同的。

{% highlight ruby %}
ImageRequest imageRequest = new ImageRequest(  
        "http://developer.android.com/images/home/aw_dac.png",  
        new Response.Listener<Bitmap>() {  
            @Override 
            public void onResponse(Bitmap response) {  
                imageView.setImageBitmap(response);  
            }  
        }, 0, 0, Config.RGB_565, new Response.ErrorListener() {  
            @Override 
            public void onErrorResponse(VolleyError error) {  
                imageView.setImageResource(R.drawable.default_image);  
            }  
        });  
{% endhighlight %}

ImageRequest的构造函数有六个参数，第一个参数是图片URL；第二个参数是图片请求成功的回调；第三第四个参数分别用于指定允许图片最大的宽度和高度，
如果指定的网络图片的宽度或高度大于这里的最大值，则会对图片进行压缩，指定成0的话就表示不管图片有多大，都不会进行压缩；
第五个参数用于指定图片深度；第六个参数是图片请求失败的回调。

<hr>

##**使用newRequestQueue和ImageLoader**

ImageLoader也可以用于加载网络上的图片，并且它的内部也是使用ImageRequest来实现的，不过ImageLoader明显要比ImageRequest更加高效，
因为它不仅可以帮我们对图片进行缓存，还可以过滤掉重复的链接，避免重复发送请求。

由于ImageLoader已经不是继承自Request，所以它的用法变为如下：

- 创建一个RequestQueue对象。
- 创建一个ImageLoader对象。
- 获取一个ImageListener对象。
- 调用ImageLoader的get()方法加载网络上的图片。

{% highlight ruby %}
RequestQueue mQueue = Volley.newRequestQueue(context);

ImageLoader imageLoader = new ImageLoader(mQueue, new ImageCache() {  
    @Override 
    public void putBitmap(String url, Bitmap bitmap) {  
		//未实现
    }  
 
    @Override 
    public Bitmap getBitmap(String url) {  
        //未实现
		return null;  
    }  
});

ImageListener listener = ImageLoader.getImageListener(imageView,  
        R.drawable.default_image, R.drawable.failed_image);
		
imageLoader.get("http://google/pictures/1201.jpeg", listener, 200, 200);		
{% endhighlight %}

如上简单使用了ImageLoader进行图片加载。

现在进行升级，实现ImageCache接口如下：

{% highlight ruby %}
public class BitmapCache implements ImageCache {  
 
    private LruCache<String, Bitmap> mCache;  
 
    public BitmapCache() {  
        int maxSize = 5 * 1024 * 1024;  //演示写死，实际需要动态计算
        mCache = new LruCache<String, Bitmap>(maxSize) {  
            @Override 
            protected int sizeOf(String key, Bitmap bitmap) {  
                return bitmap.getRowBytes() * bitmap.getHeight();  
            }  
        };  
    }  
 
    @Override 
    public Bitmap getBitmap(String url) {  
        return mCache.get(url);  
    }  
 
    @Override 
    public void putBitmap(String url, Bitmap bitmap) {  
        mCache.put(url, bitmap);  
    }  
}
{% endhighlight %}

这里这么搞一下就简单的把ImageLoader优势展示了一把。

<hr>

##**使用newRequestQueue和NetworkImageView**

NetworkImageView是一个继承自ImageView的View控件，拥有ImageView控件的所有功能，并且在ImageView的基础上加入了加载网络图片的功能。
NetworkImageView控件的用法分为以下五步：
 
- 创建一个RequestQueue对象。
- 创建一个ImageLoader对象。
- 在布局文件中添加一个NetworkImageView控件。
- 在代码中获取该控件的实例。
- 设置要加载的图片地址。

这里以XML文件结合java的方式演示：

{% highlight ruby %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android" 
    android:layout_width="fill_parent" 
    android:layout_height="fill_parent" 
    android:orientation="vertical" > 
      
    <com.android.volley.toolbox.NetworkImageView   
        android:id="@+id/network_image_view" 
        android:layout_width="200dp" 
        android:layout_height="200dp" 
        android:layout_gravity="center_horizontal" /> 
 
</LinearLayout>
{% endhighlight %}

{% highlight ruby %}
networkImageView = (NetworkImageView) findViewById(R.id.network_image_view);

networkImageView.setDefaultImageResId(R.drawable.default_image);  
networkImageView.setErrorImageResId(R.drawable.failed_image);  
networkImageView.setImageUrl("http://img.my.csdn.net/uploads/201404/13/1397393290_5765.jpeg", imageLoader);
{% endhighlight %}

如上演示了Volley的一些基本使用特性和方法。

##**Volley请求管理方式**

在Android Developer上看到的这幅图：

<img src="http://yanbober.github.io/image/open/volley_1.png" />

RequestQueue会维护一个缓存调度线程（cache线程）和一个网络调度线程池（net线程），
当一个Request被加到队列中的时候，cache线程会把这个请求进行筛选：如果这个请求的内容可以在缓存中找到，
cache线程会亲自解析相应内容，并分发到主线程（UI）。
如果缓存中没有，这个request就会被加入到另一个NetworkQueue，所有真正准备进行网络通信的request都在这里，
第一个可用的net线程会从NetworkQueue中拿出一个request扔向服务器。
当响应数据到的时候，这个net线程会解析原始响应数据，写入缓存，并把解析后的结果返回给主线程。
