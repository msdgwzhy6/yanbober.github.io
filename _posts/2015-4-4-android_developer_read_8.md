---
layout: post
keywords: Android Developer
description: Android Developer读书笔记
title: "[Training] Displaying Bitmaps Efficiently"
categories: [开发文档]
tags: [Android读书笔记]
group: archive
icon: file-alt
---
{% include site/setup %}
<hr>
重读Android Developer笔记核心记录
<hr>

#**Loading Large Bitmaps Efficiently**

原文重点摘要：

There are a number of reasons why loading bitmaps in your Android application is tricky:

- Mobile devices typically have constrained system resources. Android devices can have as little as 16MB of memory available to a single application.
The Android Compatibility Definition Document (CDD), Section 3.7. Virtual Machine Compatibility gives the required minimum application memory for various
screen sizes and densities. Applications should be optimized to perform under this minimum memory limit. However,
keep in mind many devices are configured with higher limits.
- Bitmaps take up a lot of memory, especially for rich images like photographs. For example, the camera on the Galaxy Nexus takes photos up to 2592x1936 pixels
(5 megapixels). If the bitmap configuration used is ARGB_8888 (the default from the Android 2.3 onward) then loading this image into memory takes about 19MB
of memory (2592*1936*4 bytes), immediately exhausting the per-app limit on some devices.
- Android app UI’s frequently require several bitmaps to be loaded at once. Components such as ListView, GridView and ViewPager commonly include multiple bitmaps
on-screen at once with many more potentially off-screen ready to show at the flick of a finger.

精髓指点：

如下就是一系列导致你使用Bitmap时OutofMemoryError的原因：

- 系统对每个应用都有一个内存限制，通常是16M，但是不同设备也不一定，需要依据系统客户化决定，目的就是为了在屏幕尺寸、密度上显示的更加好。
- Bitmap需要很多内存，尤其是富图片。
- 当手指滑动时ListView、GridView、ViewPager等都需要多次重加载其内部图片，所以也很容易溢出。

原文重点摘要：

Images come in all shapes and sizes. In many cases they are larger than required for a typical application user interface (UI). 

精髓指点：

很多情况下处理相同大小的Bitmap比UI显示需要更多的内存。

##**Read Bitmap Dimensions and Type**

原文重点摘要：

The BitmapFactory class provides several decoding methods (decodeByteArray(), decodeFile(), decodeResource(), etc.) for creating a Bitmap from various sources.
Choose the most appropriate decode method based on your image data source.
These methods attempt to allocate memory for the constructed bitmap and therefore can easily result in an OutOfMemory exception. Each type of decode method
has additional signatures that let you specify decoding options via the BitmapFactory.Options class. Setting the inJustDecodeBounds property to true while
decoding avoids memory allocation, returning null for the bitmap object but setting outWidth, outHeight and outMimeType. This technique allows you to read
the dimensions and type of the image data prior to construction (and memory allocation) of the bitmap.

{% highlight ruby %}
BitmapFactory.Options options = new BitmapFactory.Options();
options.inJustDecodeBounds = true;
BitmapFactory.decodeResource(getResources(), R.id.myimage, options);
int imageHeight = options.outHeight;
int imageWidth = options.outWidth;
String imageType = options.outMimeType;
{% endhighlight %}

To avoid java.lang.OutOfMemory exceptions, check the dimensions of a bitmap before decoding it, unless you absolutely trust the source to provide you with
predictably sized image data that comfortably fits within the available memory.

精髓指点：

BitmapFactory提供一系列解析不同来源的解码方法，譬如decodeByteArray(), decodeFile(), decodeResource()等。这些方法都会尝试通过bitmap构造函数申请内存，所以很容易导致
OutOfMemory异常。每一个解码方法都提供通过BitmapFactory.Options设置的参数项。设置inJustDecodeBounds属性为true时可以避免申请内存，同时返回null的bitmap对象，所以通过
这个特性我们可以很容易获得bitmap的尺寸和类型等数据。

如果你想避免OOM错误，必须在decode前check图片的尺寸，除非你非常确信图片的来源。

##**Load a Scaled Down Version into Memory**

原文重点摘要：

Here are some factors to consider:

- Estimated memory usage of loading the full image in memory.
- Amount of memory you are willing to commit to loading this image given any other memory requirements of your application.
- Dimensions of the target ImageView or UI component that the image is to be loaded into.
- Screen size and density of the current device.

For example, it’s not worth loading a 1024x768 pixel image into memory if it will eventually be displayed in a 128x96 pixel thumbnail in an ImageView.

To tell the decoder to subsample the image, loading a smaller version into memory, set inSampleSize to true in your BitmapFactory.Options object.
For example, an image with resolution 2048x1536 that is decoded with an inSampleSize of 4 produces a bitmap of approximately 512x384.
Loading this into memory uses 0.75MB rather than 12MB for the full image (assuming a bitmap configuration of ARGB_8888). Here’s a method to
calculate a sample size value that is a power of two based on a target width and height:

{% highlight ruby %}
public static int calculateInSampleSize(BitmapFactory.Options options, int reqWidth, int reqHeight) {
    // Raw height and width of image
    final int height = options.outHeight;
    final int width = options.outWidth;
    int inSampleSize = 1;

    if (height > reqHeight || width > reqWidth) {

        final int halfHeight = height / 2;
        final int halfWidth = width / 2;

        // Calculate the largest inSampleSize value that is a power of 2 and keeps both
        // height and width larger than the requested height and width.
        while ((halfHeight / inSampleSize) > reqHeight
                && (halfWidth / inSampleSize) > reqWidth) {
            inSampleSize *= 2;
        }
    }

    return inSampleSize;
}
{% endhighlight %}

Note: A power of two value is calculated because the decoder uses a final value by rounding down to the nearest power of two, as per the inSampleSize documentation.

To use this method, first decode with inJustDecodeBounds set to true, pass the options through and then decode again using the new inSampleSize value
and inJustDecodeBounds set to false:

