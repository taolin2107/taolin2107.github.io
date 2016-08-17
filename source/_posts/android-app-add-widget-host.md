---
title: Android应用添加插件容器widget host功能
date: 2016-08-16 16:40:02
tags: [Android, widget-host, 插件容器, widget]
categories: android
---

项目中遇到这样的问题，我们有一款大屏Android设备，为了充分利用大屏的优势，我们开发了一套操作界面(Launcher)，所有的功能都在这个Launcher界面的一个个独立的panel中，要使用哪个功能，直接操作对应的panel就可以了，panel可以随意拖动。（如下图所示）
{% asset_img img1.png module panels %}

随着功能模块越来越多，代码结构变得复杂，不好维护，后面我们就规划把其中一些模块用widget的方式实现，这样差不多有一半的模块可以分离出来，但整体效果不变。
这些功能模块要想通过widget的方式添加进来，Launcher就得实现widget容器的功能。

下面介绍如何让Android app支持widget容器功能

## 1. 初始化Widget host

```java
//系统 Launcher的host id 一般为1024，这里选个跟它不冲突的id就可以
private static final int APPWIDGET_HOST_ID = 2039;
private AppWidgetManager mAppWidgetManager;
private AppWidgetHost mAppWidgetHost;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    initWidgetHost();
}

private void initWidgetHost() {
    mAppWidgetManager = AppWidgetManager.getInstance(this);
    mAppWidgetHost = new AppWidgetHost(this, APPWIDGET_HOST_ID);
    mAppWidgetHost.startListening();   //监听widget变化事件，没有这句widget不能刷新
}

@Override
public void onDestroy() {
    super.onDestroy();
    try {
        mAppWidgetHost.stopListening();   //退出时停止监听
    } catch (NullPointerException ex) {
        ex.printStackTrace();
    }
}
```

## 2. 显示默认添加的widget
有一些widget我们希望一打开Launcher就显示，不用用户手动添加，方法如下：

### 2.1. 在AndroidManifest中添加权限
```xml
<uses-permission android:name="android.permission.BIND_APPWIDGET" />
```

### 2.2. widget容器的布局
```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/widget_container"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>
```

### 2.3. 添加默认显示的widget
```java
private RelativeLayout mWidgetContainer;
// 添加的widget个数，方便给widget设置id
private int mWidgetCount;

@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    //初始化widget容器
    mWidgetContainer = (RelativeLayout) findViewById(R.id.widget_container);
    addDefaultWidget();    //添加默认显示的widget
    ...
}

private void addDefaultWidget() {
    //为即将添加的widget分配一个widget id
    int widgetId = mAppWidgetHost.allocateAppWidgetId();

    //用系统设置的widget作为示例
    ComponentName componentName = new ComponentName("com.android.settings", "com.android.settings.widget.SettingsAppWidgetProvider");
    mAppWidgetManager.bindAppWidgetIdIfAllowed(widgetId, componentName);
    AppWidgetProviderInfo widgetInfo = mAppWidgetManager.getAppWidgetInfo(widgetId);
    //创建widget view
    AppWidgetHostView widgetView = mAppWidgetHost.createView(mContext, widgetId, widgetInfo);

    //这儿计算widget的宽高，注意要加上widget的padding
    int widgetWidth = widgetInfo.minWidth + widgetView.getPaddingLeft() + widgetView.getPaddingRight();
    int widgetHeight = widgetInfo.minHeight + widgetView.getPaddingTop() + widgetView.getPaddingBottom();
    RelativeLayout.LayoutParams layoutParams = new RelativeLayout.LayoutParams(widgetWidth, widgetHeight);
    layoutParams.leftMargin = 100;    //设置widget view的margin
    layoutParams.topMargin = 100;
    widgetView.setId(mWidgetCount + 100);    //给widget view设置一个id，不设置的话widget不能更新
    mWidgetContainer.addView(widgetView, layoutParams);
    mWidgetCount++;
}

```

