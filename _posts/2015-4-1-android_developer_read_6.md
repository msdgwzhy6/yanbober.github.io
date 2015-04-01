---
layout: post
keywords: Android Developer
description: Android Developer读书笔记
title: "[Training] Capturing Photos"
categories: [开发文档]
tags: [Android读书笔记]
group: archive
icon: file-alt
---
{% include site/setup %}
<hr>
重读Android Developer笔记核心记录
<hr>

#**Taking Photos Simply**

##**Request Camera Permission**

原文重点摘要：

{% highlight ruby %}
<manifest ... >
    <uses-feature android:name="android.hardware.camera"
                  android:required="true" />
    ...
</manifest>
{% endhighlight %}

If your application uses, but does not require a camera in order to function, instead set android:required to false.
In doing so, Google Play will allow devices without a camera to download your application.
It's then your responsibility to check for the availability of the camera at runtime by calling hasSystemFeature(PackageManager.FEATURE_CAMERA).
If a camera is not available, you should then disable your camera features.

精髓指点：

和NFC那一讲差不多，需要声明权限，而且android:required设置同样类似于NFC，这个设置关系到你的应用在market是否展示。

##**Take a Photo with the Camera App**

原文重点摘要：

{% highlight ruby %}
static final int REQUEST_IMAGE_CAPTURE = 1;

private void dispatchTakePictureIntent() {
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
        startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);
    }
}
{% endhighlight %}

Notice that the startActivityForResult() method is protected by a condition that calls resolveActivity(),
which returns the first activity component that can handle the intent.
Performing this check is important because if you call startActivityForResult() using an intent that no app can handle,
your app will crash. So as long as the result is not null, it's safe to use the intent.

精髓指点：

注意，上面代码startActivityForResult（）方法是通过调用resolveActivity条件保护（），执行这项检查是很重要的，
因为如果你使用的是intent调用startActivityForResult（）没有应用程序可以处理，您的应用程序会崩溃。所以只要结果不为空，它才是安全的使用意图。

疑惑：和[Verify There is an App to Receive the Intent](http://yanbober.github.io/2015/03/30/android_developer_read_3/)区别在哪呢？

##**Get the Thumbnail**

原文重点摘要：

{% highlight ruby %}
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
        Bundle extras = data.getExtras();
        Bitmap imageBitmap = (Bitmap) extras.get("data");
        mImageView.setImageBitmap(imageBitmap);
    }
}
{% endhighlight %}

Note: This thumbnail image from "data" might be good for an icon, but not a lot more. Dealing with a full-sized image takes a bit more work.

精髓指点：

注意：此从“data”得到的是缩略图适合做图标。你如果需要用全尺寸的图像处理需要一些更多的工作。

##**Save the Full-size Photo**

原文重点摘要：

The Android Camera application saves a full-size photo if you give it a file to save into.
You must provide a fully qualified file name where the camera app should save the photo.

Generally, any photos that the user captures with the device camera should be saved on the device in the public external storage so they are accessible by all apps.
The proper directory for shared photos is provided by getExternalStoragePublicDirectory(), with the DIRECTORY_PICTURES argument.
Because the directory provided by this method is shared among all apps,
reading and writing to it requires the READ_EXTERNAL_STORAGE and WRITE_EXTERNAL_STORAGE permissions, respectively.
The write permission implicitly allows reading, so if you need to write to the external storage then you need to request only one permission:

{% highlight ruby %}
<manifest ...>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    ...
</manifest>
{% endhighlight %}

However, if you'd like the photos to remain private to your app only, you can instead use the directory provided by getExternalFilesDir().
On Android 4.3 and lower, writing to this directory also requires the WRITE_EXTERNAL_STORAGE permission.
Beginning with Android 4.4, the permission is no longer required because the directory is not accessible by other apps,
so you can declare the permission should be requested only on the lower versions of Android by adding the maxSdkVersion attribute:

{% highlight ruby %}
<manifest ...>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
                     android:maxSdkVersion="18" />
    ...
</manifest>
{% endhighlight %}

