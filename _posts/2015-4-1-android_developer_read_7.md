---
layout: post
keywords: Android Developer
description: Android Developer读书笔记
title: "[Training] Printing Content"
categories: [开发文档]
tags: [Android读书笔记]
group: archive
icon: file-alt
---
{% include site/setup %}
<hr>
重读Android Developer笔记核心记录
<hr>

#**Printing Photos**

原文重点摘要：

This lesson shows you how to print an image using the v4 support library PrintHelper class.

精髓指点：

下文讲述的打印图片使用的是support v4 包的PrintHelper类。

##**Print an Image**

原文重点摘要：

The Android Support Library PrintHelper class provides a simple way to print of images.
The class has a single layout option, setScaleMode(), which allows you to print with one of two options:

- SCALE_MODE_FIT - This option sizes the image so that the whole image is shown within the printable area of the page.
- SCALE_MODE_FILL - This option scales the image so that it fills the entire printable area of the page. Choosing this setting means that some portion
of the top and bottom, or left and right edges of the image is not printed. This option is the default value if you do not set a scale mode.

Both scaling options for setScaleMode() keep the existing aspect ratio of the image intact.
The following code example shows how to create an instance of the PrintHelper class, set the scaling option, and start the printing process:

{% highlight ruby %}
private void doPhotoPrint() {
    PrintHelper photoPrinter = new PrintHelper(getActivity());
    photoPrinter.setScaleMode(PrintHelper.SCALE_MODE_FIT);
    Bitmap bitmap = BitmapFactory.decodeResource(getResources(),
            R.drawable.droids);
    photoPrinter.printBitmap("droids.jpg - test print", bitmap);
}
{% endhighlight %}

This method can be called as the action for a menu item.
Note that menu items for actions that are not always supported (such as printing) should be placed in the overflow menu.
For more information, see the Action Bar design guide.

After the printBitmap() method is called, no further action from your application is required.
The Android print user interface appears, allowing the user to select a printer and printing options.
The user can then print the image or cancel the action. If the user chooses to print the image,
a print job is created and a printing notification appears in the system bar.

If you want to include additional content in your printouts beyond just an image,
you must construct a print document. For information on creating documents for printing,
see the Printing an HTML Document or Printing a Custom Document lessons.

精髓指点：

support包提供一个简单的PrintHelper方式进行图片打印。
这个类有一个单一的布局选项，setScaleMode（）方法允许您打印带有如下两个参数选项：

- SCALE_MODE_FIT    这个选项大小使得整个显示在打印区域内的图像被打印出来。
- SCALE_MODE_FILL    此选项缩放图像，使其充满了整个页面的可打印区域。选择此设置意味着某些部分的顶部和底部，或图像的左和右边缘的不打印。
如果你没有设置缩放模式此选项是默认值。

对于setScaleMode两种缩放选项（）都保持了图像的现有宽高比。
上面的代码示例演示如何创建PrintHelper类的实例，设置缩放选项，并启动了打印过程。

上面这种方法可以被称为单项的操作。
注意，对于不支持的动作（例如打印），该菜单项应放置在overflow菜单。欲了解更多信息，请参阅操作栏的设计指南。

在printBitmap（）方法被调用后，需要你的应用程序没有进一步的操作。
因为接下来出现在Android的打印用户界面允许用户选择打印机和打印选项。
然后，用户可以打印该图像或取消该操作。如果用户选择打印图像，打印作业创建和打印通知将出现在系统栏中。

如果你想在您的打印输出额外的内容不仅仅是图像，你必须创建打印文档。有关创建打印文档的信息参考打印HTML文档或打印自定义文档教程。

<hr>

#**Printing HTML Documents**

原文重点摘要：

In Android 4.4 (API level 19), the WebView class has been updated to enable printing HTML content.
The class allows you to load a local HTML resource or download a page from the web,
create a print job and hand it off to Android's print services.

This lesson shows you how to quickly build an HTML document containing text and graphics and use WebView to print it.

精髓指点：

在Android 4.4的WebView已经可以支持打印HTML内容了。这个类可以让你载入本地HTML资源或从网上下载一个网页，
然后创建一个打印任务，并把它关联Android的打印服务。这节课向您展示如何快速构建包含文本和图形的东东，然后使用WebView打印出来。

