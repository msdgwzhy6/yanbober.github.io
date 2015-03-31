---
layout: post
keywords: Android Developer
description: Android Developer读书笔记
title: "[Training] Building Apps with Content Sharing"
categories: [开发文档]
tags: [Android读书笔记]
group: archive
icon: file-alt
---
{% include site/setup %}
<hr>
重读Android Developer笔记核心记录
<hr>

#**Sharing Simple Data**

原文重点摘要：

The best way to add a share action item to an ActionBar is to use ShareActionProvider, which became available in API level 14.

精髓指点：

最好的分享方式是使用ActionBar的ShareActionProvider，但是他的基本API是14。

##**Send Text Content**

原文重点摘要：

{% highlight ruby %}
Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_SEND);
sendIntent.putExtra(Intent.EXTRA_TEXT, "This is my text to send.");
sendIntent.setType("text/plain");
startActivity(Intent.createChooser(sendIntent, getResources().getText(R.string.send_to)));
{% endhighlight %}

精髓指点：

如果你在ACTION_SEND时使用了createChooser则每次发intent时都会显示选择界面，而如果没有用createChooser则发送时会提醒是否设置默认选项，以后就不
显示选择界面。

##**Send Binary Content**

原文重点摘要：

{% highlight ruby %}
Intent shareIntent = new Intent();
shareIntent.setAction(Intent.ACTION_SEND);
shareIntent.putExtra(Intent.EXTRA_STREAM, uriToImage);
shareIntent.setType("image/jpeg");
startActivity(Intent.createChooser(shareIntent, getResources().getText(R.string.send_to)));
{% endhighlight %}

精髓指点：

接收应用程序需要访问数据Uri需要权限。推荐的方法是:

- 将数据存储在自己的内容提供者,确保其他应用正确的权限访问你的提供者。
提供访问的首选机制是使用per-URI权限只暂时的,接收应用程序的授权访问。
一个简单的方法来创建一个ContentProvider就是使用FileProvider助手类。
- 使用系统MediaStore。MediaStore主要是针对视频、音频和图像的MIME类型,然而从Android 3.0(API级别11)它也可以存储等非媒体类型
(见MediaStore。Filesfor更多信息)。文件可以使用scanFile插入到MediaStore()之后,content://风格的Uri分享传递到提供onScanCompleted()回调。
注意,一旦添加到系统MediaStore设备上的任何应用程序都可以访问内容。

如下是我的一个分享文字和图片实例：

{% highlight ruby %}
    private View.OnClickListener clickListener = new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            Intent getAlbum = new Intent(Intent.ACTION_GET_CONTENT);
            getAlbum.setType("image/*");
            startActivityForResult(getAlbum, 0);
        }
    };

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == 0 && resultCode == RESULT_OK) {
            Uri originalUri = data.getData();
            share(MainActivity.this, "选择分享到：", "我的测试", "这是一个测试字符串！", originalUri);
        }
    }

    public static void share(Context context, String activityTitle, String msgTitle, String msgText, Uri imgPath) {
        Intent intent = new Intent(Intent.ACTION_SEND);
        if (imgPath == null || imgPath.equals("")) {
            intent.setType("text/plain");
        } else {
            if (imgPath != null) {
                intent.setType("image/png");
                intent.putExtra(Intent.EXTRA_STREAM, imgPath);
            }
        }
        intent.putExtra(Intent.EXTRA_SUBJECT, msgTitle);
        intent.putExtra(Intent.EXTRA_TEXT, msgText);
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(Intent.createChooser(intent, activityTitle));
    }
{% endhighlight %}

##**Send Multiple Pieces of Content**

原文重点摘要：

{% highlight ruby %}
ArrayList<Uri> imageUris = new ArrayList<Uri>();
imageUris.add(imageUri1); // Add your image URIs here
imageUris.add(imageUri2);

Intent shareIntent = new Intent();
shareIntent.setAction(Intent.ACTION_SEND_MULTIPLE);
shareIntent.putParcelableArrayListExtra(Intent.EXTRA_STREAM, imageUris);
shareIntent.setType("image/*");
startActivity(Intent.createChooser(shareIntent, "Share images to.."));
{% endhighlight %}

