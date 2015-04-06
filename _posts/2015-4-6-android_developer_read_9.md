---
layout: post
keywords: Android Developer
description: Android Developer读书笔记
title: "[Training] Displaying Graphics with OpenGL ES"
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

##**Building an OpenGL ES Environment**

原文重点摘要：

In order to draw graphics with OpenGL ES in your Android application, you must create a view container for them.
One of the more straight-forward ways to do this is to implement both a GLSurfaceView and a GLSurfaceView.Renderer.
A GLSurfaceView is a view container for graphics drawn with OpenGL and GLSurfaceView.Renderer controls what is drawn within that view.
For more information about these classes, see the OpenGL ES developer guide.

GLSurfaceView is just one way to incorporate OpenGL ES graphics into your application.
For a full-screen or near-full screen graphics view, it is a reasonable choice.
Developers who want to incorporate OpenGL ES graphics in a small portion of their layouts should take a look at TextureView.
For real, do-it-yourself developers, it is also possible to build up an OpenGL ES view using SurfaceView, but this requires writing quite a bit of additional code.

This lesson explains how to complete a minimal implementation of GLSurfaceView and GLSurfaceView.Renderer in a simple application activity.

精髓指点：

使用OpenGL ES绘制图像前你必须创建一个View容器给他们。最直接的一种办法就是继承实现GLSurfaceView和GLSurfaceView.Renderer。
GLSurfaceView是使用OpenGL绘制的一个View容器，GLSurfaceView.Renderer用来控制View上面画啥。

GLSurfaceView是唯一一个能够结合OpenGL ES到你App的方法。
对于一个全屏或者接近全屏的View来说，使用它是一个不错的选择。
开发者想使用OpenGL ES处理一些局部效果可以使用TextureView。其实如果你足够牛逼，作为一个真真的开发者你应该使用SurfaceView来进行OpenGL ES绘制，但是这个过程需要非常多的
代码。本节内容只是展示了常规的OpenGL ES使用方法。

##**Declare OpenGL ES Use in the Manifest**

原文重点摘要：

In order for your application to use the OpenGL ES 2.0 API, you must add the following declaration to your manifest:

{% highlight ruby %}
<uses-feature android:glEsVersion="0x00020000" android:required="true" />
{% endhighlight %}

If your application uses texture compression, you must also declare which compression formats your app supports, so that it is only installed on compatible devices.

{% highlight ruby %}
<supports-gl-texture android:name="GL_OES_compressed_ETC1_RGB8_texture" />
<supports-gl-texture android:name="GL_OES_compressed_paletted_texture" />
{% endhighlight %}

精髓指点：

如果你想使用OpenGL ES 2.0 API就必须如第一段代码一样进行权限声明。如果还想使用纹理压缩效果，你必须再添加如第二段代码所示的声明。这样声明之后的应用只会被安装在兼容的设备上。

##**Create an Activity for OpenGL ES Graphics**

原文重点摘要：

Android applications that use OpenGL ES have activities just like any other application that has a user interface.
The main difference from other applications is what you put in the layout for your activity.
While in many applications you might use TextView, Button and ListView, in an app that uses OpenGL ES, you can also add a GLSurfaceView.

The following code example shows a minimal implementation of an activity that uses a GLSurfaceView as its primary view:

{% highlight ruby %}
public class OpenGLES20Activity extends Activity {

    private GLSurfaceView mGLView;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // Create a GLSurfaceView instance and set it
        // as the ContentView for this Activity.
        mGLView = new MyGLSurfaceView(this);
        setContentView(mGLView);
    }
}
{% endhighlight %}

Note: OpenGL ES 2.0 requires Android 2.2 (API Level 8) or higher, so make sure your Android project targets that API or higher.

精髓指点：

使用OpenGL ES的Activity和正常的其他使用Activity差不多。其实最主要的区别就是Activity的layout上放置的View不同，在使用OpenGL ES的layout上你还可以添加一个
GLSurfaceView。如上代码就是一个实例。切记，OpenGL ES 2.0只有在Android 2.2以上的设备才被支持。

##**Build a GLSurfaceView Object**

原文重点摘要：

