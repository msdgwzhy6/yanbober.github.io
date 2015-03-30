---
layout: post
keywords: Android Developer
description: Android Developer读书笔记
title: "[Training] Building Your First App & Adding the ActionBar"
categories: [开发文档]
tags: [Android读书笔记]
group: archive
icon: file-alt
---
{% include site/setup %}
<hr>
重读Android Developer笔记核心记录
<hr>

##**Building Your First App**

原文重点摘要：

The default weight for all views is 0, so if you specify any weight value greater than 0 to only one view,
then that view fills whatever space remains after all views are given the space they require.

{% highlight ruby %}
<EditText
    android:layout_weight="1"
    android:layout_width="0dp"
    ... />
{% endhighlight %}

To improve the layout efficiency when you specify the weight,
you should change the width of the EditText to be zero (0dp).
Setting the width to zero improves layout performance because using "wrap_content" as the width
requires the system to calculate a width that is ultimately irrelevant because the weight value
requires another width calculation to fill the remaining space.

精髓指点：

任何View的weight属性不设置默认为0。在使用weight时建议将对应的width或者hight设置为0dp，这样可以提升解析效率，
减少不必要的计算。

<hr>

##**Adding the ActionBar**

####**Setting Up the Action Bar**

原文重点摘要：

Beginning with Android 3.0 (API level 11), the action bar is included in all activities that use the Theme.Holo
theme (or one of its descendants), which is the default theme when either the targetSdkVersion or minSdkVersion
attribute is set to "11" or greater.If you've created a custom theme, be sure it uses one of the Theme.Holo themes
as its parent.

Adding the action bar when running on versions older than Android 3.0 (down to Android 2.1) requires that you
include the Android Support Library in your application. To get started, read the Support Library Setup document
and set up the v7 appcompat library (once you've downloaded the library package, follow the instructions for Adding
libraries with resources).Once you have the Support Library integrated with your app project:

- Update your activity so that it extends ActionBarActivity. 
- In your manifest file, update either the <application> element or individual <activity> elements to use one of
the Theme.AppCompat themes. If you've created a custom theme, be sure it uses one of the Theme.AppCompat themes as
its parent.

精髓指点：

- 如果你的min版本是Android 3.0以上，你的主题使用Theme.Holo或者他的子项，你的Activity继承Activity就可以。
- 如果你要兼容到Android 2.1版本使用ActionBar，你需要使用官方Support Library v7 appcompat。你的主题必须使用Theme.AppCompat
或者其子项，你的Activity必须继承support包的ActionBarActivity。

####**Adding Action Buttons**

原文重点摘要：

{% highlight ruby %}
<menu xmlns:android="http://schemas.android.com/apk/res/android" >
    <item android:id="@+id/action_search"
          android:icon="@drawable/ic_action_search"
          android:title="@string/action_search"
          android:showAsAction="ifRoom" />
</menu>
{% endhighlight %}

If your app is using the Support Library for compatibility on versions as low as Android 2.1, 
the showAsAction attribute is not available from the android: namespace. Instead this attribute
is provided by the Support Library and you must define your own XML namespace and use that namespace
as the attribute prefix. (A custom XML namespace should be based on your app name, but it can be any
name you want and is only accessible within the scope of the file in which you declare it.) For example:

{% highlight ruby %}
<menu xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:yourapp="http://schemas.android.com/apk/res-auto" >
    <!-- Search, should appear as action button -->
    <item android:id="@+id/action_search"
          android:icon="@drawable/ic_action_search"
          android:title="@string/action_search"
          yourapp:showAsAction="ifRoom"  />
    ...
</menu>
{% endhighlight %}

精髓指点：

如果你要考虑3.0以下兼容2.1版本，你自定义的menu文件夹下的xml中showAsAction属性不能使用android：命名空间，
需要自己申明xmlns:yourapp="http://schemas.android.com/apk/res-auto"。

showAsAction的取值：

- always：这个值会使菜单项一直显示在Action Bar上。
- ifRoom：如果有足够的空间，这个值会使菜单项显示在Action Bar上。
- never：这个值使菜单项永远都不出现在Action Bar上。
- withText：这个值使菜单项和它的图标，菜单文本一起显示。

插一句题外话，之前版本很多人自定义attr时喜欢使用xmlns:yourapp="http://schemas.android.com/apk/你的app包名"。
而现在推荐使用xmlns:app="http://schemas.android.com/apk/res-auto"。

原文重点摘要：

When running on Android 4.1 (API level 16) or higher, or when using ActionBarActivity from the
Support Library, performing Up navigation simply requires that you declare the parent activity
in the manifest file and enable the Up button for the action bar.

{% highlight ruby %}
<application ... >
    ...
    <!-- The main/home activity (it has no parent activity) -->
    <activity
        android:name="com.example.myfirstapp.MainActivity" ...>
        ...
    </activity>
    <!-- A child of the main activity -->
    <activity
        android:name="com.example.myfirstapp.DisplayMessageActivity"
        android:label="@string/title_activity_display_message"
        android:parentActivityName="com.example.myfirstapp.MainActivity" >
        <!-- Parent activity meta-data to support 4.0 and lower -->
        <meta-data
            android:name="android.support.PARENT_ACTIVITY"
            android:value="com.example.myfirstapp.MainActivity" />
    </activity>