##**Receiving Simple Data from Other Apps**

原文重点摘要：

{% highlight ruby %}
<activity android:name=".ui.MyActivity" >
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="image/*" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/plain" />
    </intent-filter>
    <intent-filter>
        <action android:name="android.intent.action.SEND_MULTIPLE" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="image/*" />
    </intent-filter>
</activity>
{% endhighlight %}

Take extra care to check the incoming data, you never know what some other application may send you.
For example, the wrong MIME type might be set, or the image being sent might be extremely large. Also,
remember to process binary data in a separate thread rather than the main ("UI") thread.

精髓指点：

格外小心检查传入的数据，你永远不会知道其他应用程序发给你的数据会是啥样。例如，错误的MIME类型可能被设置，或发送的图像可能会非常大。
还有，记得在一个单独的线程来处理二进制数据,而不是主(UI)线程。

##**Adding an Easy Share Action**

原文重点摘要：

ShareActionProvider is available starting with API Level 14 and higher.

{% highlight ruby %}
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
            android:id="@+id/menu_item_share"
            android:showAsAction="ifRoom"
            android:title="Share"
            android:actionProviderClass=
                "android.widget.ShareActionProvider" />
    ...
</menu>
{% endhighlight %}

{% highlight ruby %}
private ShareActionProvider mShareActionProvider;
...

@Override
public boolean onCreateOptionsMenu(Menu menu) {
    // Inflate menu resource file.
    getMenuInflater().inflate(R.menu.share_menu, menu);

    // Locate MenuItem with ShareActionProvider
    MenuItem item = menu.findItem(R.id.menu_item_share);

    // Fetch and store ShareActionProvider
    mShareActionProvider = (ShareActionProvider) item.getActionProvider();

    // Return true to display menu
    return true;
}

// Call to update the share intent
private void setShareIntent(Intent shareIntent) {
    if (mShareActionProvider != null) {
        mShareActionProvider.setShareIntent(shareIntent);
    }
}
{% endhighlight %}

精髓指点：

ActionBar上快速添加分享按钮的下拉item方式写法，官方推荐。

<hr>

#**Sharing Files**

原文重点摘要：

The FileProvider class is part of the v4 Support Library. 

精髓指点：

FileProvider是v4包中的东东。

##**Specify the FileProvider**

原文重点摘要：

{% highlight ruby %}
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapp">
    <application
        ...>
        <provider
            android:name="android.support.v4.content.FileProvider"
            android:authorities="com.example.myapp.fileprovider"
            android:grantUriPermissions="true"
            android:exported="false">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/filepaths" />
        </provider>
        ...
    </application>
</manifest>
{% endhighlight %}

精髓指点：

这里，android:authorities属性字段指定了你希望使用由FileProvider生成的URI的authority。
在这个例子中，这个authority是“com.example.myapp.fileprovider”。对于你自己的应用，定义authority时，
是在你的应用包名（android:package的值）之后追加“fileprovider”。
“android:resource”属性字段是指定共享路径内容文件的路径和名字（无“.xml”后缀）。

一旦你在你的清单文件中为你的应用添加了FileProvider，你需要指定你希望共享文件的目录路径。
为了指定这个路径，我们首先在“res/xml/”下创建文件“filepaths.xml”。在这个文件中，为每一个目录添加一个XML标签。

##**Specify Sharable Directories**

原文重点摘要：

filepaths.xml content:

{% highlight ruby %}
<paths>
    <files-path path="images/" name="myimages" />
</paths>
{% endhighlight %}

精髓指点：

在这个例子中，标签共享的是在你的应用的内部存储中“files/”目录下的目录。
“path”属性字段指出了该子目录为“files/”目录下的子目录“images/”。
“name”属性字段告知FileProvider在“files/images/”子目录中的文件URI添加一个路径分段（path segment）标记：“myimages”。