{% highlight ruby %}
public static Bitmap decodeSampledBitmapFromResource(Resources res, int resId,
        int reqWidth, int reqHeight) {

    // First decode with inJustDecodeBounds=true to check dimensions
    final BitmapFactory.Options options = new BitmapFactory.Options();
    options.inJustDecodeBounds = true;
    BitmapFactory.decodeResource(res, resId, options);

    // Calculate inSampleSize
    options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);

    // Decode bitmap with inSampleSize set
    options.inJustDecodeBounds = false;
    return BitmapFactory.decodeResource(res, resId, options);
}
{% endhighlight %}

This method makes it easy to load a bitmap of arbitrarily large size into an ImageView that displays a 100x100 pixel thumbnail, as shown in the following example code:

{% highlight ruby %}
mImageView.setImageBitmap(
    decodeSampledBitmapFromResource(getResources(), R.id.myimage, 100, 100));
{% endhighlight %}

You can follow a similar process to decode bitmaps from other sources, by substituting the appropriate BitmapFactory.decode* method as needed.

精髓指点：

是否缩放图片有一些因素需要考虑：

- 估算加载全图需要的内存大小。
- 你应用能使用的内存大小。
- 将要显示的UI或者View的尺寸。
- 当前设备的屏幕尺寸和像素密度。

例如，显示128x96的ImageView加载1024x768的图片到内存是很浪费的。

通过设置inSampleSize属性可以告诉decoder图片采样的深度。例如，2048x1536的图片设置inSampleSize为4，则变为了512x384尺寸，假设这个图片设置为ARGB_8888，那么本来需要
12MB内存变为0.75MB。如上第一个方法就是通过宽高来计算深度的办法。

使用这个方法的第一步就是设置inJustDecodeBounds为true，然后设置inSampleSize值，最后切记将inJustDecodeBounds再设置回false。如上第二段代码就是完整的decode方法。

<hr>

#**Processing Bitmaps Off the UI Thread**

原文重点摘要：

The BitmapFactory.decode* methods, discussed in the Load Large Bitmaps Efficiently lesson, should not be executed on the main UI thread if the source data
is read from disk or a network location (or really any source other than memory). The time this data takes to load is unpredictable and depends on a variety
of factors (speed of reading from disk or network, size of image, power of CPU, etc.). If one of these tasks blocks the UI thread, the system flags your
application as non-responsive and the user has the option of closing it (see Designing for Responsiveness for more information).

精髓指点：

这一讲教你如何在子线程处理Bitmap任务和处理并发操作。因为BitmapFactory.decode*系列方法除过直接在内存加载以外的通过磁盘、网络等加载耗时是不确定的。你需要考虑
UI无响应异常的情况。

##**Use an AsyncTask**

原文重点摘要：

Here’s an example of loading a large image into an ImageView using AsyncTask and decodeSampledBitmapFromResource():

{% highlight ruby %}
class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {
    private final WeakReference<ImageView> imageViewReference;
    private int data = 0;

    public BitmapWorkerTask(ImageView imageView) {
        // Use a WeakReference to ensure the ImageView can be garbage collected
        imageViewReference = new WeakReference<ImageView>(imageView);
    }

    // Decode image in background.
    @Override
    protected Bitmap doInBackground(Integer... params) {
        data = params[0];
        return decodeSampledBitmapFromResource(getResources(), data, 100, 100));
    }

    // Once complete, see if ImageView is still around and set bitmap.
    @Override
    protected void onPostExecute(Bitmap bitmap) {
        if (imageViewReference != null && bitmap != null) {
            final ImageView imageView = imageViewReference.get();
            if (imageView != null) {
                imageView.setImageBitmap(bitmap);
            }
        }
    }
}
{% endhighlight %}

The WeakReference to the ImageView ensures that the AsyncTask does not prevent the ImageView and anything it references from being garbage collected.
There’s no guarantee the ImageView is still around when the task finishes, so you must also check the reference in onPostExecute().
The ImageView may no longer exist, if for example, the user navigates away from the activity or if a configuration change happens before the task finishes.

To start loading the bitmap asynchronously, simply create a new task and execute it:

{% highlight ruby %}
public void loadBitmap(int resId, ImageView imageView) {
    BitmapWorkerTask task = new BitmapWorkerTask(imageView);
    task.execute(resId);
}
{% endhighlight %}

精髓指点：

如上就是一个异步加载处理Bitmap的例子，使用了AsyncTask。

ImageView的WeakReference保证了在垃圾回收时AsyncTask不阻止（不强行持有引用）ImageView的回收。不敢保证在AsyncTask结束时ImageView的引用还存在，所以你需要在
onPostExecute方法中check他是否为null。

##**Handle Concurrency**

原文重点摘要：

{% highlight ruby %}
static class AsyncDrawable extends BitmapDrawable {
    private final WeakReference<BitmapWorkerTask> bitmapWorkerTaskReference;

    public AsyncDrawable(Resources res, Bitmap bitmap,
            BitmapWorkerTask bitmapWorkerTask) {
        super(res, bitmap);
        bitmapWorkerTaskReference =
            new WeakReference<BitmapWorkerTask>(bitmapWorkerTask);
    }

    public BitmapWorkerTask getBitmapWorkerTask() {
        return bitmapWorkerTaskReference.get();
    }
}
{% endhighlight %}

Before executing the BitmapWorkerTask, you create an AsyncDrawable and bind it to the target ImageView:

{% highlight ruby %}
public void loadBitmap(int resId, ImageView imageView) {
    if (cancelPotentialWork(resId, imageView)) {
        final BitmapWorkerTask task = new BitmapWorkerTask(imageView);
        final AsyncDrawable asyncDrawable =
                new AsyncDrawable(getResources(), mPlaceHolderBitmap, task);
        imageView.setImageDrawable(asyncDrawable);
        task.execute(resId);
    }
}
{% endhighlight %}