##**Load an HTML Document**

原文重点摘要：

Printing an HTML document with WebView involves loading an HTML resource or building an HTML document as a string.
This section describes how to build an HTML string and load it into a WebView for printing.

This view object is typically used as part of an activity layout. However, if your application is not using a WebView,
you can create an instance of the class specifically for printing purposes.
The main steps for creating this custom print view are:

- Create a WebViewClient that starts a print job after the HTML resource is loaded.
- Load the HTML resource into the WebView object.

The following code sample demonstrates how to create a simple WebViewClient and load an HTML document created on the fly:

{% highlight ruby %}
private WebView mWebView;

private void doWebViewPrint() {
    // Create a WebView object specifically for printing
    WebView webView = new WebView(getActivity());
    webView.setWebViewClient(new WebViewClient() {

            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                return false;
            }

            @Override
            public void onPageFinished(WebView view, String url) {
                Log.i(TAG, "page finished loading " + url);
                createWebPrintJob(view);
                mWebView = null;
            }
    });

    // Generate an HTML document on the fly:
    String htmlDocument = "<html><body><h1>Test Content</h1><p>Testing, " +
            "testing, testing...</p></body></html>";
    webView.loadDataWithBaseURL(null, htmlDocument, "text/HTML", "UTF-8", null);

    // Keep a reference to WebView object until you pass the PrintDocumentAdapter
    // to the PrintManager
    mWebView = webView;
}
{% endhighlight %}

精髓指点：

通过WebView打印HTML需要一个HTML的String。这里就要讲述如何创建HTML String并且加载入WebView。

此视图对象通常用作布局的一部分。但是，如果你的应用程序没有使用的WebView，你也可以专门针对打印的目的创建类的实例。
创建这个自定义打印视图的主要步骤是：

- 在HTML资源加载以后创建一个WebViewClient开始打印任务。
- 把HTML资源载入WebView里。

上面的代码示例演示了如何创建一个简单的WebViewClient和动态创建实例加载HTML文档的过程。

原文重点摘要：

Note: Make sure your call for generating a print job happens in the onPageFinished() method of the WebViewClient you created in the previous section.
If you don't wait until page loading is finished, the print output may be incomplete or blank, or may fail completely.

Note: The example code above holds an instance of the WebView object so that is it not garbage collected before the print job is created.
Make sure you do the same in your own implementation, otherwise the print process may fail.

If you want to include graphics in the page, place the graphic files in the assets/ directory of your project and specify
a base URL in the first parameter of the loadDataWithBaseURL() method, as shown in the following code example:

{% highlight ruby %}
webView.loadDataWithBaseURL("file:///android_asset/images/", htmlBody,
        "text/HTML", "UTF-8", null);
{% endhighlight %}

You can also load a web page for printing by replacing the loadDataWithBaseURL() method with loadUrl() as shown below.

{% highlight ruby %}
// Print an existing web page (remember to request INTERNET permission!):
webView.loadUrl("http://developer.android.com/about/index.html");
{% endhighlight %}

When using WebView for creating print documents, you should be aware of the following limitations:

- You cannot add headers or footers, including page numbers, to the document.
- The printing options for the HTML document do not include the ability to print page ranges,
for example: Printing page 2 to 4 of a 10 page HTML document is not supported.
- An instance of WebView can only process one print job at a time.
- An HTML document containing CSS print attributes, such as landscape properties, is not supported.
- You cannot use JavaScript in a HTML document to trigger printing.

Note: The content of a WebView object that is included in a layout can also be printed once it has loaded a document.
If you want to create a more customized print output and have complete control of the content draw on the printed page,
jump to the next lesson: Printing a Custom Document lesson.

精髓指点：

请确保您的打印任务就执行在WebViewClient的onPageFinished（）方法中。
如果你不等到onPageFinished的页面加载完成，打印输出就可能不完整或空白、或者完全失败。

如果你想在页面包含一张图片，你可以把他放在assets/目录，然后如上第一段代码设置参数即可。
如果你想打印网络页面，则可以使用如上第二段代码所示。

使用WebView创建打印，你需要关注如下一下限制：