标签可以有多个子标签，每一个子标签都指定一个不同的要共享的目录。
除了标签，你可以使用来分享位于外部存储的文件，而标签用来共享在你的内部缓存目录下的目录。更多关于指定共享目录的子标签的知识可以阅读：FileProvider。

注意：XML文件是你定义共享目录的唯一方式，你不可以以代码的形式添加目录。

现在你有一个完整的FileProvider说明，它在你应用的内部存储中“files/”目录下创建文件的URI，
或者是在“files/”中的子目录内的文件创建URI。当你的应用为一个文件创建了URI，
它就包含了在标签中指定的Authority（“com.example.myapp.fileprovider”），路径“myimages/”，和文件的名字。
例如定义了上边代码的一个FileProvider，你需要一个文件“default_image.jpg”的URI，FileProvider会返回如下URI：

	content://com.example.myapp.fileprovider/myimages/default_image.jpg

##**Receive File Requests**

原文重点摘要：

To receive requests for files from client apps and respond with a content URI,
your app should provide a file selection Activity.
Client apps start this Activity by calling startActivityForResult() with an Intent containing the action ACTION_PICK.
When the client app calls startActivityForResult(), your app can return a result to the client app,
in the form of a content URI for the file the user selected.

精髓指点：

为了接收来自client端app的请求并且返回一个URI，你的APP需要提供一个selection Activity。client app通过startActivityForResult发送
ACTION_PICK请求你的这个Activity。

##**Create a File Selection Activity**

正如上文说的，定义一个Selection Activity。

原文重点摘要：

{% highlight ruby %}
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    ...
        <application>
        ...
            <activity
                android:name=".FileSelectActivity"
                android:label="@"File Selector" >
                <intent-filter>
                    <action
                        android:name="android.intent.action.PICK"/>
                    <category
                        android:name="android.intent.category.DEFAULT"/>
                    <category
                        android:name="android.intent.category.OPENABLE"/>
                    <data android:mimeType="text/plain"/>
                    <data android:mimeType="image/*"/>
                </intent-filter>
            </activity>
{% endhighlight %}

{% highlight ruby %}
public class MainActivity extends Activity {
    // The path to the root of this app's internal storage
    private File mPrivateRootDir;
    // The path to the "images" subdirectory
    private File mImagesDir;
    // Array of files in the images subdirectory
    File[] mImageFiles;
    // Array of filenames corresponding to mImageFiles
    String[] mImageFilenames;
    // Initialize the Activity
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...
        // Set up an Intent to send back to apps that request a file
        mResultIntent =
                new Intent("com.example.myapp.ACTION_RETURN_FILE");
        // Get the files/ subdirectory of internal storage
        mPrivateRootDir = getFilesDir();
        // Get the files/images subdirectory;
        mImagesDir = new File(mPrivateRootDir, "images");
        // Get the files in the images subdirectory
        mImageFiles = mImagesDir.listFiles();
        // Set the Activity's result to null to begin with
        setResult(Activity.RESULT_CANCELED, null);
        /*
         * Display the file names in the ListView mFileListView.
         * Back the ListView with the array mImageFilenames, which
         * you can create by iterating through mImageFiles and
         * calling File.getAbsolutePath() for each File
         */
         ...
    }
    ...
}
{% endhighlight %}

response一个文件section：

{% highlight ruby %}
    protected void onCreate(Bundle savedInstanceState) {
        ...
        // Define a listener that responds to clicks on a file in the ListView
        mFileListView.setOnItemClickListener(
                new AdapterView.OnItemClickListener() {
            @Override
            /*
             * When a filename in the ListView is clicked, get its
             * content URI and send it to the requesting app
             */
            public void onItemClick(AdapterView<?> adapterView,
                    View view,
                    int position,
                    long rowId) {
                /*
                 * Get a File for the selected file name.
                 * Assume that the file names are in the
                 * mImageFilename array.
                 */
                File requestFile = new File(mImageFilename[position]);
                /*
                 * Most file-related method calls need to be in
                 * try-catch blocks.
                 */
                // Use the FileProvider to get a content URI
                try {
                    fileUri = FileProvider.getUriForFile(
                            MainActivity.this,
                            "com.example.myapp.fileprovider",
                            requestFile);
                } catch (IllegalArgumentException e) {
                    Log.e("File Selector",
                          "The selected file can't be shared: " +
                          clickedFilename);
                }
                ...
            }
        });
        ...
    }
{% endhighlight %}