A GLSurfaceView is a specialized view where you can draw OpenGL ES graphics. It does not do much by itself.
The actual drawing of objects is controlled in the GLSurfaceView.Renderer that you set on this view.
In fact, the code for this object is so thin, you may be tempted to skip extending it and just create an unmodified GLSurfaceView instance,
but don’t do that. You need to extend this class in order to capture touch events, which is covered in the Responding to Touch Events lesson.

The essential code for a GLSurfaceView is minimal, so for a quick implementation, it is common to just create an inner class in the activity that uses it:

{% highlight ruby %}
class MyGLSurfaceView extends GLSurfaceView {

    private final MyGLRenderer mRenderer;

    public MyGLSurfaceView(Context context){
        super(context);

        // Create an OpenGL ES 2.0 context
        setEGLContextClientVersion(2);

        mRenderer = new MyGLRenderer();

        // Set the Renderer for drawing on the GLSurfaceView
        setRenderer(mRenderer);
    }
}
{% endhighlight %}

One other optional addition to your GLSurfaceView implementation is to set the render mode to only draw the view when there is
a change to your drawing data using the GLSurfaceView.RENDERMODE_WHEN_DIRTY setting:

{% highlight ruby %}
// Render the view only when there is a change in the drawing data
setRenderMode(GLSurfaceView.RENDERMODE_WHEN_DIRTY);
{% endhighlight %}

This setting prevents the GLSurfaceView frame from being redrawn until you call requestRender(), which is more efficient for this sample app.

精髓指点：

GLSurfaceView是一个专业绘制OpenGL ES的View，他自身不需要做太多操作。真实的绘制是由你View里的GLSurfaceView.Renderer来控制的。
如上代码是为了处理触摸事件才进行继承的。

##**Build a Renderer Class**

原文重点摘要：

The implementation of the GLSurfaceView.Renderer class, or renderer, within an application that uses OpenGL ES is where things start to get interesting.
This class controls what gets drawn on the GLSurfaceView with which it is associated.
There are three methods in a renderer that are called by the Android system in order to figure out what and how to draw on a GLSurfaceView:

- onSurfaceCreated() - Called once to set up the view's OpenGL ES environment.
- onDrawFrame() - Called for each redraw of the view.
- onSurfaceChanged() - Called if the geometry of the view changes, for example when the device's screen orientation changes.

Here is a very basic implementation of an OpenGL ES renderer, that does nothing more than draw a black background in the GLSurfaceView:

{% highlight ruby %}
public class MyGLRenderer implements GLSurfaceView.Renderer {

    public void onSurfaceCreated(GL10 unused, EGLConfig config) {
        // Set the background frame color
        GLES20.glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
    }

    public void onDrawFrame(GL10 unused) {
        // Redraw background color
        GLES20.glClear(GLES20.GL_COLOR_BUFFER_BIT);
    }

    public void onSurfaceChanged(GL10 unused, int width, int height) {
        GLES20.glViewport(0, 0, width, height);
    }
}
{% endhighlight %}

That’s all there is to it! The code examples above create a simple Android application that displays a black screen using OpenGL.
While this code does not do anything very interesting, by creating these classes,
you have laid the foundation you need to start drawing graphic elements with OpenGL.

Note: You may wonder why these methods have a GL10 parameter, when you are using the OpengGL ES 2.0 APIs.
These method signatures are simply reused for the 2.0 APIs to keep the Android framework code simpler.

If you are familiar with the OpenGL ES APIs, you should now be able to set up a OpenGL ES environment in your app and start drawing graphics.
However, if you need a bit more help getting started with OpenGL, head on to the next lessons for a few more hints.

精髓指点：

GLSurfaceView.Renderer接口有三个方法需要实现，系统会回调这三个方法，这三个方法用来计算如何绘制GLSurfaceView。

- onSurfaceCreated()  被调用一次，用来设置环境变量。
- onDrawFrame()  每次重绘View都会调用。
- onSurfaceChanged()  当View的几何尺寸发生变化时调运，譬如旋转屏幕等。

如上所有代码组合起来就是一个最简单的可运行的基于Open GL ES的App，其实就是一个黑屏界面。

<hr>

#**Defining Shapes**

原文重点摘要：