Once you decide the directory for the file, you need to create a collision-resistant file name.
You may wish also to save the path in a member variable for later use.
Here's an example solution in a method that returns a unique file name for a new photo using a date-time stamp:

{% highlight ruby %}
String mCurrentPhotoPath;

private File createImageFile() throws IOException {
    // Create an image file name
    String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
    String imageFileName = "JPEG_" + timeStamp + "_";
    File storageDir = Environment.getExternalStoragePublicDirectory(
            Environment.DIRECTORY_PICTURES);
    File image = File.createTempFile(
        imageFileName,  /* prefix */
        ".jpg",         /* suffix */
        storageDir      /* directory */
    );

    // Save a file: path for use with ACTION_VIEW intents
    mCurrentPhotoPath = "file:" + image.getAbsolutePath();
    return image;
}
{% endhighlight %}

With this method available to create a file for the photo, you can now create and invoke the Intent like this:

{% highlight ruby %}
static final int REQUEST_TAKE_PHOTO = 1;

private void dispatchTakePictureIntent() {
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    // Ensure that there's a camera activity to handle the intent
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
        // Create the File where the photo should go
        File photoFile = null;
        try {
            photoFile = createImageFile();
        } catch (IOException ex) {
            // Error occurred while creating the File
            ...
        }
        // Continue only if the File was successfully created
        if (photoFile != null) {
            takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT,
                    Uri.fromFile(photoFile));
            startActivityForResult(takePictureIntent, REQUEST_TAKE_PHOTO);
        }
    }
}
{% endhighlight %}

精髓指点：

如果你指定了全路径的文件名，那就可以存储全尺寸的图片。

通常照相的照片存储在外部存储空间，以便所有App能够访问。适当的外部存储路径通过调运getExternalStoragePublicDirectory的DIRECTORY_PICTURES参数。

你需要如上代码段一样声明权限。

如果你想将照片存为私有，可以像以前的教程里数据存储说的使用getExternalFilesDir。权限也可以像上面那样声明，也可以参考数据存储那块讲的。
这种存储你拍的照片会在App卸载时自动删掉。

一旦你的路径确定了，你需要创建一个没有冲突的文件名，上面第三段代码就是教你如何创建一个带有时间戳的文件名的。

如果像上面一样文件创建好了，那你就可以像上面最后一段代码一样创建请求照相获取全尺寸的intent。

##**Add the Photo to a Gallery**

原文重点摘要：

Note: If you saved your photo to the directory provided by getExternalFilesDir(), the media scanner cannot access the files because they are private to your app.

精髓指点：

特别注意！！！！

如果你使用getExternalFilesDir存储的照片，那么media scanner不会扫描到你的照片的，因为他是你APP私有的数据。切记！！！！

原文重点摘要：

The following example method demonstrates how to invoke the system's media scanner to add your photo to the Media Provider's database,
making it available in the Android Gallery application and to other apps.

{% highlight ruby %}
private void galleryAddPic() {
    Intent mediaScanIntent = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
    File f = new File(mCurrentPhotoPath);
    Uri contentUri = Uri.fromFile(f);
    mediaScanIntent.setData(contentUri);
    this.sendBroadcast(mediaScanIntent);
}
{% endhighlight %}

精髓指点：

如上例子展示了如何调运media scanner添加你的照片到Media Provider数据库。以便于你的照片可以顺利被gallery等其他App（读取Media Provider数据库）使用。

##**Decode a Scaled Image**

原文重点摘要：

Managing multiple full-sized images can be tricky with limited memory.
If you find your application running out of memory after displaying just a few images,
you can dramatically reduce the amount of dynamic heap used by expanding the JPEG into a memory array that's already scaled to match the size of the destination view.
The following example method demonstrates this technique.