The cancelPotentialWork method referenced in the code sample above checks if another running task is already associated with the ImageView.
If so, it attempts to cancel the previous task by calling cancel(). In a small number of cases, the new task data matches the existing task
and nothing further needs to happen. Here is the implementation of cancelPotentialWork:

{% highlight ruby %}
public static boolean cancelPotentialWork(int data, ImageView imageView) {
    final BitmapWorkerTask bitmapWorkerTask = getBitmapWorkerTask(imageView);

    if (bitmapWorkerTask != null) {
        final int bitmapData = bitmapWorkerTask.data;
        // If bitmapData is not yet set or it differs from the new data
        if (bitmapData == 0 || bitmapData != data) {
            // Cancel previous task
            bitmapWorkerTask.cancel(true);
        } else {
            // The same work is already in progress
            return false;
        }
    }
    // No task associated with the ImageView, or an existing task was cancelled
    return true;
}
{% endhighlight %}

A helper method, getBitmapWorkerTask(), is used above to retrieve the task associated with a particular ImageView:

{% highlight ruby %}
private static BitmapWorkerTask getBitmapWorkerTask(ImageView imageView) {
   if (imageView != null) {
       final Drawable drawable = imageView.getDrawable();
       if (drawable instanceof AsyncDrawable) {
           final AsyncDrawable asyncDrawable = (AsyncDrawable) drawable;
           return asyncDrawable.getBitmapWorkerTask();
       }
    }
    return null;
}
{% endhighlight %}

The last step is updating onPostExecute() in BitmapWorkerTask so that it checks if the task is cancelled and if the current task matches the one associated with the ImageView:

{% highlight ruby %}
class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {
    ...

    @Override
    protected void onPostExecute(Bitmap bitmap) {
        if (isCancelled()) {
            bitmap = null;
        }

        if (imageViewReference != null && bitmap != null) {
            final ImageView imageView = imageViewReference.get();
            final BitmapWorkerTask bitmapWorkerTask =
                    getBitmapWorkerTask(imageView);
            if (this == bitmapWorkerTask && imageView != null) {
                imageView.setImageBitmap(bitmap);
            }
        }
    }
}
{% endhighlight %}

This implementation is now suitable for use in ListView and GridView components as well as any other components that recycle their child views.
Simply call loadBitmap where you normally set an image to your ImageView.
For example, in a GridView implementation this would be in the getView() method of the backing adapter.

精髓指点：

如上通过重写BitmapDrawable实现AsyncDrawable子类，将BitmapWorkerTask在AsyncDrawable中保存为弱引用。所以在执行BitmapWorkerTask前需要创建AsyncDrawable，然后通过
AsyncDrawable与ImageView绑定，如上第二段代码块loadBitmap方法。

loadBitmap方法调运的cancelPotentialWork方法作用是如果有已经与ImageView关联的另一个正在运行的任务就试图通过调用取消先前任务。如上第三段代码就是cancelPotentialWork
的实现。

getBitmapWorkerTask方法用来获取相同引用的实例task，用在task最后的onPostExecute方法中判断并发的是不是同一个实例。

<hr>

#**Caching Bitmaps**

##**Use a Memory Cache**

原文重点摘要：

In the past, a popular memory cache implementation was a SoftReference or WeakReference bitmap cache, however this is not recommended.
Starting from Android 2.3 (API Level 9) the garbage collector is more aggressive with collecting soft/weak references which makes them fairly ineffective.
In addition, prior to Android 3.0 (API Level 11), the backing data of a bitmap was stored in native memory which is not released in a predictable manner,
potentially causing an application to briefly exceed its memory limits and crash.

In order to choose a suitable size for a LruCache, a number of factors should be taken into consideration, for example:

- How memory intensive is the rest of your activity and/or application?
- How many images will be on-screen at once? How many need to be available ready to come on-screen?
- What is the screen size and density of the device? An extra high density screen (xhdpi) device like Galaxy Nexus will need a larger cache to hold the same number of images in memory compared to a device like Nexus S (hdpi).
- What dimensions and configuration are the bitmaps and therefore how much memory will each take up?
- How frequently will the images be accessed? Will some be accessed more frequently than others? If so,
perhaps you may want to keep certain items always in memory or even have multiple LruCache objects for different groups of bitmaps.
- Can you balance quality against quantity? Sometimes it can be more useful to store a larger number of lower quality bitmaps
potentially loading a higher quality version in another background task.

There is no specific size or formula that suits all applications, it's up to you to analyze your usage and come up with a suitable solution.
A cache that is too small causes additional overhead with no benefit, a cache that is too large can once again cause java.lang.OutOfMemory exceptions
and leave the rest of your app little memory to work with.
 
精髓指点：

现在不推荐使用SoftReference or WeakReference来缓存Bitmap，因为在Android2.3以后的垃圾回收机制更加牛逼了，使得SoftReference or WeakReference几乎无效。
另外，在Android 3.0以前Bitmap使用的是native堆区资源，他的释放时不可预测的。

为了选择合适的LruCache尺寸，有如下一系列因素你需要考虑：

- 你的Activity或者App可用多少内存？
- 屏幕上一次要显示多少图片？
- 你的屏幕尺寸和像素密度是多少？
- 你给Bitmap配置的config和尺寸是多少，他们占用的内存大小是多少？
- Bitmap存取的频率是多少？
- 权衡速度与体验的关系。

没有特殊的值或者公式适合所有App，这些都需要你的代码去自己处理分析计算。

原文重点摘要：

Here’s an example of setting up a LruCache for bitmaps:

{% highlight ruby %}
private LruCache<String, Bitmap> mMemoryCache;

@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    // Get max available VM memory, exceeding this amount will throw an
    // OutOfMemory exception. Stored in kilobytes as LruCache takes an
    // int in its constructor.
    final int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);

    // Use 1/8th of the available memory for this memory cache.
    final int cacheSize = maxMemory / 8;

    mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {
        @Override
        protected int sizeOf(String key, Bitmap bitmap) {
            // The cache size will be measured in kilobytes rather than
            // number of items.
            return bitmap.getByteCount() / 1024;
        }
    };
    ...
}