授予权限给client客户端（Grant Permissions for the File）：

{% highlight ruby %}
    protected void onCreate(Bundle savedInstanceState) {
        ...
        // Define a listener that responds to clicks in the ListView
        mFileListView.setOnItemClickListener(
                new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView,
                    View view,
                    int position,
                    long rowId) {
                ...
                if (fileUri != null) {
                    // Grant temporary read permission to the content URI
                    mResultIntent.addFlags(
                        Intent.FLAG_GRANT_READ_URI_PERMISSION);
                }
                ...
             }
             ...
        });
    ...
    }
{% endhighlight %}

返回给请求的App URI（Share the File with the Requesting App）：

{% highlight ruby %}
    protected void onCreate(Bundle savedInstanceState) {
        ...
        // Define a listener that responds to clicks on a file in the ListView
        mFileListView.setOnItemClickListener(
                new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView,
                    View view,
                    int position,
                    long rowId) {
                ...
                if (fileUri != null) {
                    ...
                    // Put the Uri and MIME type in the result Intent
                    mResultIntent.setDataAndType(
                            fileUri,
                            getContentResolver().getType(fileUri));
                    // Set the result
                    MainActivity.this.setResult(Activity.RESULT_OK,
                            mResultIntent);
                    } else {
                        mResultIntent.setDataAndType(null, "");
                        MainActivity.this.setResult(RESULT_CANCELED,
                                mResultIntent);
                    }
                }
        });
{% endhighlight %}

Caution: Calling setFlags() is the only way to securely grant access to your files using temporary access permissions.
Avoid calling Context.grantUriPermission() method for a file's content URI, since this method grants access that you
can only revoke by calling Context.revokeUriPermission().

精髓指点：

如上就是server端的大致代码结构。特别注意setFlags是唯一的零时授权方式，grantUriPermission授权的有效期是到你主动调运
revokeUriPermission为止，所以特别注意！！！！！！！！！！

##**Requesting a Shared File**

原文重点摘要：

{% highlight ruby %}
public class MainActivity extends Activity {
    private Intent mRequestFileIntent;
    private ParcelFileDescriptor mInputPFD;
    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mRequestFileIntent = new Intent(Intent.ACTION_PICK);
        mRequestFileIntent.setType("image/jpg");
        ...
    }
    ...
    protected void requestFile() {
        /**
         * When the user requests a file, send an Intent to the
         * server app.
         * files.
         */
            startActivityForResult(mRequestFileIntent, 0);
        ...
    }
    ...
}
{% endhighlight %}

{% highlight ruby %}
	/*
     * When the Activity of the app that hosts files sets a result and calls
     * finish(), this method is invoked. The returned Intent contains the
     * content URI of a selected file. The result code indicates if the
     * selection worked or not.
     */
    @Override
    public void onActivityResult(int requestCode, int resultCode,
            Intent returnIntent) {
        // If the selection didn't work
        if (resultCode != RESULT_OK) {
            // Exit without doing anything else
            return;
        } else {
            // Get the file's content URI from the incoming Intent
            Uri returnUri = returnIntent.getData();
            /*
             * Try to open the file for "read" access using the
             * returned URI. If the file isn't found, write to the
             * error log and return.
             */
            try {
                /*
                 * Get the content resolver instance for this context, and use it
                 * to get a ParcelFileDescriptor for the file.
                 */
                mInputPFD = getContentResolver().openFileDescriptor(returnUri, "r");
            } catch (FileNotFoundException e) {
                e.printStackTrace();
                Log.e("MainActivity", "File not found.");
                return;
            }
            // Get a regular file descriptor for the file
            FileDescriptor fd = mInputPFD.getFileDescriptor();
            ...
        }
    }
{% endhighlight %}

精髓指点：

client端请求时通过如上代码即可。

