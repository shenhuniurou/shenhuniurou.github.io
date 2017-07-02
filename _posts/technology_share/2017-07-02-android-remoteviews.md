---
layout: post
title: Android中的RemoteViews
category: 技术分享
tags: RemoteViews
---



## 概述

RemoteViews顾名思义就是远程View，它表示的是一个View结构，它可以在其他进程中显示，为了跨进程更新它的界面，RemoteViews提供了一组基础的操作来实现这个效果。RemoteViews在Android中的使用场景有两种：通知栏和桌面小部件。

## RemoteViews在通知栏上的应用

我们知道通知栏除了默认的效果外还支持自定义布局。

使用系统默认的样式弹出一个通知的方式如下：（android3.0之后）

```java
private void showDefaultNotification() {
    NotificationCompat.Builder builder = new NotificationCompat.Builder(this);

    // 设置通知的基本信息：icon、标题、内容
    builder.setSmallIcon(R.mipmap.ic_launcher);
    builder.setContentTitle("My notification");
    builder.setContentText("Hello World!");
    builder.setAutoCancel(true);

    // 设置通知的点击行为：这里启动一个 Activity
    Intent intent = new Intent(this, SecondActivity.class);
    PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);
    builder.setContentIntent(pendingIntent);

    // 发送通知 id 需要在应用内唯一
    NotificationManager notificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
    notificationManager.notify(id, builder.build());
}
```

上述代码会弹出一个系统默认样式的通知，单击通知后会打开SecondActivity同时会清除本身。效果如图：