public void addBitmapToMemoryCache(String key, Bitmap bitmap) {
    if (getBitmapFromMemCache(key) == null) {
        mMemoryCache.put(key, bitmap);
    }
}

public Bitmap getBitmapFromMemCache(String key) {
    return mMemoryCache.get(key);
}
{% endhighlight %}

Note: In this example, one eighth of the application memory is allocated for our cache. On a normal/hdpi device this is a minimum of around 4MB (32/8).
A full screen GridView filled with images on a device with 800x480 resolution would use around 1.5MB (800*480*4 bytes),
so this would cache a minimum of around 2.5 pages of images in memory.

When loading a bitmap into an ImageView, the LruCache is checked first. If an entry is found, it is used immediately to update the ImageView,
otherwise a background thread is spawned to process the image:

{% highlight ruby %}
public void loadBitmap(int resId, ImageView imageView) {
    final String imageKey = String.valueOf(resId);

    final Bitmap bitmap = getBitmapFromMemCache(imageKey);
    if (bitmap != null) {
        mImageView.setImageBitmap(bitmap);
    } else {
        mImageView.setImageResource(R.drawable.image_placeholder);
        BitmapWorkerTask task = new BitmapWorkerTask(mImageView);
        task.execute(resId);
    }
}
{% endhighlight %}

The BitmapWorkerTask also needs to be updated to add entries to the memory cache:

{% highlight ruby %}
class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {
    ...
    // Decode image in background.
    @Override
    protected Bitmap doInBackground(Integer... params) {
        final Bitmap bitmap = decodeSampledBitmapFromResource(
                getResources(), params[0], 100, 100));
        addBitmapToMemoryCache(String.valueOf(params[0]), bitmap);
        return bitmap;
    }
    ...
}
{% endhighlight %}

精髓指点：

如上代码就是一个LruCache内存缓存的简单实现。

##**Use a Disk Cache**

原文重点摘要：

{% highlight ruby %}
private DiskLruCache mDiskLruCache;
private final Object mDiskCacheLock = new Object();
private boolean mDiskCacheStarting = true;
private static final int DISK_CACHE_SIZE = 1024 * 1024 * 10; // 10MB
private static final String DISK_CACHE_SUBDIR = "thumbnails";

@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    // Initialize memory cache
    ...
    // Initialize disk cache on background thread
    File cacheDir = getDiskCacheDir(this, DISK_CACHE_SUBDIR);
    new InitDiskCacheTask().execute(cacheDir);
    ...
}

class InitDiskCacheTask extends AsyncTask<File, Void, Void> {
    @Override
    protected Void doInBackground(File... params) {
        synchronized (mDiskCacheLock) {
            File cacheDir = params[0];
            mDiskLruCache = DiskLruCache.open(cacheDir, DISK_CACHE_SIZE);
            mDiskCacheStarting = false; // Finished initialization
            mDiskCacheLock.notifyAll(); // Wake any waiting threads
        }
        return null;
    }
}

class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {
    ...
    // Decode image in background.
    @Override
    protected Bitmap doInBackground(Integer... params) {
        final String imageKey = String.valueOf(params[0]);

        // Check disk cache in background thread
        Bitmap bitmap = getBitmapFromDiskCache(imageKey);

        if (bitmap == null) { // Not found in disk cache
            // Process as normal
            final Bitmap bitmap = decodeSampledBitmapFromResource(
                    getResources(), params[0], 100, 100));
        }

        // Add final bitmap to caches
        addBitmapToCache(imageKey, bitmap);

        return bitmap;
    }
    ...
}

public void addBitmapToCache(String key, Bitmap bitmap) {
    // Add to memory cache as before
    if (getBitmapFromMemCache(key) == null) {
        mMemoryCache.put(key, bitmap);
    }

    // Also add to disk cache
    synchronized (mDiskCacheLock) {
        if (mDiskLruCache != null && mDiskLruCache.get(key) == null) {
            mDiskLruCache.put(key, bitmap);
        }
    }
}

public Bitmap getBitmapFromDiskCache(String key) {
    synchronized (mDiskCacheLock) {
        // Wait while disk cache is started from background thread
        while (mDiskCacheStarting) {
            try {
                mDiskCacheLock.wait();
            } catch (InterruptedException e) {}
        }
        if (mDiskLruCache != null) {
            return mDiskLruCache.get(key);
        }
    }
    return null;
}

// Creates a unique subdirectory of the designated app cache directory. Tries to use external
// but if not mounted, falls back on internal storage.
public static File getDiskCacheDir(Context context, String uniqueName) {
    // Check if media is mounted or storage is built-in, if so, try and use external cache dir
    // otherwise use internal cache dir
    final String cachePath =
            Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState()) ||
                    !isExternalStorageRemovable() ? getExternalCacheDir(context).getPath() :
                            context.getCacheDir().getPath();

    return new File(cachePath + File.separator + uniqueName);
}
{% endhighlight %}

While the memory cache is checked in the UI thread, the disk cache is checked in the background thread. Disk operations should never take place on the UI thread.
When image processing is complete, the final bitmap is added to both the memory and disk cache for future use.

精髓指点：

如上就是使用DiskLruCache类实现的磁盘缓存。

内存缓存在UI线程处理，磁盘缓存是在后台线程处理的。磁盘操作不应该在UI线程上进行。当图像处理完成后，最终的位图被添加到了内存和磁盘高速缓存。

##**Handle Configuration Changes**

原文重点摘要：

Runtime configuration changes, such as a screen orientation change, cause Android to destroy and restart the running activity with the new configuration
(For more information about this behavior, see Handling Runtime Changes). You want to avoid having to process all your images again so the user has a
smooth and fast experience when a configuration change occurs.

Luckily, you have a nice memory cache of bitmaps that you built in the Use a Memory Cache section. This cache can be passed through to the new activity
instance using a Fragment which is preserved by calling setRetainInstance(true)). After the activity has been recreated, this retained Fragment is
reattached and you gain access to the existing cache object, allowing images to be quickly fetched and re-populated into the ImageView objects.