Being able to define shapes to be drawn in the context of an OpenGL ES view is the first step in creating your high-end graphics masterpiece.
Drawing with OpenGL ES can be a little tricky without knowing a few basic things about how OpenGL ES expects you to define graphic objects.

This lesson explains the OpenGL ES coordinate system relative to an Android device screen, the basics of defining a shape, shape faces,
as well as defining a triangle and a square.

精髓指点：

在不了解OpenGL ES情况下绘制OpenGL ES图形是有些棘手的。这节课解释了OpenGL ES相对于Android屏幕的坐标，定义形状，形状面的基础知识，以及定义一个三角形和正方形。

##**Define a Triangle**

原文重点摘要：

OpenGL ES allows you to define drawn objects using coordinates in three-dimensional space.
So, before you can draw a triangle, you must define its coordinates. In OpenGL,
the typical way to do this is to define a vertex array of floating point numbers for the coordinates.
For maximum efficiency, you write these coordinates into a ByteBuffer, that is passed into the OpenGL ES graphics pipeline for processing.

{% highlight ruby %}
public class Triangle {

    private FloatBuffer vertexBuffer;

    // number of coordinates per vertex in this array
    static final int COORDS_PER_VERTEX = 3;
    static float triangleCoords[] = {   // in counterclockwise order:
             0.0f,  0.622008459f, 0.0f, // top
            -0.5f, -0.311004243f, 0.0f, // bottom left
             0.5f, -0.311004243f, 0.0f  // bottom right
    };

    // Set color with red, green, blue and alpha (opacity) values
    float color[] = { 0.63671875f, 0.76953125f, 0.22265625f, 1.0f };

    public Triangle() {
        // initialize vertex byte buffer for shape coordinates
        ByteBuffer bb = ByteBuffer.allocateDirect(
                // (number of coordinate values * 4 bytes per float)
                triangleCoords.length * 4);
        // use the device hardware's native byte order
        bb.order(ByteOrder.nativeOrder());

        // create a floating point buffer from the ByteBuffer
        vertexBuffer = bb.asFloatBuffer();
        // add the coordinates to the FloatBuffer
        vertexBuffer.put(triangleCoords);
        // set the buffer to read the first coordinate
        vertexBuffer.position(0);
    }
}
{% endhighlight %}

By default, OpenGL ES assumes a coordinate system where [0,0,0] (X,Y,Z) specifies the center of the GLSurfaceView frame, [1,1,0]
is the top right corner of the frame and [-1,-1,0] is bottom left corner of the frame. For an illustration of this coordinate system,
see the OpenGL ES developer guide.

Note that the coordinates of this shape are defined in a counterclockwise order.
The drawing order is important because it defines which side is the front face of the shape,
which you typically want to have drawn, and the back face, which you can choose to not draw using the OpenGL ES cull face feature.
For more information about faces and culling, see the OpenGL ES developer guide.

精髓指点：

OpenGL ES允许你定义绘图三维坐标。所以在你绘制一个三角形之前你必须先定义它的坐标，在OpenGL中典型的做法就是定义一个浮点型的坐标数组。
最高效率的做法就是把这些坐标写入ByteBuffer，然后直接传递到OpenGL ES图形管道进行处理。

默认情况下OpenGL ES把GLSurfaceView的中心点当作[0,0,0]，右上角是[1,1,0]，左下角是[-1,-1,0]。具体图形表示参见OpenGL ES developer guide。

这里的图形的坐标定义是逆时针顺序的。绘制的顺序是非常重要的，因为它定义了哪边是图形的前面、后面等等的信息。

##**Define a Square**

原文重点摘要：

Defining triangles is pretty easy in OpenGL, but what if you want to get a just a little more complex?
Say, a square? There are a number of ways to do this, but a typical path to drawing such a shape in OpenGL ES is to use two triangles drawn together:

<img src="http://yanbober.github.io/image/ad/2.png" />

Again, you should define the vertices in a counterclockwise order for both triangles that represent this shape, and put the values in a ByteBuffer.
In order to avoid defining the two coordinates shared by each triangle twice, use a drawing list to tell the OpenGL ES graphics pipeline how to draw these vertices.
Here’s the code for this shape:

{% highlight ruby %}
public class Square {