![系统默认样式](http://upload-images.jianshu.io/upload_images/1159224-bcdfb45973c62297.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


为了满足个性化需求，我们还可能会用到自定义通知。实现自定义通知我们首先需要提供一个布局文件，然后通过RemoteViews来加载这个布局文件即可改变通知的样式。

```java
private void showCustomNotification() {

    RemoteViews remoteView;

    // 构建 remoteView
    remoteView = new RemoteViews(getPackageName(), R.layout.layout_notification);
    remoteView.setTextViewText(R.id.tvMsg, "哈shenhuniurou");
    remoteView.setImageViewResource(R.id.ivIcon, R.mipmap.ic_launcher_round);

    NotificationCompat.Builder builder = new NotificationCompat.Builder(this);

    // 设置自定义 RemoteViews
    builder.setContent(remoteView).setSmallIcon(R.mipmap.ic_launcher);

    // 设置通知的优先级(悬浮通知)
    builder.setPriority(NotificationCompat.PRIORITY_MAX);
    Uri alarmSound = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
    // 设置通知的提示音
    builder.setSound(alarmSound);


    // 设置通知的点击行为：这里启动一个 Activity
    Intent intent = new Intent(this, SecondActivity.class);
    PendingIntent pendingIntent = PendingIntent.getActivity(this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);
    builder.setContentIntent(pendingIntent);
    builder.setAutoCancel(true);
    Notification notification = builder.build();

    NotificationManager notificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
    notificationManager.notify(1001, notification);
}
```

效果如图所示：

![自定义样式](http://upload-images.jianshu.io/upload_images/1159224-cae72f5ac1ef438a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

创建RemoteViews对象我们只需要知道当前应用包名和布局文件的资源id，比较简单，但是要更新RemoteViews就不是那么容易了，因为我们无法直接访问布局文件中的View，而必须通过RemoteViews提供的特定的方法来更新View。比如设置TextView文本内容需要用setTextViewText方法，设置ImageView图片需要通过setImageViewResource方法。也可以给里面的View设置点击事件，需要使用PendingIntent并通过setOnClickPendingIntent方法来实现。之所以更新RemoteViews如此复杂，直接原因是因为RemoteViews没有提供跟View类似的findViewById这个方法，我们无法获取到RemoteViews中的子View。


## RemoteViews在桌面小部件上的应用

现在我要实现的效果是这样一个小部件：

![](http://upload-images.jianshu.io/upload_images/1159224-8013bef90323ca28.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

AppWidgetProvider是Android中提供用于实现桌面小部件的类，它的本质其实是一个广播。开发桌面小部件的步骤：

#### 定义小部件布局

在res/layout/下新建一个布局文件layout_widget.xml，内容命名根据需求自定。我在里面放了四个线程布局当做按钮，外面再套一层线性布局横向排列。

```java
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:weightSum="4">

    <LinearLayout
        android:id="@+id/btn1"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:gravity="center"
        android:orientation="vertical">

        <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="5dp"
            android:src="@mipmap/ic_launcher_round" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginBottom="5dp"
            android:layout_marginTop="5dp"
            android:text="按钮1"
            android:textColor="@android:color/white" />

    </LinearLayout>

    <LinearLayout
        android:id="@+id/btn2"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:gravity="center"
        android:orientation="vertical">

        <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="5dp"
            android:src="@mipmap/ic_launcher_round" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginBottom="5dp"
            android:layout_marginTop="5dp"
            android:text="按钮2"
            android:textColor="@android:color/white" />

    </LinearLayout>

    <LinearLayout
        android:id="@+id/btn3"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:gravity="center"
        android:orientation="vertical">

        <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="5dp"
            android:src="@mipmap/ic_launcher_round" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginBottom="5dp"
            android:layout_marginTop="5dp"
            android:text="按钮3"
            android:textColor="@android:color/white" />

    </LinearLayout>

    <LinearLayout
        android:id="@+id/btn4"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:gravity="center"
        android:orientation="vertical">

        <ImageView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="5dp"
            android:src="@mipmap/ic_launcher_round" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginBottom="5dp"
            android:layout_marginTop="5dp"
            android:text="按钮4"
            android:textColor="@android:color/white" />

    </LinearLayout>


</LinearLayout>
```

#### 定义小部件配置信息

在res/xml/下新建一个资源文件，命名自定：

```java
<?xml version="1.0" encoding="utf-8"?>
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
    android:initialLayout="@layout/widget"
    android:minHeight="56dp"
    android:minWidth="272dp"
    android:previewImage="@mipmap/ic_launcher"
    android:resizeMode="horizontal|vertical"
    android:updatePeriodMillis="100000"
    android:widgetCategory="home_screen">

</appwidget-provider>
```

解释下各个属性的含义
android:initialLayout：指定小部件的初始化布局
android:minHeight：小部件最小高度
android:minWidth：小部件最小宽度
android:previewImage：小部件列表显示的图标
android:updatePeriodMillis：小部件自动更新的周期
android:widgetCategory：小部件显示的位置，home_screen表示只在桌面上显示

#### 定义小部件的实现类

```java
package com.shenhuniurou.remoteviewsdemo;

import android.app.PendingIntent;
import android.appwidget.AppWidgetManager;
import android.appwidget.AppWidgetProvider;
import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.widget.RemoteViews;

/**
 * Created by Daniel on 2017/7/1.
 */

public class CustomAppWidgetProvider extends AppWidgetProvider {

    public static final String CLICK_WEDGET_ONE = "com.shenhuniurou.appwidgetprovider.click.one";

    public static final String CLICK_WEDGET_TWO = "com.shenhuniurou.appwidgetprovider.click.two";

    public static final String CLICK_WEDGET_THREE = "com.shenhuniurou.appwidgetprovider.click.three";

    public static final String CLICK_WEDGET_FOUR = "com.shenhuniurou.appwidgetprovider.click.four";


    public CustomAppWidgetProvider() {
        super();
    }

    @Override
    public void onReceive(final Context context, Intent intent) {
        super.onReceive(context, intent);

        String action = intent.getAction();

        AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(context);

        RemoteViews remoteViews = new RemoteViews(context.getPackageName(), R.layout.widget);

        // 判断action是否是自己定义的action
        if (action.equals(CLICK_WEDGET_ONE)) {

            // 点击的是第一个按钮
            Intent firstIntent = Intent.makeRestartActivityTask(new ComponentName(context, MainActivity.class));
            Intent secondIntent = new Intent(context, SecondActivity.class);
            PendingIntent pendingIntent = PendingIntent.getActivities(context, 0, new Intent[] { firstIntent, secondIntent }, PendingIntent.FLAG_UPDATE_CURRENT);
            remoteViews.setOnClickPendingIntent(R.id.btn1, pendingIntent);

        } else if (action.equals(CLICK_WEDGET_TWO)) {

            // 点击的是第二个按钮
            Intent clickIntent = new Intent(context, ThirdActivity.class);
            PendingIntent pendingIntent = PendingIntent.getActivity(context, 0, clickIntent, 0);
            remoteViews.setOnClickPendingIntent(R.id.btn2, pendingIntent);

        } else if (action.equals(CLICK_WEDGET_THREE)) {

            // 点击的是第三个按钮
            Intent clickIntent = new Intent(context, ForthActivity.class);
            PendingIntent pendingIntent = PendingIntent.getActivity(context, 0, clickIntent, 0);
            remoteViews.setOnClickPendingIntent(R.id.btn3, pendingIntent);

        } else if (action.equals(CLICK_WEDGET_FOUR)) {

            // 点击的是第四个按钮
            Intent clickIntent = new Intent(context, FifthActivity.class);
            PendingIntent pendingIntent = PendingIntent.getActivity(context, 0, clickIntent, 0);
            remoteViews.setOnClickPendingIntent(R.id.btn4, pendingIntent);

        }

        appWidgetManager.updateAppWidget(new ComponentName(context, CustomAppWidgetProvider.class), remoteViews);

    }

    /**
     * 桌面小部件每次更新时调用的方法
     * @param context
     * @param appWidgetManager
     * @param appWidgetIds
     */
    @Override
    public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
        super.onUpdate(context, appWidgetManager, appWidgetIds);

        int count = appWidgetIds.length;
        for (int i = 0; i < count; i++) {
            int appWidgetId = appWidgetIds[i];
            onWidgetUpdate(context, appWidgetManager, appWidgetId);
        }
    }


    /**
     * 更新桌面小部件
     * @param context
     * @param appWidgetManager
     * @param appWidgetId
     */
    private void onWidgetUpdate(Context context, AppWidgetManager appWidgetManager, int appWidgetId) {

        RemoteViews remoteViews = new RemoteViews(context.getPackageName(), R.layout.widget);

        Intent intent1 = new Intent(CLICK_WEDGET_ONE);
        PendingIntent pendingIntent1 = PendingIntent.getBroadcast(context, 0, intent1, PendingIntent.FLAG_UPDATE_CURRENT);
        remoteViews.setOnClickPendingIntent(R.id.btn1, pendingIntent1);

        Intent intent2 = new Intent(CLICK_WEDGET_TWO);
        PendingIntent pendingIntent2 = PendingIntent.getBroadcast(context, 0, intent2, PendingIntent.FLAG_UPDATE_CURRENT);
        remoteViews.setOnClickPendingIntent(R.id.btn2, pendingIntent2);

        Intent intent3 = new Intent(CLICK_WEDGET_THREE);
        PendingIntent pendingIntent3 = PendingIntent.getBroadcast(context, 0, intent3, PendingIntent.FLAG_UPDATE_CURRENT);
        remoteViews.setOnClickPendingIntent(R.id.btn3, pendingIntent3);

        Intent intent4 = new Intent(CLICK_WEDGET_FOUR);
        PendingIntent pendingIntent4 = PendingIntent.getBroadcast(context, 0, intent4, PendingIntent.FLAG_UPDATE_CURRENT);
        remoteViews.setOnClickPendingIntent(R.id.btn4, pendingIntent4);

        appWidgetManager.updateAppWidget(appWidgetId, remoteViews);
    }

}
```

#### 在清单文件上声明小部件

```java
<receiver android:name=".CustomAppWidgetProvider">
    <meta-data
        android:name="android.appwidget.provider"
        android:resource="@xml/app_widget_provider_info" />

    <intent-filter>
        <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
        <action android:name="com.shenhuniurou.appwidgetprovider.click.one" />
        <action android:name="com.shenhuniurou.appwidgetprovider.click.two" />
        <action android:name="com.shenhuniurou.appwidgetprovider.click.three" />
        <action android:name="com.shenhuniurou.appwidgetprovider.click.four" />
    </intent-filter>
</receiver>
```

这里面meta-data标签中的name属性是固定的`android.appwidget.provider`，而resource属性则是我们刚才新建的小部件的配置信息的xml，intent-filter中的android.appwidget.action.APPWIDGET_UPDATE是必须加的，它作为小部件的标识存在，这是系统的规范，否则这个receiver就不是一个桌面小部件，并且也无法出现在手机的小部件列表里。下面其他的action分别对应各个按钮点击的动作。

最后实现的效果图：

![运行效果图](http://upload-images.jianshu.io/upload_images/1159224-94d250a9ba6570c8.gif?imageMogr2/auto-orient/strip)



总结下这个操作过程：当小部件一被添加到桌面时会调用Provider中的onUpdate方法，在这个方法中我们会通过AppWidgetManager去更新小部件的界面，但是这个更新我们是没办法直接更新的，而是通过RemoteViews来操作，setOnClickPendingIntent给每个按钮设置了点击时会发送的广播动作，而在清单文件中我们声明小部件时已经将这些广播动作都加到intent-filter，所以当我们点击桌面上该小部件中的某个按钮时，就会发送对应的广播，而小部件监听了这个广播，接收到广播后再onReceive方法中根据动作来分别处理点击事件。当然，对小部件的一些其他操作方法（比如onEnabled、onDisabled、onDeleted）的广播也会在onReceive中接收到，然后分发给不同的方法。(我这里处理点击事件用的也是RemoteViews的方式，其实不必，直接使用context.startActivity即可，但如果不是打开页面，而是要更新小部件的界面，那么就需要继续使用RemoteViews来更新了。)


## PendingIntent

在上面实现小部件时我们多次使用到了PendingIntent，这个东西顾名思义我们可以理解为将要发生的意图，就是在某个待定的时刻会发生。所以它和Intent的区别就在于一个是立即执行的一个是在未来某个时候执行。PendingIntent典型的使用场景是通知中点击通知时跳转页面，因为我们不知道用户什么时候点击，另外就是给RemoteViews添加单击事件，因为RemoteViews运行在远程进程中，所以它不同于普通的View，不能想View那样通过setOnClickListener方法来给设置单击事件，想要给RemoteViews设置点击事件，就必须使用PendingIntent，通过setOnClickPendingIntent方法来设置。PendingIntent是通过send和cancel方法来发送和取消待执行的Intent。

PendingIntent支持三种待定意图，启动activity（常见的通知）、启动Service和发送广播。它的主要方法有下面这些：

![PendingIntent的方法](http://upload-images.jianshu.io/upload_images/1159224-b4cf6d03cc97f0f3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

启动Activity它有两种，启动单个和启动多个，当使用getActivities时，实际上启动的是Intent数组中最后一个activity，如果要让最后一个activity返回时不退出app而是退回到上一个activity，实现方式可参照我上面第一个按钮的点击处理。

getActivity、getService、getBroadcast这三个方法的参数意义都是相同的，第一个上下文，第三个待定的意图，第二个requestCode表示PendingIntent发送方的请求码，多数情况下设置为0即可，另外requestCode会影响到第四个参数flags的效果。flags这个标志位表示执行效果。

常见的flags类型有FLAG_ONE_SHOT、FLAG_NO_CREATE、FLAG_CANCEL_CURRENT、FLAG_UPDATE_CURRENT。要理解这四个标志位的含义和区别，我们首先要弄明白PendingIntent的匹配规则，也就是什么情况下PendingIntent是相同的。

匹配规则：

- 如果两个PendingIntent它们内部的Intent相同，且requestCode也相同，那么这两个PendingIntent就是相同的；

Intent相同的情况：

- 如果两个Intent的ComponentName和intent-filter都相同，那么这两个Intent就是相同的。（Extras不参与Intent的匹配过程，就是它不同，只要ComponentName和intent-filter相同，Intent都算相同的。）

FLAG_ONE_SHOT：表示当前描述的PendingIntent只能被使用一次，然后它就会自动cancel，如果后续还有相同的PendingIntent，那么它们的send方法就会调用失败。如果通知栏消息使用这种标记位，同类型的通知就只会被打开一次，后续的通知将无法点开。

FLAG_NO_CREATE：表示当前描述的PendingIntent不会主动创建，如果当前PendingIntent之前不存在，那么getActivities等这些方法会直接返回null，获取PendingIntent失败。它无法单独使用。

FLAG_CANCEL_CURRENT：表示当前描述的PendingIntent如果已经存在，就cancel它，然后系统会创建一个新的。

FLAG_UPDATE_CURRENT：表示当前描述的PendingIntent如果已经存在，那么它会被更新，内部的Intent中的Extras也会被更新。


## RemoteViews的内部机制

RemoteViews的构造方法很多，我们最常见的一个是

```java
public RemoteViews(String packageName, int layoutId) {
    this(getApplicationInfo(packageName, UserHandle.myUserId()), layoutId);
}
```

只需要包名和待加载的资源文件id，它并不能支持所有类型的View，也不支持自定义的View，它能支持的类型如下：

Layout：FrameLyout、LinearLayout、RelativeLayout、GridLayout

View：Button、ImageView、ImageButton、ProgressBar、TextView、ListView、GridView、StackView、ViewStub、AdapterViewFlipper、ViewFlipper、AnalogClock、Chronometer。

如果我们在RemoteViews中使用了它不支持的View不如EditText，那么就会发生异常。

我们看看RemoteViews的set方法

![RemoteView-set方法](http://upload-images.jianshu.io/upload_images/1159224-afee50d12f729f34.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从这些方法中看出，原本可以直接调用的View的方法，现在要通过RemoteViews的一系列set方法来完成。

我们知道，通知栏和桌面小部件分别由NotificationManager和AppWidgetManager来管理的，而NotificationManager和AppWidgetManager是通过Binder分别和SystemServer进程中的NotificationManagerService以及AppWidgetService进行通信，因此，通知栏和桌面小部件中的布局文件实际上是在NotificationManagerService和AppWidgetService中被加载的，而他们运行在SystemServer中，这其实已经和我们自己的app进程构成了跨进程通信。

**理论分析**

首先RemoteViews会通过Binder传递到SystemServer进程，因为RemoteViews实现了Parcelable接口，可以跨进程传输，系统会根据RemoteViews中的包名等信息去获取到该app的资源，然后通过LayoutInflater去加载RemoteViews中的布局文件。在SystemServer进程中加载后的布局文件是一个普通的View，只不过对于我们的app进程来说，它是一个远程View也就是RemoteViews。接着系统会对View执行一系列界面更新任务，这些任务就是之前我们通过set方法提交的，set方法对View的更新操作并不是立刻执行的，在RemoteViews内部会记录所有的更新操作，具体的执行要等到RemoteViews被完全加载以后，这样RemoteViews就可以在SystemServer中进程中显示了，这就是我们所看到的通知栏消息和桌面小部件。当需要更新RemoteViews时，我们又需要调用一系列set方法通过NotificationManager和AppWidgetManager来提交更新任务，具体更新操作也是在SystemServer进程中完成的。


理论上讲系统完全可以通过Binder去支持所有的View和View操作，但是这样做代价太大，View的方法太多了，另外大量的IPC操作会影响效率。为了解决这个问题，系统并没有通过Binder去直接支持View的跨进程访问，而是提供了一个Action的概念，Action代表一个View操作，Action同样实现了Parcelable接口。系统首先将View操作封装到Action对象并将这些对象跨进程传输到远程进程，接着在远程进程中执行Action对象中的具体操作。在我们的app中每调用一次set方法，RemoteViews中就会添加一个对应的Action对象，当我们通过NotificationManager和AppWidgetManager来提交我们的更新时，这些Action对象就会传输到远程进程并在远程进程中依次执行。远程进程通过RemoteViews的apply方法来进行View的更新操作，apply方法内部是去遍历所有的Action对象并调用它们的apply方法，具体的View更新操作是由Action对象的apply方法来完成。

上述做法的好处，首先是不需要定义大量的Binder接口，其次通过在远程进程中批量执行RemoteViews的更新操作从而避免了大量的IPC操作，这就提高了程序的性能。

**源码分析**

首先我们从RemoteViews的set方法入手，比如设置图片的方法setImageViewResource它内部实现是这样的：

```java
/**
 * Equivalent to calling ImageView.setImageResource
 *
 * @param viewId The id of the view whose drawable should change
 * @param srcId The new resource id for the drawable
 */
public void setImageViewResource(int viewId, int srcId) {
    setInt(viewId, "setImageResource", srcId);
}
```

上面的代码中viewId是被操作的View的id，setImageResource是方法名，srcId是要给这个ImageView设置的图片资源id。这里的方法名和ImageView的setImageResource是一致的。我们再看看setInt方法的具体实现：

```java
/**
 * Call a method taking one int on a view in the layout for this RemoteViews.
 *
 * @param viewId The id of the view on which to call the method.
 * @param methodName The name of the method to call.
 * @param value The value to pass to the method.
 */
public void setInt(int viewId, String methodName, int value) {
    addAction(new ReflectionAction(viewId, methodName, ReflectionAction.INT, value));
}
```

可以看到它内部并没有对View进行直接操作，而是添加了一个ReflectionAction对象，字面上理解应该是一个反射类型的动作，再看addAction的实现：

```java
/**
 * Add an action to be executed on the remote side when apply is called.
 *
 * @param a The action to add
 */
private void addAction(Action a) {
    if (hasLandscapeAndPortraitLayouts()) {
        throw new RuntimeException("RemoteViews specifying separate landscape and portrait" +
                " layouts cannot be modified. Instead, fully configure the landscape and" +
                " portrait layouts individually before constructing the combined layout.");
    }
    if (mActions == null) {
        mActions = new ArrayList<Action>();
    }
    mActions.add(a);

    // update the memory usage stats
    a.updateMemoryUsageEstimate(mMemoryUsageCounter);
}
```

上述代码可以看到，在RemoteViews内部维护了一个名为mActions的ArrrayList，所有的对View更新的操作动作都被添加到这个集合中，注意，仅仅是添加进来保存，并没有去执行这些Action。到这里setImageViewResource方法的源码已经结束了，下面我们要弄清楚这些Action的执行。我们再看看RemoteViews的apply方法：

```java
public View apply(Context context, ViewGroup parent, OnClickHandler handler) {
    RemoteViews rvToApply = getRemoteViewsToApply(context);

    View result = inflateView(context, rvToApply, parent);
    loadTransitionOverride(context, handler);

    rvToApply.performApply(result, parent, handler);

    return result;
}
```

首先RemoteViews会通过LayoutInflater去加载它的布局文件，加载完之后通过performApply方法去执行一些更新操作。

```java
private void performApply(View v, ViewGroup parent, OnClickHandler handler) {
    if (mActions != null) {
        handler = handler == null ? DEFAULT_ON_CLICK_HANDLER : handler;
        final int count = mActions.size();
        for (int i = 0; i < count; i++) {
            Action a = mActions.get(i);
            a.apply(v, parent, handler);
        }
    }
}
```

这里遍历了mActions集合且执行每个Action的apply方法，应该可以看出，Action的apply方法才是真正操作View更新的地方。

当我们调用RemoteViews的set方法时，并不会立刻更新它们的界面，而必须要通过NotificationManager的notify方法以及AppWidgetManager的updateAppWidget方法才能更新它们的界面。实际上在AppWidgetManager的updateAppWidget内部实现中，的确是通过RemoteViews的apply方法和reapply方法来加载或更新界面的，apply和reapply的区别在于：apply会加载布局并更新界面，而reapply则只会更新界面，初始化时调用apply方法，后面的更新则调用reapply方法。

ReflectionAction是Action的子类，我们看下它的源码：

```java
/**
 * Base class for the reflection actions.
 */
private final class ReflectionAction extends Action {

    String methodName;
    int type;
    Object value;

    ReflectionAction(int viewId, String methodName, int type, Object value) {
        this.viewId = viewId;
        this.methodName = methodName;
        this.type = type;
        this.value = value;
    }

    @Override
    public void apply(View root, ViewGroup rootParent, OnClickHandler handler) {
        final View view = root.findViewById(viewId);
        if (view == null) return;

        Class<?> param = getParameterType();
        if (param == null) {
            throw new ActionException("bad type: " + this.type);
        }

        try {
            getMethod(view, this.methodName, param).invoke(view, wrapArg(this.value));
        } catch (ActionException e) {
            throw e;
        } catch (Exception ex) {
            throw new ActionException(ex);
        }
    }
}
```

它的内部实现有点长，我们主要看它的apply方法。

```java
getMethod(view, this.methodName, param).invoke(view, wrapArg(this.value));
```

这句代码就是它以反射的方式来对View进行操作，getMethod根据方法名得到反射所需的Method对象，然后执行该方法。

RemoteViews中的单击事件，只支持发起PendingIntent，不支持onClickListener这种方法。我们需要注意setOnClickPendingIntent、setPendingIntentTemplate和setOnClickFillInIntent这几个方法之间的区别和联系。setOnClickPendingIntent是用于给普通的View设置点击事件，但是它不能给ListView或者GridView、StackView中的item设置点击事件，因为开销比较大，系统禁止了这种方式。而setPendingIntentTemplate方法就能给item设置单击事件，具体使用请参照这篇文章[Android 之窗口小部件高级篇--App Widget 之 RemoteViews](http://www.cnblogs.com/skywang12345/p/3264991.html)。


## RemoteVIews的优缺点

实际开发中，跨进程通信我们可以选择AIDL去实现，但是如果对界面的更新比较频繁，这时会有效率问题，而且AIDL接口可能会变得很复杂，但如果采用RemoteViews来实现就没有这个问题了，RemoteViews的缺点就是它仅支持一些常见的View，而对于自定义View是不支持的。