Here’s an example of retaining a LruCache object across configuration changes using a Fragment:

{% highlight ruby %}
private LruCache<String, Bitmap> mMemoryCache;

@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    RetainFragment retainFragment =
            RetainFragment.findOrCreateRetainFragment(getFragmentManager());
    mMemoryCache = retainFragment.mRetainedCache;
    if (mMemoryCache == null) {
        mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {
            ... // Initialize cache here as usual
        }
        retainFragment.mRetainedCache = mMemoryCache;
    }
    ...
}

class RetainFragment extends Fragment {
    private static final String TAG = "RetainFragment";
    public LruCache<String, Bitmap> mRetainedCache;

    public RetainFragment() {}

    public static RetainFragment findOrCreateRetainFragment(FragmentManager fm) {
        RetainFragment fragment = (RetainFragment) fm.findFragmentByTag(TAG);
        if (fragment == null) {
            fragment = new RetainFragment();
            fm.beginTransaction().add(fragment, TAG).commit();
        }
        return fragment;
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setRetainInstance(true);
    }
}
{% endhighlight %}

To test this out, try rotating a device both with and without retaining the Fragment.
You should notice little to no lag as the images populate the activity almost instantly from memory when you retain the cache.
Any images not found in the memory cache are hopefully available in the disk cache, if not, they are processed as usual.

精髓指点：

考虑当屏幕旋转等运行时变换时，显示一般都会重加载，包含你的bitmap，所以你需要考虑一种快速加载的办法。
幸运的是，缓存可以通过setRetainInstance(true)传递给新的Activity，在Activity重启后，你可以通过附着的Fragment重新使用已存在的缓存，
这样就可以快速加载到ImageView中了。上面就是一个当配置改变时用Fragment重用已有的缓存的例子。

**切记Fragment的这个牛逼方法：setRetainInstance(true)**
Fragment有一个非常强大的功能——就是可以在Activity重新创建时可以不完全销毁Fragment，以便Fragment可以恢复。
在onCreate()方法中调用setRetainInstance(true/false)方法是最佳位置。当在onCreate()方法中调用了setRetainInstance(true)后，
Fragment恢复时会跳过onCreate()和onDestroy()方法，因此不能在onCreate()中放置一些初始化逻辑，切忌！

<hr>

#**Managing Bitmap Memory**

原文重点摘要：

On Android Android 2.2 (API level 8) and lower, when garbage collection occurs, your app's threads get stopped.
This causes a lag that can degrade performance. Android 2.3 adds concurrent garbage collection,
which means that the memory is reclaimed soon after a bitmap is no longer referenced.
On Android 2.3.3 (API level 10) and lower, the backing pixel data for a bitmap is stored in native memory.
It is separate from the bitmap itself, which is stored in the Dalvik heap. The pixel data in native memory is not released in a predictable manner,
potentially causing an application to briefly exceed its memory limits and crash. As of Android 3.0 (API level 11),
the pixel data is stored on the Dalvik heap along with the associated bitmap.

精髓指点：

在Android 2.2（API等级8）以前，当垃圾回收时应用程序的线程会停止。这会导致一个滞后，降低性能。在Android 2.3增加了并发垃圾回收。

##**Manage Memory on Android 2.3.3 and Lower**

原文重点摘要：

On Android 2.3.3 (API level 10) and lower, using recycle() is recommended.
If you're displaying large amounts of bitmap data in your app, you're likely to run into OutOfMemoryError errors.
The recycle() method allows an app to reclaim memory as soon as possible.

Caution: You should use recycle() only when you are sure that the bitmap is no longer being used.
If you call recycle() and later attempt to draw the bitmap, you will get the error: "Canvas: trying to use a recycled bitmap".

The following code snippet gives an example of calling recycle().
It uses reference counting (in the variables mDisplayRefCount and mCacheRefCount) to track whether a bitmap is currently being displayed or in the cache.
The code recycles the bitmap when these conditions are met:

- The reference count for both mDisplayRefCount and mCacheRefCount is 0.
- The bitmap is not null, and it hasn't been recycled yet.

{% highlight ruby %}
private int mCacheRefCount = 0;
private int mDisplayRefCount = 0;
...
// Notify the drawable that the displayed state has changed.
// Keep a count to determine when the drawable is no longer displayed.
public void setIsDisplayed(boolean isDisplayed) {
    synchronized (this) {
        if (isDisplayed) {
            mDisplayRefCount++;
            mHasBeenDisplayed = true;
        } else {
            mDisplayRefCount--;
        }
    }
    // Check to see if recycle() can be called.
    checkState();
}

// Notify the drawable that the cache state has changed.
// Keep a count to determine when the drawable is no longer being cached.
public void setIsCached(boolean isCached) {
    synchronized (this) {
        if (isCached) {
            mCacheRefCount++;
        } else {
            mCacheRefCount--;
        }
    }
    // Check to see if recycle() can be called.
    checkState();
}

private synchronized void checkState() {
    // If the drawable cache and display ref counts = 0, and this drawable
    // has been displayed, then recycle.
    if (mCacheRefCount <= 0 && mDisplayRefCount <= 0 && mHasBeenDisplayed
            && hasValidBitmap()) {
        getBitmap().recycle();
    }
}

private synchronized boolean hasValidBitmap() {
    Bitmap bitmap = getBitmap();
    return bitmap != null && !bitmap.isRecycled();
}
{% endhighlight %}

精髓指点：

在Android 2.3.3以前推荐使用recycle回收Bitmap。recycle使得Bitmap尽可能快的回收。

特别注意：在你使用recycle时必须确保Bitmap不再使用，如果你尝试回收使用的Bitmap会有异常。

如上代码就是一个通过引用计数计算的回收例子。

##**Manage Memory on Android 3.0 and Higher**

原文重点摘要：