### 2.4. 编译、签名、安装
默认显示widget需要的权限比较高，要求用系统签名和放到system/app目录下面
1. 编译生成apk文件，如widget-host-demo.apk。
2. 用系统的platformkey对apk签名，命令：java -jar signapk.jar platform.x509.pem platform.pk8 widget-host-demo.apk widget-host-demo_signed.apk
3. push到system/app下面，adb push widget-host-demo_signed.apk /system/app。（需要adb有root权限）
4. 重启设备，打开app就能看到默认显示的widget。

## 3. 实现动态添加widget功能
动态添加widget需要的权限就比较低，不需要系统签名和放到system/app目录下面。

### 3.1. 发送widget pick的Intent
```java
private static final int REQUEST_PICK_APPWIDGET = 1;

//打开选择widget的弹出框
private void doWidgetPick() {
    //为widget分配一个widget id
    int appWidgetId = mAppWidgetHost.allocateAppWidgetId();
    Intent pickIntent = new Intent(AppWidgetManager.ACTION_APPWIDGET_PICK);
    pickIntent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, appWidgetId);
    startActivityForResult(pickIntent, REQUEST_PICK_APPWIDGET);
}
```

### 3.2. 处理widget pick返回的结果

重写Activity的onActivityResult方法

```java
private static final int REQUEST_CREATE_APPWIDGET = 2;

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    if (resultCode == RESULT_OK) {
        switch (requestCode) {
            case REQUEST_PICK_APPWIDGET:
                addAppWidget(data);
                break;

            case REQUEST_CREATE_APPWIDGET:
                completeAddAppWidget(data);
                break;
        }
    } else if ((requestCode == REQUEST_PICK_APPWIDGET || requestCode == REQUEST_CREATE_APPWIDGET)
            && resultCode == RESULT_CANCELED && data != null) {
        // Clean up the appWidgetId if we canceled
        int appWidgetId = data.getIntExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, AppWidgetManager.INVALID_APPWIDGET_ID);
        if (appWidgetId != AppWidgetManager.INVALID_APPWIDGET_ID) {
            mAppWidgetHost.deleteAppWidgetId(appWidgetId);
        }
    }
}

//判断添加widget前是否需要对widget进行配置
private void addAppWidget(Intent data) {
    int appWidgetId = data.getIntExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, -1);
    AppWidgetProviderInfo appWidget = mAppWidgetManager.getAppWidgetInfo(appWidgetId);
    if (appWidget.configure != null) {
        // Launch over to configure widget, if needed
        Intent intent = new Intent(AppWidgetManager.ACTION_APPWIDGET_CONFIGURE);
        intent.setComponent(appWidget.configure);
        intent.putExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, appWidgetId);
        // 需要配置就打开widget配置的activity
        startActivityForResult(intent, REQUEST_CREATE_APPWIDGET);
    } else {
        // 不需要配置就直接添加
        onActivityResult(REQUEST_CREATE_APPWIDGET, Activity.RESULT_OK, data);
    }
}

//将widget添加到界面中显示出来，除了widget id外，其他和默认显示的widget一样
private void completeAddAppWidget(Intent data) {
    Bundle extras = data.getExtras();
    int appWidgetId = extras.getInt(AppWidgetManager.EXTRA_APPWIDGET_ID, -1);
    AppWidgetProviderInfo widgetInfo = mAppWidgetManager.getAppWidgetInfo(appWidgetId);
    AppWidgetHostView widgetView = mAppWidgetHost.createView(this, appWidgetId, widgetInfo);

    //计算widget的宽高
    int widgetWidth = widgetInfo.minWidth + widgetView.getPaddingLeft() + widgetView.getPaddingRight();
    int widgetHeight = widgetInfo.minHeight + widgetView.getPaddingTop() + widgetView.getPaddingBottom();

    RelativeLayout.LayoutParams layoutParams = new RelativeLayout.LayoutParams(widgetWidth, widgetHeight);
    layoutParams.leftMargin = 300;
    layoutParams.topMargin = 300;
    widgetView.setId(mWidgetCount + 100);
    mViewContainer.addView(widgetView, layoutParams);
    mWidgetCount++;
}


```

## 4. 总结
上面介绍了在Android app中显示widget的两种方法，要做好widget容器的功能，还应该处理好widget的显示位置、尺寸大小、尺寸缩放等问题，需另外讨论，可以参考Android系统的Launcher处理方式。