</application>
{% endhighlight %}

{% highlight ruby %}
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_displaymessage);

    getSupportActionBar().setDisplayHomeAsUpEnabled(true);
    // If your minSdkVersion is 11 or higher, instead use:
    // getActionBar().setDisplayHomeAsUpEnabled(true);
}
{% endhighlight %}

Because the system now knows MainActivity is the parent activity for DisplayMessageActivity,
when the user presses the Up button, the system navigates to the parent activity as appropriate—you
do not need to handle the Up button's event.

精髓指点：

如果你要使用ActionBar的向上导航，你需要在Activity中设置setDisplayHomeAsUpEnabled(true)，同时在manifest
文件声明你当前Activity向上导航返回的父Activity的名字，譬如上边属性android:parentActivityName="com.example.myfirstapp.MainActivity"
，其中<meta-data>里的写法是为兼容4.0以下版本的。

特别注意，这种写法再也不需要我们像以前一样在onOptionsItemSelected方法中自己去finish当前Activity啥的。你什么都不需要做。

####**Styling the Action Bar**

原文重点摘要：

If you are using the Support Library APIs for the action bar, then you must use (or override)
the Theme.AppCompat family of styles (rather than the Theme.Holo family, available in API level
11 and higher). In doing so, each style property that you declare must be declared twice: once
using the platform's style properties (the android: properties) and once using the style
properties included in the Support Library (the appcompat.R.attr properties—the context for these
properties is actually your app). 

精髓指点：

还是在强调你是为了兼容低版本还是不需要兼容而使用的主题不同，不过如果你要自定义ActionBar风格需要注意：
如果是为了兼容低版本你需要each style property that you declare must be declared twice。这个在下面的例子说明：

For Android 3.0 and higher only：

{% highlight ruby %}
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!-- the theme applied to the application or activity -->
    <style name="CustomActionBarTheme"
           parent="@android:style/Theme.Holo.Light.DarkActionBar">
        <item name="android:actionBarStyle">@style/MyActionBar</item>
    </style>

    <!-- ActionBar styles -->
    <style name="MyActionBar"
           parent="@android:style/Widget.Holo.Light.ActionBar.Solid.Inverse">
        <item name="android:background">@drawable/actionbar_background</item>
    </style>
</resources>
{% endhighlight %}

For Android 2.1 and higher：

{% highlight ruby %}
<resources>
    <style name="MYTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!--自定义兼容低版本的ActionBar主题-->
        <item name="android:actionBarStyle">@style/MyActionBar</item>
        <item name="actionBarStyle">@style/MyActionBar</item>
        <!--自定义兼容低版本的ActionBar Menu的文字颜色-->
        <item name="android:actionMenuTextColor">#ff0fffff</item>
        <item name="actionMenuTextColor">#ff0fffff</item>
        <!--自定义兼容低版本的ActionBar Menu的文字其他属性-->
        <item name="android:actionMenuTextAppearance">@style/MyMenuStyle</item>
        <item name="actionMenuTextAppearance">@style/MyMenuStyle</item>
        <!--设置是否为悬浮ActionBar-->
        <item name="android:windowActionBarOverlay">true</item>
        <item name="windowActionBarOverlay">true</item>
    </style>

    <style name="MyActionBar" parent="@style/Widget.AppCompat.Light.ActionBar">
        <!--自定义背景-->
        <item name="android:background">#66ff0ff0</item>
        <item name="background">@android:color/holo_red_dark</item>
        <!--自定义左上角返回按钮-->
        <item name="android:homeAsUpIndicator">@drawable/ic_action_undo</item>
        <item name="homeAsUpIndicator">@drawable/ic_action_undo</item>
        <!--自定义Title字体颜色大小-->
        <item name="android:titleTextStyle">@style/MyTitleTextStyle</item>
        <item name="titleTextStyle">@style/MyTitleTextStyle</item>
    </style>

    <style name="MyTitleTextStyle" parent="TextAppearance.AppCompat.Widget.ActionBar.Title">
        <!--自定义Title字体颜色大小-->
        <item name="android:textColor">#ff00ff00</item>
        <item name="android:textSize">30sp</item>
    </style>

    <style name="MyMenuStyle" parent="TextAppearance.AppCompat.Widget.ActionBar.Menu">
        <!--Menu取消粗体，设置字体-->
        <item name="android:textStyle">@null</item>
        <item name="android:textSize">25sp</item>
    </style>
</resources>
{% endhighlight %}

####Overlaying the Action Bar

设置悬浮在上面兼容版本代码主题中已经写了windowActionBarOverlay。注意悬浮以后ActionBar的空间被算作content空间。
如果你想让你的ActionBar悬浮，同时想让你的content布局从ActionBar不悬浮处开始显示（还会被计算），则可以如下设置：

{% highlight ruby %}
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingTop="?android:attr/actionBarSize">
    ...
	<!--Support library compatibility use: android:paddingTop="?attr/actionBarSize"-->
</RelativeLayout>
{% endhighlight %}