The following snippet demonstrates how an existing bitmap is stored for possible later use in the sample app.
When an app is running on Android 3.0 or higher and a bitmap is evicted from the LruCache, a soft reference to the bitmap is placed in a HashSet,
for possible reuse later with inBitmap:

{% highlight ruby %}
Set<SoftReference<Bitmap>> mReusableBitmaps;
private LruCache<String, BitmapDrawable> mMemoryCache;

// If you're running on Honeycomb or newer, create a
// synchronized HashSet of references to reusable bitmaps.
if (Utils.hasHoneycomb()) {
    mReusableBitmaps =
            Collections.synchronizedSet(new HashSet<SoftReference<Bitmap>>());
}

mMemoryCache = new LruCache<String, BitmapDrawable>(mCacheParams.memCacheSize) {

    // Notify the removed entry that is no longer being cached.
    @Override
    protected void entryRemoved(boolean evicted, String key,
            BitmapDrawable oldValue, BitmapDrawable newValue) {
        if (RecyclingBitmapDrawable.class.isInstance(oldValue)) {
            // The removed entry is a recycling drawable, so notify it
            // that it has been removed from the memory cache.
            ((RecyclingBitmapDrawable) oldValue).setIsCached(false);
        } else {
            // The removed entry is a standard BitmapDrawable.
            if (Utils.hasHoneycomb()) {
                // We're running on Honeycomb or later, so add the bitmap
                // to a SoftReference set for possible use with inBitmap later.
                mReusableBitmaps.add
                        (new SoftReference<Bitmap>(oldValue.getBitmap()));
            }
        }
    }
....
}
{% endhighlight %}

In the running app, decoder methods check to see if there is an existing bitmap they can use. For example:

{% highlight ruby %}
public static Bitmap decodeSampledBitmapFromFile(String filename,
        int reqWidth, int reqHeight, ImageCache cache) {

    final BitmapFactory.Options options = new BitmapFactory.Options();
    ...
    BitmapFactory.decodeFile(filename, options);
    ...

    // If we're running on Honeycomb or newer, try to use inBitmap.
    if (Utils.hasHoneycomb()) {
        addInBitmapOptions(options, cache);
    }
    ...
    return BitmapFactory.decodeFile(filename, options);
}
{% endhighlight %}

The next snippet shows the addInBitmapOptions() method that is called in the above snippet.
It looks for an existing bitmap to set as the value for inBitmap.
Note that this method only sets a value for inBitmap if it finds a suitable match (your code should never assume that a match will be found):

{% highlight ruby %}
private static void addInBitmapOptions(BitmapFactory.Options options,
        ImageCache cache) {
    // inBitmap only works with mutable bitmaps, so force the decoder to
    // return mutable bitmaps.
    options.inMutable = true;

    if (cache != null) {
        // Try to find a bitmap to use for inBitmap.
        Bitmap inBitmap = cache.getBitmapFromReusableSet(options);

        if (inBitmap != null) {
            // If a suitable bitmap has been found, set it as the value of
            // inBitmap.
            options.inBitmap = inBitmap;
        }
    }
}

// This method iterates through the reusable bitmaps, looking for one 
// to use for inBitmap:
protected Bitmap getBitmapFromReusableSet(BitmapFactory.Options options) {
        Bitmap bitmap = null;

    if (mReusableBitmaps != null && !mReusableBitmaps.isEmpty()) {
        synchronized (mReusableBitmaps) {
            final Iterator<SoftReference<Bitmap>> iterator
                    = mReusableBitmaps.iterator();
            Bitmap item;

            while (iterator.hasNext()) {
                item = iterator.next().get();

                if (null != item && item.isMutable()) {
                    // Check to see it the item can be used for inBitmap.
                    if (canUseForInBitmap(item, options)) {
                        bitmap = item;

                        // Remove from reusable set so it can't be used again.
                        iterator.remove();
                        break;
                    }
                } else {
                    // Remove from the set if the reference has been cleared.
                    iterator.remove();
                }
            }
        }
    }
    return bitmap;
}
{% endhighlight %}

Finally, this method determines whether a candidate bitmap satisfies the size criteria to be used for inBitmap:

{% highlight ruby %}
static boolean canUseForInBitmap(
        Bitmap candidate, BitmapFactory.Options targetOptions) {

    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
        // From Android 4.4 (KitKat) onward we can re-use if the byte size of
        // the new bitmap is smaller than the reusable bitmap candidate
        // allocation byte count.
        int width = targetOptions.outWidth / targetOptions.inSampleSize;
        int height = targetOptions.outHeight / targetOptions.inSampleSize;
        int byteCount = width * height * getBytesPerPixel(candidate.getConfig());
        return byteCount <= candidate.getAllocationByteCount();
    }

    // On earlier versions, the dimensions must match exactly and the inSampleSize must be 1
    return candidate.getWidth() == targetOptions.outWidth
            && candidate.getHeight() == targetOptions.outHeight
            && targetOptions.inSampleSize == 1;
}

/**
 * A helper function to return the byte usage per pixel of a bitmap based on its configuration.
 */
static int getBytesPerPixel(Config config) {
    if (config == Config.ARGB_8888) {
        return 4;
    } else if (config == Config.RGB_565) {
        return 2;
    } else if (config == Config.ARGB_4444) {
        return 2;
    } else if (config == Config.ALPHA_8) {
        return 1;
    }
    return 1;
}
{% endhighlight %}

精髓指点：

如上就是高版本的Bitmap的处理方式。

<hr>

#**Displaying Bitmaps in Your UI**

原文重点摘要：

This lesson brings together everything from previous lessons,
showing you how to load multiple bitmaps into ViewPager and GridView components using a background thread and bitmap cache,
while dealing with concurrency and configuration changes.

精髓指点：

这一节是综合实战，讲述了ViewPager and GridView的图片显示，还有config发生变化的处理。

##**Load Bitmaps into a ViewPager Implementation**

原文重点摘要：