    private FloatBuffer vertexBuffer;
    private ShortBuffer drawListBuffer;

    // number of coordinates per vertex in this array
    static final int COORDS_PER_VERTEX = 3;
    static float squareCoords[] = {
            -0.5f,  0.5f, 0.0f,   // top left
            -0.5f, -0.5f, 0.0f,   // bottom left
             0.5f, -0.5f, 0.0f,   // bottom right
             0.5f,  0.5f, 0.0f }; // top right

    private short drawOrder[] = { 0, 1, 2, 0, 2, 3 }; // order to draw vertices

    public Square() {
        // initialize vertex byte buffer for shape coordinates
        ByteBuffer bb = ByteBuffer.allocateDirect(
        // (# of coordinate values * 4 bytes per float)
                squareCoords.length * 4);
        bb.order(ByteOrder.nativeOrder());
        vertexBuffer = bb.asFloatBuffer();
        vertexBuffer.put(squareCoords);
        vertexBuffer.position(0);

        // initialize byte buffer for the draw list
        ByteBuffer dlb = ByteBuffer.allocateDirect(
        // (# of coordinate values * 2 bytes per short)
                drawOrder.length * 2);
        dlb.order(ByteOrder.nativeOrder());
        drawListBuffer = dlb.asShortBuffer();
        drawListBuffer.put(drawOrder);
        drawListBuffer.position(0);
    }
}
{% endhighlight %}

This example gives you a peek at what it takes to create more complex shapes with OpenGL.
In general, you use collections of triangles to draw objects. In the next lesson, you learn how to draw these shapes on screen.

精髓指点：

定义一个三角形非常easy，如果你想复杂点可以绘制一个矩形，典型的OpenGL ES绘制矩形做法是画两个三角形，如上图所示。
一样的模式，你需要逆时针方向定义两个三角形坐标然后放入ByteBuffer。防止绘制多次我们可以创建一个绘制列表告诉OpenGL ES绘制这些点的顺序。

如上代码就是一个矩形的绘制坐标定义。

<hr>

#**Drawing Shapes**

原文重点摘要：

After you define shapes to be drawn with OpenGL, you probably want to draw them.
Drawing shapes with the OpenGL ES 2.0 takes a bit more code than you might imagine,
because the API provides a great deal of control over the graphics rendering pipeline.

This lesson explains how to draw the shapes you defined in the previous lesson using the OpenGL ES 2.0 API.

精髓指点：

定义shape后就该drawn了，使用OpenGL ES 2.0绘制图形比你想象中的代码量要多些，因为API提供了大量的图形绘制渲染管道。

##**Initialize Shapes**

原文重点摘要：

Before you do any drawing, you must initialize and load the shapes you plan to draw.
Unless the structure (the original coordinates) of the shapes you use in your program change during the course of execution,
you should initialize them in the onSurfaceCreated() method of your renderer for memory and processing efficiency.

{% highlight ruby %}
public class MyGLRenderer implements GLSurfaceView.Renderer {

    ...
    private Triangle mTriangle;
    private Square   mSquare;

    public void onSurfaceCreated(GL10 unused, EGLConfig config) {
        ...

        // initialize a triangle
        mTriangle = new Triangle();
        // initialize a square
        mSquare = new Square();
    }
    ...
}
{% endhighlight %}

精髓指点：

##**Draw a Shape**

原文重点摘要：

Drawing a defined shape using OpenGL ES 2.0 requires a significant amount of code,
because you must provide a lot of details to the graphics rendering pipeline. Specifically, you must define the following:

- Vertex Shader - OpenGL ES graphics code for rendering the vertices of a shape.
- Fragment Shader - OpenGL ES code for rendering the face of a shape with colors or textures.
- Program - An OpenGL ES object that contains the shaders you want to use for drawing one or more shapes.

You need at least one vertex shader to draw a shape and one fragment shader to color that shape.
These shaders must be complied and then added to an OpenGL ES program, which is then used to draw the shape.
Here is an example of how to define basic shaders you can use to draw a shape in the Triangle class:


{% highlight ruby %}
public class Triangle {

    private final String vertexShaderCode =
        "attribute vec4 vPosition;" +
        "void main() {" +
        "  gl_Position = vPosition;" +
        "}";