##**Retrieving File Information**

原文重点摘要：

{% highlight ruby %}
    ...
    /*
     * Get the file's content URI from the incoming Intent, then
     * get the file's MIME type
     */
    Uri returnUri = returnIntent.getData();
    String mimeType = getContentResolver().getType(returnUri);
    ...
{% endhighlight %}

{% highlight ruby %}
    ...
    /*
     * Get the file's content URI from the incoming Intent,
     * then query the server app to get the file's display name
     * and size.
     */
    Uri returnUri = returnIntent.getData();
    Cursor returnCursor =
            getContentResolver().query(returnUri, null, null, null, null);
    /*
     * Get the column indexes of the data in the Cursor,
     * move to the first row in the Cursor, get the data,
     * and display it.
     */
    int nameIndex = returnCursor.getColumnIndex(OpenableColumns.DISPLAY_NAME);
    int sizeIndex = returnCursor.getColumnIndex(OpenableColumns.SIZE);
    returnCursor.moveToFirst();
    TextView nameView = (TextView) findViewById(R.id.filename_text);
    TextView sizeView = (TextView) findViewById(R.id.filesize_text);
    nameView.setText(returnCursor.getString(nameIndex));
    sizeView.setText(Long.toString(returnCursor.getLong(sizeIndex)));
    ...
{% endhighlight %}

精髓指点：

获取文件信息通过如上代码即可实现。

<hr>

#**Sharing Files with NFC**

##**Sending Files to Another Device**

原文重点摘要：

The Android Beam file transfer feature has the following requirements:

- Android Beam file transfer for large files is only available in Android 4.1 (API level 16) and higher.
- Files you want to transfer must reside in external storage. To learn more about using external storage, read Using the External Storage.
- Each file you want to transfer must be world-readable. You can set this permission by calling the method File.setReadable(true,false).
- You must provide a file URI for the files you want to transfer. Android Beam file transfer is unable to handle content URIs generated by FileProvider.getUriForFile.

精髓指点：

使用Android近场（10cm左右）NFC传输特性需要满足如下要求：

- 版本限制在Android 4.1 (API level 16)及以上。
- 要传输的文件必须在external storage。
- 要传输的文件必须全局可读，你可以使用代码设置：File.setReadable(true,false)。
- 你必须提供一个URI。Android近场传输不能操作FileProvider.getUriForFile得到的URI。

原文重点摘要：

Declare Features in the Manifest：

{% highlight ruby %}
<uses-permission android:name="android.permission.NFC" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />

<uses-feature
    android:name="android.hardware.nfc"
    android:required="true" />
{% endhighlight %}

Note that if your app only uses NFC as an option, but still functions if NFC isn't present,
you should set android:required to false, and test for NFC in code.

Now, Test for Android Beam File Transfer Support:

{% highlight ruby %}
<uses-feature android:name="android.hardware.nfc" android:required="false" />
{% endhighlight %}

{% highlight ruby %}
public class MainActivity extends Activity {
    ...
    NfcAdapter mNfcAdapter;
    // Flag to indicate that Android Beam is available
    boolean mAndroidBeamAvailable  = false;
    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...
        // NFC isn't available on the device
        if (!PackageManager.hasSystemFeature(PackageManager.FEATURE_NFC)) {
            /*
             * Disable NFC features here.
             * For example, disable menu items or buttons that activate
             * NFC-related features
             */
            ...
        // Android Beam file transfer isn't supported
        } else if (Build.VERSION.SDK_INT <
                Build.VERSION_CODES.JELLY_BEAN_MR1) {
            // If Android Beam isn't available, don't continue.
            mAndroidBeamAvailable = false;
            /*
             * Disable Android Beam file transfer features here.
             */
            ...
        // Android Beam file transfer is available, continue
        } else {
        mNfcAdapter = NfcAdapter.getDefaultAdapter(this);
        ...
        }
    }
    ...
}
{% endhighlight %}

精髓指点：

若果你的uses-feature设置true，强行使用NFC，那就按照第一段代码声明特性。如果NFC不是必须的，那就设置为false，然后在java代码
中做test check，如后半段代码。