The swipe view pattern is an excellent way to navigate the detail view of an image gallery.
You can implement this pattern using a ViewPager component backed by a PagerAdapter.
However, a more suitable backing adapter is the subclass FragmentStatePagerAdapter which automatically destroys and
saves state of the Fragments in the ViewPager as they disappear off-screen, keeping memory usage down.

Note: If you have a smaller number of images and are confident they all fit within the application memory limit,
then using a regular PagerAdapter or FragmentPagerAdapter might be more appropriate.

Here’s an implementation of a ViewPager with ImageView children. The main activity holds the ViewPager and the adapter:

{% highlight ruby %}
public class ImageDetailActivity extends FragmentActivity {
    public static final String EXTRA_IMAGE = "extra_image";

    private ImagePagerAdapter mAdapter;
    private ViewPager mPager;

    // A static dataset to back the ViewPager adapter
    public final static Integer[] imageResIds = new Integer[] {
            R.drawable.sample_image_1, R.drawable.sample_image_2, R.drawable.sample_image_3,
            R.drawable.sample_image_4, R.drawable.sample_image_5, R.drawable.sample_image_6,
            R.drawable.sample_image_7, R.drawable.sample_image_8, R.drawable.sample_image_9};

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.image_detail_pager); // Contains just a ViewPager

        mAdapter = new ImagePagerAdapter(getSupportFragmentManager(), imageResIds.length);
        mPager = (ViewPager) findViewById(R.id.pager);
        mPager.setAdapter(mAdapter);
    }

    public static class ImagePagerAdapter extends FragmentStatePagerAdapter {
        private final int mSize;

        public ImagePagerAdapter(FragmentManager fm, int size) {
            super(fm);
            mSize = size;
        }

        @Override
        public int getCount() {
            return mSize;
        }

        @Override
        public Fragment getItem(int position) {
            return ImageDetailFragment.newInstance(position);
        }
    }
}
{% endhighlight %}

Here is an implementation of the details Fragment which holds the ImageView children.
This might seem like a perfectly reasonable approach, but can you see the drawbacks of this implementation? How could it be improved?

{% highlight ruby %}
public class ImageDetailFragment extends Fragment {
    private static final String IMAGE_DATA_EXTRA = "resId";
    private int mImageNum;
    private ImageView mImageView;

    static ImageDetailFragment newInstance(int imageNum) {
        final ImageDetailFragment f = new ImageDetailFragment();
        final Bundle args = new Bundle();
        args.putInt(IMAGE_DATA_EXTRA, imageNum);
        f.setArguments(args);
        return f;
    }

    // Empty constructor, required as per Fragment docs
    public ImageDetailFragment() {}

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mImageNum = getArguments() != null ? getArguments().getInt(IMAGE_DATA_EXTRA) : -1;
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
            Bundle savedInstanceState) {
        // image_detail_fragment.xml contains just an ImageView
        final View v = inflater.inflate(R.layout.image_detail_fragment, container, false);
        mImageView = (ImageView) v.findViewById(R.id.imageView);
        return v;
    }

    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        final int resId = ImageDetailActivity.imageResIds[mImageNum];
        mImageView.setImageResource(resId); // Load image into ImageView
    }
}
{% endhighlight %}

Hopefully you noticed the issue: the images are being read from resources on the UI thread,
which can lead to an application hanging and being force closed. Using an AsyncTask as described in the Processing Bitmaps Off the UI Thread lesson,
it’s straightforward to move image loading and processing to a background thread:

{% highlight ruby %}
public class ImageDetailActivity extends FragmentActivity {
    ...

    public void loadBitmap(int resId, ImageView imageView) {
        mImageView.setImageResource(R.drawable.image_placeholder);
        BitmapWorkerTask task = new BitmapWorkerTask(mImageView);
        task.execute(resId);
    }

    ... // include BitmapWorkerTask class
}

public class ImageDetailFragment extends Fragment {
    ...

    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        if (ImageDetailActivity.class.isInstance(getActivity())) {
            final int resId = ImageDetailActivity.imageResIds[mImageNum];
            // Call out to ImageDetailActivity to load the bitmap in a background thread
            ((ImageDetailActivity) getActivity()).loadBitmap(resId, mImageView);
        }
    }
}
{% endhighlight %}

Any additional processing (such as resizing or fetching images from the network) can take place in the BitmapWorkerTask without
affecting responsiveness of the main UI. If the background thread is doing more than just loading an image directly from disk,
it can also be beneficial to add a memory and/or disk cache as described in the lesson Caching Bitmaps.
Here's the additional modifications for a memory cache:

{% highlight ruby %}
public class ImageDetailActivity extends FragmentActivity {
    ...
    private LruCache<String, Bitmap> mMemoryCache;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        ...
        // initialize LruCache as per Use a Memory Cache section
    }

    public void loadBitmap(int resId, ImageView imageView) {
        final String imageKey = String.valueOf(resId);

        final Bitmap bitmap = mMemoryCache.get(imageKey);
        if (bitmap != null) {
            mImageView.setImageBitmap(bitmap);
        } else {
            mImageView.setImageResource(R.drawable.image_placeholder);
            BitmapWorkerTask task = new BitmapWorkerTask(mImageView);
            task.execute(resId);
        }
    }

    ... // include updated BitmapWorkerTask from Use a Memory Cache section
}
{% endhighlight %}

Putting all these pieces together gives you a responsive ViewPager implementation with minimal image loading latency and the ability
to do as much or as little background processing on your images as needed.

精髓指点：

在ViewPager使用时，若果图片较多推荐使用FragmentStatePagerAdapter，较少且固定推荐使用FragmentPagerAdapter。

如上第一段代码就是一个简单的例子。
第二段代码是ViewPager内部每个item的Fragment实现例子。
第三段代码是将ImageView挪动到异步加载处理模式的例子。
第四段代码是带缓存的加载例子。

##**Load Bitmaps into a GridView Implementation**

原文重点摘要：