    private final String fragmentShaderCode =
        "precision mediump float;" +
        "uniform vec4 vColor;" +
        "void main() {" +
        "  gl_FragColor = vColor;" +
        "}";

    ...
}
{% endhighlight %}

精髓指点：

原文重点摘要：

Shaders contain OpenGL Shading Language (GLSL) code that must be compiled prior to using it in the OpenGL ES environment.
To compile this code, create a utility method in your renderer class:

{% highlight ruby %}
public static int loadShader(int type, String shaderCode){

    // create a vertex shader type (GLES20.GL_VERTEX_SHADER)
    // or a fragment shader type (GLES20.GL_FRAGMENT_SHADER)
    int shader = GLES20.glCreateShader(type);

    // add the source code to the shader and compile it
    GLES20.glShaderSource(shader, shaderCode);
    GLES20.glCompileShader(shader);

    return shader;
}
{% endhighlight %}

精髓指点：

原文重点摘要：

In order to draw your shape, you must compile the shader code, add them to a OpenGL ES program object and then link the program.
Do this in your drawn object’s constructor, so it is only done once.

Note: Compiling OpenGL ES shaders and linking programs is expensive in terms of CPU cycles and processing time, so you should avoid doing this more than once.
If you do not know the content of your shaders at runtime, you should build your code such that they only get created once and then cached for later use.

{% highlight ruby %}
public class Triangle() {
    ...

    private final int mProgram;

    public Triangle() {
        ...

        int vertexShader = MyGLRenderer.loadShader(GLES20.GL_VERTEX_SHADER,
                                        vertexShaderCode);
        int fragmentShader = MyGLRenderer.loadShader(GLES20.GL_FRAGMENT_SHADER,
                                        fragmentShaderCode);

        // create empty OpenGL ES Program
        mProgram = GLES20.glCreateProgram();

        // add the vertex shader to program
        GLES20.glAttachShader(mProgram, vertexShader);

        // add the fragment shader to program
        GLES20.glAttachShader(mProgram, fragmentShader);

        // creates OpenGL ES program executables
        GLES20.glLinkProgram(mProgram);
    }
}
{% endhighlight %}

精髓指点：

原文重点摘要：

At this point, you are ready to add the actual calls that draw your shape.
Drawing shapes with OpenGL ES requires that you specify several parameters to tell the rendering pipeline what you want to draw and how to draw it.
Since drawing options can vary by shape, it's a good idea to have your shape classes contain their own drawing logic.

Create a draw() method for drawing the shape. This code sets the position and color values to the shape’s vertex shader and fragment shader,
and then executes the drawing function.

{% highlight ruby %}
private int mPositionHandle;
private int mColorHandle;

private final int vertexCount = triangleCoords.length / COORDS_PER_VERTEX;
private final int vertexStride = COORDS_PER_VERTEX * 4; // 4 bytes per vertex

public void draw() {
    // Add program to OpenGL ES environment
    GLES20.glUseProgram(mProgram);

    // get handle to vertex shader's vPosition member
    mPositionHandle = GLES20.glGetAttribLocation(mProgram, "vPosition");

    // Enable a handle to the triangle vertices
    GLES20.glEnableVertexAttribArray(mPositionHandle);

    // Prepare the triangle coordinate data
    GLES20.glVertexAttribPointer(mPositionHandle, COORDS_PER_VERTEX,
                                 GLES20.GL_FLOAT, false,
                                 vertexStride, vertexBuffer);

    // get handle to fragment shader's vColor member
    mColorHandle = GLES20.glGetUniformLocation(mProgram, "vColor");

    // Set color for drawing the triangle
    GLES20.glUniform4fv(mColorHandle, 1, color, 0);

    // Draw the triangle
    GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, vertexCount);

    // Disable vertex array
    GLES20.glDisableVertexAttribArray(mPositionHandle);
}
{% endhighlight %}

Once you have all this code in place, drawing this object just requires a call to the draw() method from within your renderer’s onDrawFrame() method:

{% highlight ruby %}
public void onDrawFrame(GL10 unused) {
    ...

    mTriangle.draw();
}
{% endhighlight %}

精髓指点：

原文重点摘要：

