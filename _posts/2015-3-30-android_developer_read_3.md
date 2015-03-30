---
layout: post
keywords: Android Developer
description: Android Developer读书笔记
title: "[Training] Saving Data"
categories: [开发文档]
tags: [Android读书笔记]
group: archive
icon: file-alt
---
{% include site/setup %}
<hr>
重读Android Developer笔记核心记录
<hr>

##**Saving Key-Value Sets**

原文重点摘要：

You can create a new shared preference file or access an existing one by calling one of two methods:

- getSharedPreferences() — Use this if you need multiple shared preference files identified by name, which you specify with the first parameter. You can call this from any Context in your app.

- getPreferences() — Use this from an Activity if you need to use only one shared preference file for the activity. Because this retrieves a default shared preference file that belongs to the activity, you don't need to supply a name.

Context.MODE_PRIVATE | MODE_WORLD_READABLE | MODE_WORLD_WRITEABLE

精髓指点：

可以通过两个方法拿到SharedPreferences的handle。他们区别是：

- 一个是activity的方法，不需要名字，默认为Activity类名。

- 一个是Context的方法，需要名字，没有默认值。

都可以设置Context的读写权限为三者之一或者组合。

##**Saving Files**

####**Choose Internal or External Storage**

原文重点摘要：

All Android devices have two file storage areas: "internal" and "external" storage.
These names come from the early days of Android, when most devices offered built-in non-volatile memory
(internal storage), plus a removable storage medium such as a micro SD card (external storage).
Some devices divide the permanent storage space into "internal" and "external" partitions,
so even without a removable storage medium, there are always two storage spaces and the API behavior
is the same whether the external storage is removable or not.

精髓指点：

Android的存储分为内部存储和外部存储，这个名字是历史遗留问题。大多数设备都有一个永久的内部存储和一个可挂载的
外部存储；但是有些设备将永久存储又分为内部和外部，所以有些设备即使没有挂在外部存储API也识别有外设存储。

原文重点摘要：

Although apps are installed onto the internal storage by default, you can specify the android:installLocation
attribute in your manifest so your app may be installed on external storage. Users appreciate this option when
the APK size is very large and they have an external storage space that's larger than the internal storage. 

精髓指点：

App可以通过android:installLocation设置安装位置。你可以依据你的App大小酌情考虑安装在哪。

####**Obtain Permissions for External Storage**

原文重点摘要：

Currently, all apps have the ability to read the external storage without a special permission.
However, this will change in a future release. If your app needs to read the external storage
(but not write to it), then you will need to declare the READ_EXTERNAL_STORAGE permission.
To ensure that your app continues to work as expected, you should declare this permission now,
before the change takes effect.
 
You don’t need any permissions to save files on the internal storage. Your application always has
permission to read and write files in its internal storage directory.

{% highlight ruby %}
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
{% endhighlight %}

精髓指点：

当前所有App都有权限读取外部存储的文件，不需要额外权限。但是将来的版本将会改变，所以你需要声明READ_EXTERNAL_STORAGE
权限。对于App自己的文件夹不需要权限，因为自身就具备。

####**Save a File on Internal Storage**

原文重点摘要：

When saving a file to internal storage, you can acquire the appropriate directory as a File by calling oneof two methods:

- getFilesDir()    Returns a File representing an internal directory for your app.

- getCacheDir()    Returns a File representing an internal directory for your app's temporary cache files. 
Be sure to delete each file once it is no longer needed and implement a reasonable size limit for the amount
of memory you use at any given time, such as 1MB. If the system begins running low on storage, it may delete
your cache files without warning. 

To create a new file in one of these directories, you can use the File() constructor, 
passing the File provided by one of the above methods that specifies your internal storage directory. 

For example:

{% highlight ruby %}
File file = new File(context.getFilesDir(), filename);
{% endhighlight %}

精髓指点：

当你想内部存储时有两个方法可以方便存储，一个用来获取data/data/apppackage/file路径，
一个用来获取临时cache路径，cache路径当系统紧张时会自动给你删掉，最好自己也要设置限制。

原文重点摘要：

{% highlight ruby %}
String filename = "myfile";
String string = "Hello world!";
FileOutputStream outputStream;

try {
  outputStream = openFileOutput(filename, Context.MODE_PRIVATE);
  outputStream.write(string.getBytes());
  outputStream.close();
} catch (Exception e) {
  e.printStackTrace();
}
{% endhighlight %}