- 你不能添加头尾和页数到文档。
- 不可以打印HTML文档的局部，譬如打印HTML的第二页到第四页等是不支持的。
- 一个WebView实例同时只能执行一个打印任务。
- CSS等属性在HTML里时打印是不支持的。
- JavaScript也是不支持的。

注意：包含在布局中的web视图对象的内容也可以被打印，只要他加载了文档。
如果你想创建一个更加个性化的打印输出，跳到下一课：打印自定义文档的教学。

##**Create a Print Job**

原文重点摘要：

After creating a WebView and loading your HTML content, your application is almost done with its part of the printing process.
The next steps are accessing the PrintManager, creating a print adapter, and finally, creating a print job.
The following example illustrates how to perform these steps:

{% highlight ruby %}
private void createWebPrintJob(WebView webView) {

    // Get a PrintManager instance
    PrintManager printManager = (PrintManager) getActivity()
            .getSystemService(Context.PRINT_SERVICE);

    // Get a print adapter instance
    PrintDocumentAdapter printAdapter = webView.createPrintDocumentAdapter();

    // Create a print job with name and adapter instance
    String jobName = getString(R.string.app_name) + " Document";
    PrintJob printJob = printManager.print(jobName, printAdapter,
            new PrintAttributes.Builder().build());

    // Save the job object for later status checking
    mPrintJobs.add(printJob);
}
{% endhighlight %}

This example saves an instance of the PrintJob object for use by the application,
which is not required. Your application may use this object to track the progress of the print job as it's being processed.
This approach is useful when you want to monitor the status of the print job in you application for completion, failure, or user cancellation.
Creating an in-app notification is not required, because the print framework automatically creates a system notification for the print job.

精髓指点：

上面的创建WebView和加载HTML过程结束以后的任务就是创建PrintManager和print adapter，然后创建打印任务。
如上代码展示了如何执行这一步操作。

应用程序可以使用PrintJob对象来跟踪打印任务的进度，你可以监视完成，失败，或用户注销应用程序的打印任务的状态，这个方法是非常有用的。

创建一个应用程序通知不是必需的，因为在打印框架里系统会自动创建用于打印作业的通知。

<hr>

#**Printing Custom Documents**

##**Connect to the Print Manager**

原文重点摘要：

When your application manages the printing process directly,
the first step after receiving a print request from your user is to connect to the Android print framework and obtain an instance of the PrintManager class.
This class allows you to initialize a print job and begin the printing lifecycle.
The following code example shows how to get the print manager and start the printing process.

{% highlight ruby %}
private void doPrint() {
    // Get a PrintManager instance
    PrintManager printManager = (PrintManager) getActivity()
            .getSystemService(Context.PRINT_SERVICE);

    // Set job name, which will be displayed in the print queue
    String jobName = getActivity().getString(R.string.app_name) + " Document";

    // Start a print job, passing in a PrintDocumentAdapter implementation
    // to handle the generation of a print document
    printManager.print(jobName, new MyPrintDocumentAdapter(getActivity()),
            null); //
}
{% endhighlight %}

The example code above demonstrates how to name a print job and set an instance of the PrintDocumentAdapter class which handles the steps of the printing lifecycle.
The implementation of the print adapter class is discussed in the next section.

Note: The last parameter in the print() method takes a PrintAttributes object.
You can use this parameter to provide hints to the printing framework and pre-set options based on the previous printing cycle,
thereby improving the user experience. You may also use this parameter to set options that are more appropriate to the content being printed,
such as setting the orientation to landscape when printing a photo that is in that orientation.

精髓指点：



##**Create a Print Adapter**

原文重点摘要：

A print adapter interacts with the Android print framework and handles the steps of the printing process.
This process requires users to select printers and print options before creating a document for printing.
These selections can influence the final output as the user chooses printers with different output capabilities,
different page sizes, or different page orientations.
As these selections are made, the print framework asks your adapter to lay out and generate a print document,
in preparation for final output. Once a user taps the print button, the framework takes the final print document and passes it to a print provider for output.
During the printing process, users can choose to cancel the print action, so your print adapter must also listen for and react to a cancellation requests.

The PrintDocumentAdapter abstract class is designed to handle the printing lifecycle,
which has four main callback methods. You must implement these methods in your print adapter in order to interact properly with the print framework:

- onStart() - Called once at the beginning of the print process. If your application has any one-time preparation tasks to perform,
such as getting a snapshot of the data to be printed, execute them here. Implementing this method in your adapter is not required.
- onLayout() - Called each time a user changes a print setting which impacts the output, such as a different page size, or page orientation,
giving your application an opportunity to compute the layout of the pages to be printed. At the minimum,
this method must return how many pages are expected in the printed document.
- onWrite() - Called to render printed pages into a file to be printed. This method may be called one or more times after each onLayout() call.
- onFinish() - Called once at the end of the print process. If your application has any one-time tear-down tasks to perform, execute them here.
Implementing this method in your adapter is not required.

The following sections describe how to implement the layout and write methods, which are critical to the functioning of a print adapter.

Note: These adapter methods are called on the main thread of your application.
If you expect the execution of these methods in your implementation to take a significant amount of time, implement them to execute within a separate thread.
For example, you can encapsulate the layout or print document writing work in separate AsyncTask objects.

精髓指点：



####**Compute print document info**

原文重点摘要：

Within an implementation of the PrintDocumentAdapter class, your application must be able to specify the type of document it is creating and
calculate the total number of pages for print job, given information about the printed page size. The implementation of the onLayout() method in
the adapter makes these calculations and provides information about the expected output of the print job in a PrintDocumentInfo class,
including the number of pages and content type. The following code example shows a basic implementation of the onLayout() method for a PrintDocumentAdapter:

{% highlight ruby %}
@Override
public void onLayout(PrintAttributes oldAttributes,
                     PrintAttributes newAttributes,
                     CancellationSignal cancellationSignal,
                     LayoutResultCallback callback,
                     Bundle metadata) {
    // Create a new PdfDocument with the requested page attributes
    mPdfDocument = new PrintedPdfDocument(getActivity(), newAttributes);

    // Respond to cancellation request
    if (cancellationSignal.isCancelled() ) {
        callback.onLayoutCancelled();
        return;
    }

    // Compute the expected number of printed pages
    int pages = computePageCount(newAttributes);

    if (pages > 0) {
        // Return print information to print framework
        PrintDocumentInfo info = new PrintDocumentInfo
                .Builder("print_output.pdf")
                .setContentType(PrintDocumentInfo.CONTENT_TYPE_DOCUMENT)
                .setPageCount(pages);
                .build();
        // Content layout reflow is complete
        callback.onLayoutFinished(info, true);
    } else {
        // Otherwise report an error to the print framework
        callback.onLayoutFailed("Page count calculation failed.");
    }
}
{% endhighlight %}

The execution of onLayout() method can have three outcomes: completion, cancellation, or failure in the case where calculation of the layout cannot be completed.
You must indicate one of these results by calling the appropriate method of the PrintDocumentAdapter.LayoutResultCallback object.

Note: The boolean parameter of the onLayoutFinished() method indicates whether or not the layout content has actually changed since the last request.
Setting this parameter properly allows the print framework to avoid unnecessarily calling the onWrite() method,
essentially caching the previously written print document and improving performance.

The main work of onLayout() is calculating the number of pages that are expected as output given the attributes of the printer.
How you calculate this number is highly dependent on how your application lays out pages for printing.
The following code example shows an implementation where the number of pages is determined by the print orientation:

{% highlight ruby %}
private int computePageCount(PrintAttributes printAttributes) {
    int itemsPerPage = 4; // default item count for portrait mode

    MediaSize pageSize = printAttributes.getMediaSize();
    if (!pageSize.isPortrait()) {
        // Six items per page in landscape orientation
        itemsPerPage = 6;
    }

    // Determine number of print items
    int printItemCount = getPrintItemCount();

    return (int) Math.ceil(printItemCount / itemsPerPage);
}
{% endhighlight %}

精髓指点：

####**Write a print document file**

原文重点摘要：

When it is time to write print output to a file, the Android print framework calls the onWrite() method of your application's PrintDocumentAdapter class.
The method's parameters specify which pages should be written and the output file to be used.
Your implementation of this method must then render each requested page of content to a multi-page PDF document file.
When this process is complete, you call the onWriteFinished() method of the callback object.

Note: The Android print framework may call the onWrite() method one or more times for every call to onLayout(). For this reason,
it is important to set the boolean parameter of onLayoutFinished() method to false when the print content layout has not changed,
to avoid unnecessary re-writes of the print document.