{% highlight ruby %}
private void setPic() {
    // Get the dimensions of the View
    int targetW = mImageView.getWidth();
    int targetH = mImageView.getHeight();

    // Get the dimensions of the bitmap
    BitmapFactory.Options bmOptions = new BitmapFactory.Options();
    bmOptions.inJustDecodeBounds = true;
    BitmapFactory.decodeFile(mCurrentPhotoPath, bmOptions);
    int photoW = bmOptions.outWidth;
    int photoH = bmOptions.outHeight;

    // Determine how much to scale down the image
    int scaleFactor = Math.min(photoW/targetW, photoH/targetH);

    // Decode the image file into a Bitmap sized to fill the View
    bmOptions.inJustDecodeBounds = false;
    bmOptions.inSampleSize = scaleFactor;
    bmOptions.inPurgeable = true;

    Bitmap bitmap = BitmapFactory.decodeFile(mCurrentPhotoPath, bmOptions);
    mImageView.setImageBitmap(bitmap);
}
{% endhighlight %}

精髓指点：

管理多个全尺寸的图像在内存限制上可能会非常棘手。如果您发现运行应用程序显示显示一点图片后内存不足，
你可以动态控制图像维度来匹配显示尺寸。上面的例子演示了这种技术。

<hr>

#**Recording Videos Simply**

##**Request Camera Permission**

原文重点摘要：

{% highlight ruby %}
<manifest ... >
    <uses-feature android:name="android.hardware.camera"
                  android:required="true" />
    ...
</manifest>
{% endhighlight %}

精髓指点：

同上照相机一样先声明权限，一样的注意事项，参见文首。

##**Record a Video with a Camera App**

原文重点摘要：

{% highlight ruby %}
static final int REQUEST_VIDEO_CAPTURE = 1;

private void dispatchTakeVideoIntent() {
    Intent takeVideoIntent = new Intent(MediaStore.ACTION_VIDEO_CAPTURE);
    if (takeVideoIntent.resolveActivity(getPackageManager()) != null) {
        startActivityForResult(takeVideoIntent, REQUEST_VIDEO_CAPTURE);
    }
}
{% endhighlight %}

精髓指点：

类似照相机一样，通过intent启动摄像机。不解释。

##**View the Video**

原文重点摘要：

The Android Camera application returns the video in the Intent delivered to onActivityResult() as a Uri pointing to the video location in storage.
The following code retrieves this video and displays it in a VideoView.

{% highlight ruby %}
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (requestCode == REQUEST_VIDEO_CAPTURE && resultCode == RESULT_OK) {
        Uri videoUri = intent.getData();
        mVideoView.setVideoURI(videoUri);
    }
}
{% endhighlight %}

精髓指点：

Android返回请求的video通过onActivityResult，返回的是video的URI，而不是video的数据。如上代码演示了获取URI并且显示在VideoView播放的过程。

<hr>

#**Controlling the Camera**

原文重点摘要：

if you want to build a specialized camera application or something fully integrated in your app UI, this lesson shows you how.

精髓指点：

如果你想开发一个专业的相机app或者自定义的UI等，那这节课就会展示给你如何调运相机的硬件API操作。

##**Open the Camera Object**

原文重点摘要：

Getting an instance of the Camera object is the first step in the process of directly controlling the camera.
As Android's own Camera application does, the recommended way to access the camera is to open Camera on a separate thread that's launched from onCreate().
This approach is a good idea since it can take a while and might bog down the UI thread. In a more basic implementation,
opening the camera can be deferred to the onResume() method to facilitate code reuse and keep the flow of control simple.

Calling Camera.open() throws an exception if the camera is already in use by another application, so we wrap it in a try block.

{% highlight ruby %}
private boolean safeCameraOpen(int id) {
    boolean qOpened = false;
  
    try {
        releaseCameraAndPreview();
        mCamera = Camera.open(id);
        qOpened = (mCamera != null);
    } catch (Exception e) {
        Log.e(getString(R.string.app_name), "failed to open Camera");
        e.printStackTrace();
    }

    return qOpened;    
}

private void releaseCameraAndPreview() {
    mPreview.setCamera(null);
    if (mCamera != null) {
        mCamera.release();
        mCamera = null;
    }
}
{% endhighlight %}

Since API level 9, the camera framework supports multiple cameras. If you use the legacy API and call open() without an argument,
you get the first rear-facing camera.