When you run the application, it should look something like this:

<img src="http://yanbober.github.io/image/ad/3.png" />

There are a few problems with this code example. First of all, it is not going to impress your friends.
Secondly, the triangle is a bit squashed and changes shape when you change the screen orientation of the device.
The reason the shape is skewed is due to the fact that the object’s vertices have not been corrected for the proportions of the screen area
where the GLSurfaceView is displayed. You can fix that problem using a projection and camera view in the next lesson.

Lastly, the triangle is stationary, which is a bit boring. In the Adding Motion lesson,
you make this shape rotate and make more interesting use of the OpenGL ES graphics pipeline.

精髓指点：

<hr>

#**Applying Projection and Camera Views**

原文重点摘要：

In the OpenGL ES environment, projection and camera views allow you to display drawn objects in a way that more closely resembles
how you see physical objects with your eyes. This simulation of physical viewing is done with mathematical transformations of drawn object coordinates:

- Projection - This transformation adjusts the coordinates of drawn objects based on the width and height of the GLSurfaceView where they are displayed.
Without this calculation, objects drawn by OpenGL ES are skewed by the unequal proportions of the view window.
A projection transformation typically only has to be calculated when the proportions of the OpenGL view are established or changed in the onSurfaceChanged()
method of your renderer. For more information about OpenGL ES projections and coordinate mapping, see Mapping Coordinates for Drawn Objects.
- Camera View - This transformation adjusts the coordinates of drawn objects based on a virtual camera position.
It’s important to note that OpenGL ES does not define an actual camera object, but instead provides utility methods that simulate a camera by transforming
the display of drawn objects. A camera view transformation might be calculated only once when you establish your GLSurfaceView,
or might change dynamically based on user actions or your application’s function.

This lesson describes how to create a projection and camera view and apply it to shapes drawn in your GLSurfaceView.

精髓指点：

##**Define a Projection**

原文重点摘要：

The data for a projection transformation is calculated in the onSurfaceChanged() method of your GLSurfaceView.Renderer class.
The following example code takes the height and width of the GLSurfaceView and uses it to populate a projection transformation
Matrix using the Matrix.frustumM() method:

{% highlight ruby %}
// mMVPMatrix is an abbreviation for "Model View Projection Matrix"
private final float[] mMVPMatrix = new float[16];
private final float[] mProjectionMatrix = new float[16];
private final float[] mViewMatrix = new float[16];

@Override
public void onSurfaceChanged(GL10 unused, int width, int height) {
    GLES20.glViewport(0, 0, width, height);

    float ratio = (float) width / height;

    // this projection matrix is applied to object coordinates
    // in the onDrawFrame() method
    Matrix.frustumM(mProjectionMatrix, 0, -ratio, ratio, -1, 1, 3, 7);
}
{% endhighlight %}

This code populates a projection matrix, mProjectionMatrix which you can then combine with a camera view transformation in the onDrawFrame() method,
which is shown in the next section.

Note: Just applying a projection transformation to your drawing objects typically results in a very empty display.
In general, you must also apply a camera view transformation in order for anything to show up on screen.

精髓指点：

##**Define a Camera View**

原文重点摘要：

Complete the process of transforming your drawn objects by adding a camera view transformation as part of the drawing process in your renderer.
In the following example code, the camera view transformation is calculated using the Matrix.setLookAtM() method and then combined with the
previously calculated projection matrix. The combined transformation matrices are then passed to the drawn shape.

{% highlight ruby %}
@Override
public void onDrawFrame(GL10 unused) {
    ...
    // Set the camera position (View matrix)
    Matrix.setLookAtM(mViewMatrix, 0, 0, 0, -3, 0f, 0f, 0f, 0f, 1.0f, 0.0f);

    // Calculate the projection and view transformation
    Matrix.multiplyMM(mMVPMatrix, 0, mProjectionMatrix, 0, mViewMatrix, 0);

    // Draw shape
    mTriangle.draw(mMVPMatrix);
}
{% endhighlight %}

精髓指点：

##**Apply Projection and Camera Transformations**

原文重点摘要：

In order to use the combined projection and camera view transformation matrix shown in the previews sections,
first add a matrix variable to the vertex shader previously defined in the Triangle class:

{% highlight ruby %}
public class Triangle {

    private final String vertexShaderCode =
        // This matrix member variable provides a hook to manipulate
        // the coordinates of the objects that use this vertex shader
        "uniform mat4 uMVPMatrix;" +
        "attribute vec4 vPosition;" +
        "void main() {" +
        // the matrix must be included as a modifier of gl_Position
        // Note that the uMVPMatrix factor *must be first* in order
        // for the matrix multiplication product to be correct.
        "  gl_Position = uMVPMatrix * vPosition;" +
        "}";

    // Use to access and set the view transformation
    private int mMVPMatrixHandle;

    ...
}
{% endhighlight %}

Next, modify the draw() method of your graphic objects to accept the combined transformation matrix and apply it to the shape:

{% highlight ruby %}
public void draw(float[] mvpMatrix) { // pass in the calculated transformation matrix
    ...

    // get handle to shape's transformation matrix
    mMVPMatrixHandle = GLES20.glGetUniformLocation(mProgram, "uMVPMatrix");

    // Pass the projection and view transformation to the shader
    GLES20.glUniformMatrix4fv(mMVPMatrixHandle, 1, false, mvpMatrix, 0);

    // Draw the triangle
    GLES20.glDrawArrays(GLES20.GL_TRIANGLES, 0, vertexCount);

    // Disable vertex array
    GLES20.glDisableVertexAttribArray(mPositionHandle);
}
{% endhighlight %}

Once you have correctly calculated and applied the projection and camera view transformations,
your graphic objects are drawn in correct proportions and should look like this:

<img src="http://yanbober.github.io/image/ad/4.png" />

Now that you have an application that displays your shapes in correct proportions, it's time to add motion to your shapes.

精髓指点：

<hr>

#**Adding Motion**

原文重点摘要：

Drawing objects on screen is a pretty basic feature of OpenGL, but you can do this with other Android graphics framwork classes,
including Canvas and Drawable objects. OpenGL ES provides additional capabilities for moving and transforming drawn objects in
three dimensions or in other unique ways to create compelling user experiences.

In this lesson, you take another step forward into using OpenGL ES by learning how to add motion to a shape with rotation.

精髓指点：

##**Rotate a Shape**

原文重点摘要：

Rotating a drawing object with OpenGL ES 2.0 is relatively simple.
In your renderer, create another transformation matrix (a rotation matrix) and then combine it with your projection and camera view transformation matrices:

{% highlight ruby %}
private float[] mRotationMatrix = new float[16];
public void onDrawFrame(GL10 gl) {
    float[] scratch = new float[16];

    ...

    // Create a rotation transformation for the triangle
    long time = SystemClock.uptimeMillis() % 4000L;
    float angle = 0.090f * ((int) time);
    Matrix.setRotateM(mRotationMatrix, 0, angle, 0, 0, -1.0f);

    // Combine the rotation matrix with the projection and camera view
    // Note that the mMVPMatrix factor *must be first* in order
    // for the matrix multiplication product to be correct.
    Matrix.multiplyMM(scratch, 0, mMVPMatrix, 0, mRotationMatrix, 0);

    // Draw triangle
    mTriangle.draw(scratch);
}
{% endhighlight %}

If your triangle does not rotate after making these changes,
make sure you have commented out the GLSurfaceView.RENDERMODE_WHEN_DIRTY setting, as described in the next section.

精髓指点：

##**Enable Continuous Rendering**

原文重点摘要：

If you have diligently followed along with the example code in this class to this point,
make sure you comment out the line that sets the render mode only draw when dirty,
otherwise OpenGL rotates the shape only one increment and then waits for a call to requestRender() from the GLSurfaceView container:

{% highlight ruby %}
public MyGLSurfaceView(Context context) {
    ...
    // Render the view only when there is a change in the drawing data.
    // To allow the triangle to rotate automatically, this line is commented out:
    //setRenderMode(GLSurfaceView.RENDERMODE_WHEN_DIRTY);
}
{% endhighlight %}

Unless you have objects changing without any user interaction, it’s usually a good idea have this flag turned on.
Be ready to uncomment this code, because the next lesson makes this call applicable once again.

精髓指点：