原文重点摘要：

Create a Callback Method that Provides Files：

add a callback method that the system invokes when Android Beam file transfer detects that the user wants to send files to another NFC-enabled device.
In this callback method, return an array of Uri objects. Android Beam file transfer copies the files represented by these URIs to the receiving device.

{% highlight ruby %}
public class MainActivity extends Activity {
    ...
    // List of URIs to provide to Android Beam
    private Uri[] mFileUris = new Uri[10];
    ...
    /**
     * Callback that Android Beam file transfer calls to get
     * files to share
     */
    private class FileUriCallback implements NfcAdapter.CreateBeamUrisCallback {
        public FileUriCallback() {
        }
        /**
         * Create content URIs as needed to share with another device
         */
        @Override
        public Uri[] createBeamUris(NfcEvent event) {
            return mFileUris;
        }
    }
    ...
}
{% endhighlight %}

{% highlight ruby %}
public class MainActivity extends Activity {
    ...
    // Instance that returns available files from this app
    private FileUriCallback mFileUriCallback;
    ...
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        ...
        // Android Beam file transfer is available, continue
        ...
        mNfcAdapter = NfcAdapter.getDefaultAdapter(this);
        /*
         * Instantiate a new FileUriCallback to handle requests for
         * URIs
         */
        mFileUriCallback = new FileUriCallback();
        // Set the dynamic callback for URI requests.
        mNfcAdapter.setBeamPushUrisCallback(mFileUriCallback,this);
        ...
    }
    ...
}
{% endhighlight %}

Specify the Files to Send：

To transfer one or more files to another NFC-enabled device, get a file URI (a URI with a file scheme) for each file and then add the URI to an array of Uri objects.
To transfer a file, you must also have permanent read access for the file.

{% highlight ruby %}
        /*
         * Create a list of URIs, get a File,
         * and set its permissions
         */
        private Uri[] mFileUris = new Uri[10];
        String transferFile = "transferimage.jpg";
        File extDir = getExternalFilesDir(null);
        File requestFile = new File(extDir, transferFile);
        requestFile.setReadable(true, false);
        // Get a URI for the File and add it to the list of URIs
        fileUri = Uri.fromFile(requestFile);
        if (fileUri != null) {
            mFileUris[0] = fileUri;
        } else {
            Log.e("My Activity", "No File URI available for file.");
        }
{% endhighlight %}

精髓指点：

创建一个系统调运触发的回调提供文件。这个回调返回一个array的URI，Android近场通信通过这些URIs传输文件。具体如上两段代码。

指定传输的文件必须具有永久的可读权限，如上最后一段代码。

##**Receiving Files from Another Device**

原文重点摘要：

Android Beam file transfer copies files to a special directory on the receiving device.
It also scans the copied files using the Android Media Scanner and adds entries for media files to the MediaStore provider. 

精髓指点：

Android近场传输文件存储在一个特定的目录下，也会被Media Scanner扫描添加到MediaStore中去的。

原文重点摘要：

When Android Beam file transfer finishes copying files to the receiving device,
it posts a notification containing an Intent with the action ACTION_VIEW,
the MIME type of the first file that was transferred, and a URI that points to the first file. When the user clicks the notification,
this intent is sent out to the system. To have your app respond to this intent,
add an <intent-filter> element for the <activity> element of the Activity that should respond. In the <intent-filter> element,
add the following child elements:

{% highlight ruby %}
<action android:name="android.intent.action.VIEW" />
<category android:name="android.intent.category.CATEGORY_DEFAULT" />
{% endhighlight %}

such as this：

{% highlight ruby %}
    <activity
        android:name="com.example.android.nfctransfer.ViewActivity"
        android:label="Android Beam Viewer" >
        ...
        <intent-filter>
            <action android:name="android.intent.action.VIEW"/>
            <category android:name="android.intent.category.DEFAULT"/>
            ...
        </intent-filter>
    </activity>
{% endhighlight %}

{% highlight ruby %}
 <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
{% endhighlight %}