精髓指点：

想直接控制相机的话，得到相机实例是第一步。就像Android自身相机应用一样，推荐打开相机的操作在一个新的线程里操作，因为他可能导致UI卡死。
打开相机的功能可以放在onResume方法里，方便重用，简单。如上代码展示如何打开相机。

从API 19开始，相机支持多个。如果你使用传统的API，则调用open（）不带参数时默认得到的是第一个后置摄像头。

##**Create the Camera Preview**

原文重点摘要：

Taking a picture usually requires that your users see a preview of their subject before clicking the shutter.
To do so, you can use a SurfaceView to draw previews of what the camera sensor is picking up.

精髓指点：

照相按下快门前都需要预览的，在这里预览采用了SurfaceView来实时绘制你的相机传感器图像。

####**Preview Class**

原文重点摘要：

To get started with displaying a preview, you need preview class. The preview requires an implementation of the android.view.SurfaceHolder.Callback interface,
which is used to pass image data from the camera hardware to the application.

{% highlight ruby %}
class Preview extends ViewGroup implements SurfaceHolder.Callback {

    SurfaceView mSurfaceView;
    SurfaceHolder mHolder;

    Preview(Context context) {
        super(context);

        mSurfaceView = new SurfaceView(context);
        addView(mSurfaceView);

        // Install a SurfaceHolder.Callback so we get notified when the
        // underlying surface is created and destroyed.
        mHolder = mSurfaceView.getHolder();
        mHolder.addCallback(this);
        mHolder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);
    }
...
}
{% endhighlight %}

The preview class must be passed to the Camera object before the live image preview can be started, as shown in the next section.

精髓指点：

想显示预览的话你就需要一个预览类。预览需要实现android.view.SurfaceHolder.Callback接口，这个接口用来传递硬件相机的数据到App。
如上代码所示。预览界面必须在预览图像开始前传递到Camera，就像下文所述。

####**Set and Start the Preview**

原文重点摘要：

A camera instance and its related preview must be created in a specific order, with the camera object being first.
In the snippet below, the process of initializing the camera is encapsulated so that Camera.startPreview() is called by the setCamera() method,
whenever the user does something to change the camera. The preview must also be restarted in the preview class surfaceChanged() callback method.

{% highlight ruby %}
public void setCamera(Camera camera) {
    if (mCamera == camera) { return; }
    
    stopPreviewAndFreeCamera();
    
    mCamera = camera;
    
    if (mCamera != null) {
        List<Size> localSizes = mCamera.getParameters().getSupportedPreviewSizes();
        mSupportedPreviewSizes = localSizes;
        requestLayout();
      
        try {
            mCamera.setPreviewDisplay(mHolder);
        } catch (IOException e) {
            e.printStackTrace();
        }
      
        // Important: Call startPreview() to start updating the preview
        // surface. Preview must be started before you can take a picture.
        mCamera.startPreview();
    }
}
{% endhighlight %}

精髓指点：

相机实例和他相关的预览界面必须按照一定顺序创建。
在上面的代码中，初始化照相机的过程被封装好了，使得Camera.startPreview（）被setCamera（）方法调用，
每当用户做一些事情来改变摄像头。预览也必须重新启动在预览类surfaceChanged（）回调方法。

##**Modify Camera Settings**

原文重点摘要：

Camera settings change the way that the camera takes pictures, from the zoom level to exposure compensation.
This example changes only the preview size; see the source code of the Camera application for many more.

{% highlight ruby %}
public void surfaceChanged(SurfaceHolder holder, int format, int w, int h) {
    // Now that the size is known, set up the camera parameters and begin
    // the preview.
    Camera.Parameters parameters = mCamera.getParameters();
    parameters.setPreviewSize(mPreviewSize.width, mPreviewSize.height);
    requestLayout();
    mCamera.setParameters(parameters);

    // Important: Call startPreview() to start updating the preview surface.
    // Preview must be started before you can take a picture.
    mCamera.startPreview();
}
{% endhighlight %}

精髓指点：