<hr>

#**Responding to Touch Events**

原文重点摘要：

Making objects move according to a preset program like the rotating triangle is useful for getting some attention,
but what if you want to have users interact with your OpenGL ES graphics? The key to making your OpenGL ES application
touch interactive is expanding your implementation of GLSurfaceView to override the onTouchEvent() to listen for touch events.

This lesson shows you how to listen for touch events to let users rotate an OpenGL ES object.

精髓指点：

##**Setup a Touch Listener**

原文重点摘要：

In order to make your OpenGL ES application respond to touch events, you must implement the onTouchEvent() method in your GLSurfaceView class.
The example implementation below shows how to listen for MotionEvent.ACTION_MOVE events and translate them to an angle of rotation for a shape.

{% highlight ruby %}
private final float TOUCH_SCALE_FACTOR = 180.0f / 320;
private float mPreviousX;
private float mPreviousY;

@Override
public boolean onTouchEvent(MotionEvent e) {
    // MotionEvent reports input details from the touch screen
    // and other input controls. In this case, you are only
    // interested in events where the touch position changed.

    float x = e.getX();
    float y = e.getY();

    switch (e.getAction()) {
        case MotionEvent.ACTION_MOVE:

            float dx = x - mPreviousX;
            float dy = y - mPreviousY;

            // reverse direction of rotation above the mid-line
            if (y > getHeight() / 2) {
              dx = dx * -1 ;
            }

            // reverse direction of rotation to left of the mid-line
            if (x < getWidth() / 2) {
              dy = dy * -1 ;
            }

            mRenderer.setAngle(
                    mRenderer.getAngle() +
                    ((dx + dy) * TOUCH_SCALE_FACTOR));
            requestRender();
    }

    mPreviousX = x;
    mPreviousY = y;
    return true;
}
{% endhighlight %}

精髓指点：

原文重点摘要：

Notice that after calculating the rotation angle, this method calls requestRender() to tell the renderer that it is time to render the frame.
This approach is the most efficient in this example because the frame does not need to be redrawn unless there is a change in the rotation.
However, it does not have any impact on efficiency unless you also request that the renderer only redraw when the data changes using the setRenderMode() method,
so make sure this line is uncommented in the renderer:

{% highlight ruby %}
public MyGLSurfaceView(Context context) {
    ...
    // Render the view only when there is a change in the drawing data
    setRenderMode(GLSurfaceView.RENDERMODE_WHEN_DIRTY);
}
{% endhighlight %}

精髓指点：

##**Expose the Rotation Angle**

原文重点摘要：

The example code above requires that you expose the rotation angle through your renderer by adding a public member.
Since the renderer code is running on a separate thread from the main user interface thread of your application,
you must declare this public variable as volatile. Here is the code to declare the variable and expose the getter and setter pair:

{% highlight ruby %}
public class MyGLRenderer implements GLSurfaceView.Renderer {
    ...

    public volatile float mAngle;

    public float getAngle() {
        return mAngle;
    }

    public void setAngle(float angle) {
        mAngle = angle;
    }
}
{% endhighlight %}

精髓指点：

##**Apply Rotation**

原文重点摘要：

To apply the rotation generated by touch input, comment out the code that generates an angle and add mAngle, which contains the touch input generated angle:

{% highlight ruby %}
public void onDrawFrame(GL10 gl) {
    ...
    float[] scratch = new float[16];

    // Create a rotation for the triangle
    // long time = SystemClock.uptimeMillis() % 4000L;
    // float angle = 0.090f * ((int) time);
    Matrix.setRotateM(mRotationMatrix, 0, mAngle, 0, 0, -1.0f);

    // Combine the rotation matrix with the projection and camera view
    // Note that the mMVPMatrix factor *must be first* in order
    // for the matrix multiplication product to be correct.
    Matrix.multiplyMM(scratch, 0, mMVPMatrix, 0, mRotationMatrix, 0);

    // Draw triangle
    mTriangle.draw(scratch);
}
{% endhighlight %}

精髓指点：

原文重点摘要：

When you have completed the steps described above, run the program and drag your finger over the screen to rotate the triangle:

<img src="http://yanbober.github.io/image/ad/5.png" />

精髓指点：