Note: Android Beam file transfer is not the only source of an ACTION_VIEW intent. Other apps on the receiving device can also send an Intent with this action.
Handling this situation is discussed in the section Get the directory from a content URI.

精髓指点：

当近场通信传输到接收设备时会发送一个通知，附带一个intent，其action是ACTION_VIEW，MIME是收到的第一个文件解析得到的，当点击这个通知时，系统就会
发出这个intent。你需要通过如上代码添加intent-filter代码。不过要注意不需要MIME类型。

你需要添加声明外部读写权限。

由于Android近场传输不是唯一能发送ACTION_VIEW的intent，所以其他app也可以发送ACTION_VIEW的intent，为了解决这个问题下面Get the directory from a content URI小节
会讲到怎么办。

####**Get the Directory for Copied Files**

原文重点摘要：

To determine how you should handle the incoming Intent, you need to examine its scheme and authority.

To get the scheme for the URI, call Uri.getScheme(). The following code snippet shows you how to determine the scheme and handle the URI accordingly:

{% highlight ruby %}
public class MainActivity extends Activity {
    ...
    // A File object containing the path to the transferred files
    private File mParentPath;
    // Incoming Intent
    private Intent mIntent;
    ...
    /*
     * Called from onNewIntent() for a SINGLE_TOP Activity
     * or onCreate() for a new Activity. For onNewIntent(),
     * remember to call setIntent() to store the most
     * current Intent
     *
     */
    private void handleViewIntent() {
        ...
        // Get the Intent action
        mIntent = getIntent();
        String action = mIntent.getAction();
        /*
         * For ACTION_VIEW, the Activity is being asked to display data.
         * Get the URI.
         */
        if (TextUtils.equals(action, Intent.ACTION_VIEW)) {
            // Get the URI from the Intent
            Uri beamUri = mIntent.getData();
            /*
             * Test for the type of URI, by getting its scheme value
             */
            if (TextUtils.equals(beamUri.getScheme(), "file")) {
                mParentPath = handleFileUri(beamUri);
            } else if (TextUtils.equals(
                    beamUri.getScheme(), "content")) {
                mParentPath = handleContentUri(beamUri);
            }
        }
        ...
    }
    ...
}
{% endhighlight %}

精髓指点：

决定如何处理intent你需要检测他们的scheme和authority。通过如上代码得到URL。

####Get the directory from a file URI

原文重点摘要：

If the incoming Intent contains a file URI, the URI contains the absolute file name of a file,
including the full directory path and file name. For Android Beam file transfer,
the directory path points to the location of the other transferred files, if any.
To get the directory path, get the path part of the URI, which contains all of the URI except the file: prefix.
Create a File from the path part, then get the parent path of the File:

{% highlight ruby %}
  ...
    public String handleFileUri(Uri beamUri) {
        // Get the path part of the URI
        String fileName = beamUri.getPath();
        // Create a File object for this filename
        File copiedFile = new File(fileName);
        // Get a string containing the file's parent directory
        return copiedFile.getParent();
    }
    ...
{% endhighlight %}

精髓指点：

如果intent包含一个file的URI，URI包含绝对的文件名和全路径。对于近场传输文件，目录路径指定了其他近场传输文件的路径，通过如上代码
得到parent路径目录。

####Get the directory from a content URI

原文重点摘要：

If the incoming Intent contains a content URI, the URI may point to a directory and file name stored in the MediaStore content provider.
You can detect a content URI for MediaStore by testing the URI's authority value.
A content URI for MediaStore may come from Android Beam file transfer or from another app,
but in both cases you can retrieve a directory and file name for the content URI.

You can also receive an incoming ACTION_VIEW intent containing a content URI for a content provider other than MediaStore.
In this case, the content URI doesn't contain the MediaStore authority value, and the content URI usually doesn't point to a directory.

Note: For Android Beam file transfer, you receive a content URI in the ACTION_VIEW intent if the first incoming file has a MIME type of "audio/*",
"image/*", or "video/*", indicating that the file is media- related.
Android Beam file transfer indexes the media files it transfers by running Media Scanner on the directory where it stores transferred files.
Media Scanner writes its results to the MediaStore content provider, then it passes a content URI for the first file back to Android Beam file transfer.
This content URI is the one you receive in the notification Intent. To get the directory of the first file,
you retrieve it from MediaStore using the content URI.