这个例子只是演示了相机缩放预览尺寸的例子。

##**Set the Preview Orientation**

原文重点摘要：

Most camera applications lock the display into landscape mode because that is the natural orientation of the camera sensor.
This setting does not prevent you from taking portrait-mode photos, because the orientation of the device is recorded in the EXIF header.
The setCameraDisplayOrientation() method lets you change how the preview is displayed without affecting how the image is recorded.
However, in Android prior to API level 14, you must stop your preview before changing the orientation and then restart it.

精髓指点：

许多相机APP锁定显示为横屏模式，因为他是相机传感器的自然方向。
此设置不会妨碍你采取纵向模式的照片，因为设备的方向被记录在EXIF的头部。
该setCameraDisplayOrientation（）方法可以更改如何显示预览，而不影响图像是如何记录。
但是在Android API 14以前，你必须在改变方向之前停止预览，然后重新启动它。

##**Take a Picture**

原文重点摘要：

Use the Camera.takePicture() method to take a picture once the preview is started.
You can create Camera.PictureCallback and Camera.ShutterCallback objects and pass them into Camera.takePicture().

If you want to grab images continously, you can create a Camera.PreviewCallback that implements onPreviewFrame().
For something in between, you can capture only selected preview frames, or set up a delayed action to call takePicture().

精髓指点：

一旦预览开始以后你可以使用Camera.takePicture()方法拍照，你可以创建Camera.PictureCallback和Camera.ShutterCallback传递给
Camera.takePicture()。

如果你想连续抓拍，可以通过实现onPreviewFrame创建Camera.PreviewCallback方法。
对于介于两者之间时，你可以捕捉唯一入选的预览画面，或设置延迟作用调用takePicture（）。

##**Restart the Preview**

原文重点摘要：

After a picture is taken, you must restart the preview before the user can take another picture.
In this example, the restart is done by overloading the shutter button.

{% highlight ruby %}
@Override
public void onClick(View v) {
    switch(mPreviewState) {
    case K_STATE_FROZEN:
        mCamera.startPreview();
        mPreviewState = K_STATE_PREVIEW;
        break;

    default:
        mCamera.takePicture( null, rawCallback, null);
        mPreviewState = K_STATE_BUSY;
    } // switch
    shutterBtnConfig();
}
{% endhighlight %}

精髓指点：

拍完一张照片后你需要重置preview才可以拍另一张。

##**Stop the Preview and Release the Camera**

原文重点摘要：

Once your application is done using the camera, it's time to clean up. In particular, you must release the Camera object,
or you risk crashing other applications, including new instances of your own application.

When should you stop the preview and release the camera?
Well, having your preview surface destroyed is a pretty good hint that it’s time to stop the preview and release the camera,
as shown in these methods from the Preview class.

{% highlight ruby %}
public void surfaceDestroyed(SurfaceHolder holder) {
    // Surface will be destroyed when we return, so stop the preview.
    if (mCamera != null) {
        // Call stopPreview() to stop updating the preview surface.
        mCamera.stopPreview();
    }
}

/**
 * When this function returns, mCamera will be null.
 */
private void stopPreviewAndFreeCamera() {

    if (mCamera != null) {
        // Call stopPreview() to stop updating the preview surface.
        mCamera.stopPreview();
    
        // Important: Call release() to release the camera for use by other
        // applications. Applications should release the camera immediately
        // during onPause() and re-open() it during onResume()).
        mCamera.release();
    
        mCamera = null;
    }
}
{% endhighlight %}

Earlier in the lesson, this procedure was also part of the setCamera() method, so initializing a camera always begins with stopping the preview.

精髓指点：

用完相机要释放，尤其是Camera对象，否则就会出现其他应用程序崩溃，包括你自己的应用程序使用新实例也会崩溃。

什么时候停止预览和释放相机呢？

预览面调运surfaceDestroyed时就暗示该销毁预览和相机了，就像上面方法中写的。

前边的代码setCamera中也用到了stopPreviewAndFreeCamera，所以在初始化相机时也需要先停止预览。