{% highlight ruby %}
public File getTempFile(Context context, String url) {
    File file;
    try {
        String fileName = Uri.parse(url).getLastPathSegment();
        file = File.createTempFile(fileName, null, context.getCacheDir());
    catch (IOException e) {
        // Error while creating file
    }
    return file;
}
{% endhighlight %}

精髓指点：

这两个方法中openFileOutput也是向内部路径写入流，createTempFile在内部cache创建临时文件。

原文重点摘要：

Your app's internal storage directory is specified by your app's package name in a special
location of the Android file system. Technically, another app can read your internal files
if you set the file mode to be readable. However, the other app would also need to know your
app package name and file names. Other apps cannot browse your internal directories and do not
have read or write access unless you explicitly set the files to be readable or writable. So as
long as you use MODE_PRIVATE for your files on the internal storage, they are never accessible
to other apps.

精髓指点：

Android对于App内部文件夹是按照app来分权限的，如果你设置MODE_PRIVATE权限，其他app永远不能访问。
如果不是MODE_PRIVATE权限，其他App只要知道你的包名和文件名就可以访问。

####**Save a File on External Storage**

原文重点摘要：

{% highlight ruby %}
/* Checks if external storage is available for read and write */
public boolean isExternalStorageWritable() {
    String state = Environment.getExternalStorageState();
    if (Environment.MEDIA_MOUNTED.equals(state)) {
        return true;
    }
    return false;
}

/* Checks if external storage is available to at least read */
public boolean isExternalStorageReadable() {
    String state = Environment.getExternalStorageState();
    if (Environment.MEDIA_MOUNTED.equals(state) ||
        Environment.MEDIA_MOUNTED_READ_ONLY.equals(state)) {
        return true;
    }
    return false;
}
{% endhighlight %}

精髓指点：

上面说了，外部设备是不可靠的，所以使用前需要先判断。如上代码即可。

原文重点摘要：

Although the external storage is modifiable by the user and other apps,
there are two categories of files you might save here:

- Public files    Files that should be freely available to other apps and to the user. 
When the user uninstalls your app, these files should remain available to the user.
For example, photos captured by your app or other downloaded files.

- Private files    Files that rightfully belong to your app and should be deleted when
the user uninstalls your app. Although these files are technically accessible by the
user and other apps because they are on the external storage, they are files that
realistically don't provide value to the user outside your app. When the user uninstalls
your app, the system deletes all files in your app's external private directory.
For example, additional resources downloaded by your app or temporary media files.

精髓指点：

虽然外部存储是可以随意访问的，但是也分为两类，一类是当你卸载了应用她还存在，一类是卸载应用时自动清除。
你应该依据你的需求选择哪种外部存储。

####访问外部存储的权限

- Android1.0开始，写操作受权限WRITE_EXTERNAL_STORAGE保护。

- Android 4.1开始，读操作受权限READ_EXTERNAL_STORAGE保护，上文也可以看出来。

- Android 4.4开始，应用可以管理在它外部存储上的特定包名目录，而不用获取WRITE_EXTERNAL_STORAGE权限。

例子：比如我的小米平板里，一个包名为com.telent.mobileqq的应用，可以自由访问外存上的Android/data/com.telent.mobileqq/目录。

####注意事项

- 外部存储对数据提供的保护较少，所以系统不应该存储敏感数据在外部存储上。

- 特别是你的应用配置和log文件应该存储在内部存储中，这样它们可以被有效地保护。

对于多用户的情况，一般每个用户都会有自己独立的外部存储，应用仅对当前用户的外部存储有访问权限。

正如上面英文原文说的public和private：

- Context.getExternalFilesDir返回的目录下应该放应用私有的文件，在应用被卸载的时候，系统会清理的就是这个目录。

- getExternalStoragePublicDirectory(String)放一些共享文件。

getExternalStoragePublicDirectory(String type)这个方法接收一个参数，表明目录所放的文件的类型，
传入的参数是Environment类中的DIRECTORY_XXX静态变量，比如DIRECTORY_DCIM等。
注意：传入的类型参数不能是null，返回的目录路径有可能不存在，所以必须在使用之前确认一下，
比如使用File.mkdirs创建该路径。

从Android 4.4开始，如果你的应用只是需要存储一些内部数据，可以考虑使用：
getExternalFilesDir(String)或者getExternalCacheDir()，它们不需要获取权限。 

####Environment API的目录

- getDataDirectory()：用户数据目录。
- getDownloadCacheDirectory()：下载缓存内容目录。
- getExternalStorageDirectory()：主要的外部存储目录。
- getRootDirectory()：得到Android的根目录。
- isExternalStorageEmulated()设备的外存是否是用内存模拟的，是则返回true。(API Level 11)
- isExternalStorageRemovable()设备的外存是否是可以拆卸的，比如SD卡，是则返回true。(API Level 9)

####**Query Free Space**

原文重点摘要：

use Java I/O File class method : getFreeSpace() or getTotalSpace().

精髓指点：

通过Java的这两个方法可以判断你要存储的路径剩余空间等。

##**Saving Data in SQL Databases**

原文重点摘要：

By implementing the BaseColumns interface, your inner class can inherit a primary key field called _ID
that some Android classes such as cursor adaptors will expect it to have. It's not required, but this
can help your database work harmoniously with the Android framework.

{% highlight ruby %}
public final class FeedReaderContract {
    // To prevent someone from accidentally instantiating the contract class,
    // give it an empty constructor.
    public FeedReaderContract() {}

    /* Inner class that defines the table contents */
    public static abstract class FeedEntry implements BaseColumns {
        public static final String TABLE_NAME = "entry";
        public static final String COLUMN_NAME_ENTRY_ID = "entryid";
        public static final String COLUMN_NAME_TITLE = "title";
        public static final String COLUMN_NAME_SUBTITLE = "subtitle";
        ...
    }
}
{% endhighlight %}

精髓指点：

通过实现BaseColumns接口，内部类可以继承一个主键字段名为_ID接口，但这可以防止名字写错误操作。