The grid list building block is useful for showing image data sets and can be implemented using a GridView
component in which many images can be on-screen at any one time and many more need to be ready to appear if the user scrolls up or down.
When implementing this type of control, you must ensure the UI remains fluid, memory usage remains under control and concurrency is
handled correctly (due to the way GridView recycles its children views).

To start with, here is a standard GridView implementation with ImageView children placed inside a Fragment.
Again, this might seem like a perfectly reasonable approach, but what would make it better?

{% highlight ruby %}
public class ImageGridFragment extends Fragment implements AdapterView.OnItemClickListener {
    private ImageAdapter mAdapter;

    // A static dataset to back the GridView adapter
    public final static Integer[] imageResIds = new Integer[] {
            R.drawable.sample_image_1, R.drawable.sample_image_2, R.drawable.sample_image_3,
            R.drawable.sample_image_4, R.drawable.sample_image_5, R.drawable.sample_image_6,
            R.drawable.sample_image_7, R.drawable.sample_image_8, R.drawable.sample_image_9};

    // Empty constructor as per Fragment docs
    public ImageGridFragment() {}

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mAdapter = new ImageAdapter(getActivity());
    }

    @Override
    public View onCreateView(
            LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        final View v = inflater.inflate(R.layout.image_grid_fragment, container, false);
        final GridView mGridView = (GridView) v.findViewById(R.id.gridView);
        mGridView.setAdapter(mAdapter);
        mGridView.setOnItemClickListener(this);
        return v;
    }

    @Override
    public void onItemClick(AdapterView<?> parent, View v, int position, long id) {
        final Intent i = new Intent(getActivity(), ImageDetailActivity.class);
        i.putExtra(ImageDetailActivity.EXTRA_IMAGE, position);
        startActivity(i);
    }

    private class ImageAdapter extends BaseAdapter {
        private final Context mContext;

        public ImageAdapter(Context context) {
            super();
            mContext = context;
        }

        @Override
        public int getCount() {
            return imageResIds.length;
        }

        @Override
        public Object getItem(int position) {
            return imageResIds[position];
        }

        @Override
        public long getItemId(int position) {
            return position;
        }

        @Override
        public View getView(int position, View convertView, ViewGroup container) {
            ImageView imageView;
            if (convertView == null) { // if it's not recycled, initialize some attributes
                imageView = new ImageView(mContext);
                imageView.setScaleType(ImageView.ScaleType.CENTER_CROP);
                imageView.setLayoutParams(new GridView.LayoutParams(
                        LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT));
            } else {
                imageView = (ImageView) convertView;
            }
            imageView.setImageResource(imageResIds[position]); // Load image into ImageView
            return imageView;
        }
    }
}
{% endhighlight %}

Once again, the problem with this implementation is that the image is being set in the UI thread.
While this may work for small, simple images (due to system resource loading and caching),
if any additional processing needs to be done, your UI grinds to a halt.

The same asynchronous processing and caching methods from the previous section can be implemented here.
However, you also need to wary of concurrency issues as the GridView recycles its children views.
To handle this, use the techniques discussed in the Processing Bitmaps Off the UI Thread lesson.
Here is the updated solution:

{% highlight ruby %}
public class ImageGridFragment extends Fragment implements AdapterView.OnItemClickListener {
    ...

    private class ImageAdapter extends BaseAdapter {
        ...

        @Override
        public View getView(int position, View convertView, ViewGroup container) {
            ...
            loadBitmap(imageResIds[position], imageView)
            return imageView;
        }
    }

    public void loadBitmap(int resId, ImageView imageView) {
        if (cancelPotentialWork(resId, imageView)) {
            final BitmapWorkerTask task = new BitmapWorkerTask(imageView);
            final AsyncDrawable asyncDrawable =
                    new AsyncDrawable(getResources(), mPlaceHolderBitmap, task);
            imageView.setImageDrawable(asyncDrawable);
            task.execute(resId);
        }
    }

    static class AsyncDrawable extends BitmapDrawable {
        private final WeakReference<BitmapWorkerTask> bitmapWorkerTaskReference;

        public AsyncDrawable(Resources res, Bitmap bitmap,
                BitmapWorkerTask bitmapWorkerTask) {
            super(res, bitmap);
            bitmapWorkerTaskReference =
                new WeakReference<BitmapWorkerTask>(bitmapWorkerTask);
        }

        public BitmapWorkerTask getBitmapWorkerTask() {
            return bitmapWorkerTaskReference.get();
        }
    }

    public static boolean cancelPotentialWork(int data, ImageView imageView) {
        final BitmapWorkerTask bitmapWorkerTask = getBitmapWorkerTask(imageView);

        if (bitmapWorkerTask != null) {
            final int bitmapData = bitmapWorkerTask.data;
            if (bitmapData != data) {
                // Cancel previous task
                bitmapWorkerTask.cancel(true);
            } else {
                // The same work is already in progress
                return false;
            }
        }
        // No task associated with the ImageView, or an existing task was cancelled
        return true;
    }

    private static BitmapWorkerTask getBitmapWorkerTask(ImageView imageView) {
       if (imageView != null) {
           final Drawable drawable = imageView.getDrawable();
           if (drawable instanceof AsyncDrawable) {
               final AsyncDrawable asyncDrawable = (AsyncDrawable) drawable;
               return asyncDrawable.getBitmapWorkerTask();
           }
        }
        return null;
    }

    ... // include updated BitmapWorkerTask class
{% endhighlight %}

Note: The same code can easily be adapted to work with ListView as well.

This implementation allows for flexibility in how the images are processed and loaded without impeding the smoothness of the UI.
In the background task you can load images from the network or resize large digital camera photos and the images appear as the tasks finish processing.

For a full example of this and other concepts discussed in this lesson, please see the included sample application.

精髓指点：

如上第一段代码就是实现了Fragment里显示GridView，GridView的item均为Image的例子。
第二段代码展示了GridView异步加载图片的方式。

如上的例子在Listview等也是类比实现的。可以好好琢磨下官方的这种实现方式。