精髓指点：

如果intent包含一个内容URI，URI可能指向一个存储在MediaStore的路径和文件名，你可以通过测试authority在MediaStore。MediaStore的URI可能是近场传输文件或者其他，两种情况下
你都可以检索到目录和文件名。


你也可以接收传入ACTION_VIEW意图且包含内容URI MediaStore以外的内容提供者。在这种情况下,内容不包含URI MediaStore权威值,和内容URI通常不会指向一个目录。

注意:Android近场传输,您收到内容URI ACTION_VIEW意图如果第一个输入文件的MIME类型是“audio/ *”,“image/ *”或“video/ *”,表明媒体相关的文件。
Android近场传输的媒体文件传输通过运行媒体扫描的目录存储传输文件。媒体扫描将结果写入MediaStore内容提供者,那么内容URI第一个文件传送回Android近场传输。
这个内容URI是通知你收到的意图。目录的第一个文件,你使用内容检索从MediaStore URI。

####Determine the content provider

原文重点摘要：

To determine if you can retrieve a file directory from the content URI,
determine the the content provider associated with the URI by calling Uri.getAuthority() to get the URI's authority.
The result has two possible values:

- MediaStore.AUTHORITY    The URI is for a file or files tracked by MediaStore. Retrieve the full file name from MediaStore, and get directory from the file name.

- Any other authority value    A content URI from another content provider. Display the data associated with the content URI, but don't get the file directory.

To get the directory for a MediaStore content URI, run a query that specifies the incoming content URI for the Uri argument and the column MediaColumns.DATA
for the projection. The returned Cursor contains the full path and name for the file represented by the URI.
This path also contains all the other files that Android Beam file transfer just copied to the device.

The following snippet shows you how to test the authority of the content URI and retrieve the the path and file name for the transferred file:

{% highlight ruby %}
    ...
    public String handleContentUri(Uri beamUri) {
        // Position of the filename in the query Cursor
        int filenameIndex;
        // File object for the filename
        File copiedFile;
        // The filename stored in MediaStore
        String fileName;
        // Test the authority of the URI
        if (!TextUtils.equals(beamUri.getAuthority(), MediaStore.AUTHORITY)) {
            /*
             * Handle content URIs for other content providers
             */
        // For a MediaStore content URI
        } else {
            // Get the column that contains the file name
            String[] projection = { MediaStore.MediaColumns.DATA };
            Cursor pathCursor =
                    getContentResolver().query(beamUri, projection,
                    null, null, null);
            // Check for a valid cursor
            if (pathCursor != null &&
                    pathCursor.moveToFirst()) {
                // Get the column index in the Cursor
                filenameIndex = pathCursor.getColumnIndex(
                        MediaStore.MediaColumns.DATA);
                // Get the full file name including path
                fileName = pathCursor.getString(filenameIndex);
                // Create a File object for the filename
                copiedFile = new File(fileName);
                // Return the parent directory of the file
                return new File(copiedFile.getParent());
             } else {
                // The query didn't work; return null
                return null;
             }
        }
    }
    ...
{% endhighlight %}

精髓指点：

确定如果您可以检索一个文件目录从内容URI，确定与URI相关联的内容提供者通过调用Uri.getAuthority()获取URI的权威。结果有两个可能的值:

- MediaStore.AUTHORITY    一个文件或文件的URI是MediaStore追踪的。从MediaStore检索完整的文件名,目录的文件名。
- Any other authority value    从另一个内容提供者内容URI。显示数据与内容相关联的URI,不过得不到文件的目录。

为了得到URI MediaStore内容的目录，运行一个查询，该查询指定传入的内容Uri argument和列MediaColumns.DATA。
返回的指针包含文件的完整路径和名称所代表的URI。这条路径的所有其他文件也是Android近场传输的。

如上代码测试了如何得到传输文件。