Note: The boolean parameter of the onLayoutFinished() method indicates whether or not the layout content has actually changed since the last request.
Setting this parameter properly allows the print framework to avoid unnecessarily calling the onLayout() method,
essentially caching the previously written print document and improving performance.

The following sample demonstrates the basic mechanics of this process using the PrintedPdfDocument class to create a PDF file:

{% highlight ruby %}
@Override
public void onWrite(final PageRange[] pageRanges,
                    final ParcelFileDescriptor destination,
                    final CancellationSignal cancellationSignal,
                    final WriteResultCallback callback) {
    // Iterate over each page of the document,
    // check if it's in the output range.
    for (int i = 0; i < totalPages; i++) {
        // Check to see if this page is in the output range.
        if (containsPage(pageRanges, i)) {
            // If so, add it to writtenPagesArray. writtenPagesArray.size()
            // is used to compute the next output page index.
            writtenPagesArray.append(writtenPagesArray.size(), i);
            PdfDocument.Page page = mPdfDocument.startPage(i);

            // check for cancellation
            if (cancellationSignal.isCancelled()) {
                callback.onWriteCancelled();
                mPdfDocument.close();
                mPdfDocument = null;
                return;
            }

            // Draw page content for printing
            drawPage(page);

            // Rendering is complete, so page can be finalized.
            mPdfDocument.finishPage(page);
        }
    }

    // Write PDF document to file
    try {
        mPdfDocument.writeTo(new FileOutputStream(
                destination.getFileDescriptor()));
    } catch (IOException e) {
        callback.onWriteFailed(e.toString());
        return;
    } finally {
        mPdfDocument.close();
        mPdfDocument = null;
    }
    PageRange[] writtenPages = computeWrittenPages();
    // Signal the print framework the document is complete
    callback.onWriteFinished(writtenPages);

    ...
}
{% endhighlight %}

This sample delegates rendering of PDF page content to drawPage() method, which is discussed in the next section.

As with layout, execution of onWrite() method can have three outcomes: completion, cancellation, or failure in the case where the the content cannot be written.
You must indicate one of these results by calling the appropriate method of the PrintDocumentAdapter.WriteResultCallback object.

Note: Rendering a document for printing can be a resource-intensive operation. In order to avoid blocking the main user interface thread of your application,
you should consider performing the page rendering and writing operations on a separate thread, for example in an AsyncTask.
For more information about working with execution threads like asynchronous tasks, see Processes and Threads.

精髓指点：

##**Drawing PDF Page Content**

原文重点摘要：

When your application prints, your application must generate a PDF document and pass it to the Android print framework for printing.
You can use any PDF generation library for this purpose. This lesson shows how to use the PrintedPdfDocument class to generate PDF pages from your content.

The PrintedPdfDocument class uses a Canvas object to draw elements on an PDF page, similar to drawing on an activity layout.
You can draw elements on the printed page using the Canvas draw methods.
The following example code demonstrates how to draw some simple elements on a PDF document page using these methods:

{% highlight ruby %}
private void drawPage(PdfDocument.Page page) {
    Canvas canvas = page.getCanvas();

    // units are in points (1/72 of an inch)
    int titleBaseLine = 72;
    int leftMargin = 54;

    Paint paint = new Paint();
    paint.setColor(Color.BLACK);
    paint.setTextSize(36);
    canvas.drawText("Test Title", leftMargin, titleBaseLine, paint);

    paint.setTextSize(11);
    canvas.drawText("Test paragraph", leftMargin, titleBaseLine + 25, paint);

    paint.setColor(Color.BLUE);
    canvas.drawRect(100, 100, 172, 172, paint);
}
{% endhighlight %}

When using Canvas to draw on a PDF page, elements are specified in points, which is 1/72 of an inch.
Make sure you use this unit of measure for specifying the size of elements on the page. For positioning of drawn elements,
the coordinate system starts at 0,0 for the top left corner of the page.

Tip: While the Canvas object allows you to place print elements on the edge of a PDF document,
many printers are not able to print to the edge of a physical piece of paper.
Make sure that you account for the unprintable edges of the page when you build a print document with this class.

精髓指点：


