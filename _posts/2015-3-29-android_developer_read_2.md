---
layout: post
keywords: Android Developer
description: Android Developer读书笔记
title: "[Training] Managing the Activity Lifecycle & Building a Dynamic UI with Fragments"
categories: [开发文档]
tags: [Android读书笔记]
group: archive
icon: file-alt
---
{% include site/setup %}
<hr>
重读Android Developer笔记核心记录
<hr>

##**Managing the Activity Lifecycle**

原文重点摘要：

<img src="http://yanbober.github.io/image/ad/1.png" />

精髓指点：

Activity生命周期，不解释。

原文重点摘要：

{% highlight ruby %}
<activity android:name=".MainActivity" android:label="@string/app_name">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
{% endhighlight %}

If either the MAIN action or LAUNCHER category are not declared for one of your activities,
then your app icon will not appear in the Home screen's list of apps.

精髓指点：

去掉上面两个属性中任何一个都可以隐藏App图标，去掉启动项。

原文重点摘要：

Always call the superclass implementation of onSaveInstanceState()
so the default implementation can save the state of the view hierarchy.

Always call the superclass implementation of onRestoreInstanceState() so the default implementation can
restore the state of the view hierarchy.

精髓指点：

如果你要重写这两个状态保存方法，记得一定要调运super方法，以便可以取得你的数据同时取得view hierarchy state。

##**Building a Dynamic UI with Fragments**

####**Create a Fragments**

原文重点摘要：

If you decide that the minimum API level your app requires is 11 or higher, 
you don't need to use the Support Library and can instead use the framework's built
in Fragment class and related APIs. Just be aware that this lesson is focused on using
the APIs from the Support Library, which use a specific package signature and sometimes
slightly different API names than the versions included in the platform. However, you can
also include the action bar in your activities by instead using the v7 appcompat library,
which is compatible with Android 2.1 (API level 7) and also includes the Fragment APIs.

FragmentActivity is a special activity provided in the Support Library to handle fragments
on system versions older than API level 11. If the lowest system version you support is API
level 11 or higher, then you can use a regular Activity.

If you're using the v7 appcompat library, your activity should instead extend ActionBarActivity,
which is a subclass of FragmentActivity.

精髓指点：

如果你的最低版本高于11直接使用，低于11使用support v4包的API，在使用v4时你同样可以使用v7的ActionBar等。
FragmentActivity也是为低版本准备的兼容包里的东东，不需要兼容的直接使用Activity就可以了，但是如果你使用
兼容包v4同时又使用兼容包v7那就直接继承ActionBarActivity就行，因为兼容包v7中的ActionBarActivity是兼容包v4中
FragmentActivity的子类。

原文重点摘要：

Add a Fragment to an Activity using XML：
{% highlight ruby %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent">

    <fragment android:name="com.example.android.fragments.HeadlinesFragment"
              android:id="@+id/headlines_fragment"
              android:layout_weight="1"
              android:layout_width="0dp"
              android:layout_height="match_parent" />

    <fragment android:name="com.example.android.fragments.ArticleFragment"
              android:id="@+id/article_fragment"
              android:layout_weight="2"
              android:layout_width="0dp"
              android:layout_height="match_parent" />

</LinearLayout>
{% endhighlight %}

{% highlight ruby %}
public class MainActivity extends FragmentActivity {
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.news_articles);
    }
}
{% endhighlight %}

精髓指点：

以上代码就是写死不可以remove，replace的Fragment代码。

####**Building a Flexible UI**

原文重点摘要：

Keep in mind that when you perform fragment transactions, such as replace or remove one,
it's often appropriate to allow the user to navigate backward and "undo" the change.
To allow the user to navigate backward through the fragment transactions,
you must call addToBackStack(null) before you commit the FragmentTransaction.

When you remove or replace a fragment and add the transaction to the back stack,
the fragment that is removed is stopped (not destroyed).
If the user navigates back to restore the fragment, it restarts.
If you do not add the transaction to the back stack,
then the fragment is destroyed when removed or replaced.

精髓指点：

如果正常replace或者add的fragment进行回退时直接退出，想要保存栈需要addToBackStack。但是切记，
当你remove or replace时设置了addToBackStack，Fragment调运removed时执行的是stopped不是destroyed，back时
走restart方法。如果没有使用addToBackStack方法则走destory。

####**Communicating with Other Fragments**

原文重点摘要：

Often you will want one Fragment to communicate with another,
for example to change the content based on a user event.
All Fragment-to-Fragment communication is done through the associated Activity.
Two Fragments should never communicate directly.

{% highlight ruby %}
public class HeadlinesFragment extends ListFragment {
    OnHeadlineSelectedListener mCallback;

    // Container Activity must implement this interface
    public interface OnHeadlineSelectedListener {
        public void onArticleSelected(int position);
    }

    @Override
    public void onAttach(Activity activity) {
        super.onAttach(activity);
        
        // This makes sure that the container activity has implemented
        // the callback interface. If not, it throws an exception
        try {
            mCallback = (OnHeadlineSelectedListener) activity;
        } catch (ClassCastException e) {
            throw new ClassCastException(activity.toString()
                    + " must implement OnHeadlineSelectedListener");
        }
    }
    ...
}
{% endhighlight %}

{% highlight ruby %}
public static class MainActivity extends Activity implements HeadlinesFragment.OnHeadlineSelectedListener{
    ...
    public void onArticleSelected(int position) {
        // The user selected the headline of an article from the HeadlinesFragment
        // Do something here to display that article
    }
}
{% endhighlight %}

精髓指点：

使用接口进行Fragment间通行交互，注意onAttach方法里的写法，也可以自定义set方法的。

特别注意Fragment与Fragment间永远不可直接交互，必须使用Activity做中间桥梁。
