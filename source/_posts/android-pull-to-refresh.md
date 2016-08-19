---
title: Android 下拉刷新控件的使用
date: 2014-08-06 10:43:29
tags: [Android, pullToRefresh, 下拉刷新, Android控件]
categories: android
---

{% asset_img 01.png %}

如上图，是Android下拉刷新控件PullToRefresh，这种控件使用比较普遍，Google+、知乎等都有使用。
Github地址： https://github.com/chrisbanes/ActionBar-PullToRefresh.git
下面介绍此控件的用法：

## 1. PullToRefresh基本用法

```xml
//在build.gradle中添加依赖库
compile 'com.github.chrisbanes.actionbarpulltorefresh:library:+'

//在xml中引用如下：
<uk.co.senab.actionbarpulltorefresh.library.PullToRefreshLayout
    android:id="@+id/pull_refresh_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ListView
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</uk.co.senab.actionbarpulltorefresh.library.PullToRefreshLayout>
```

```java
//在Java代码中引用如下：
pullRefreshLayout = (PullToRefreshLayout) findViewById(R.id.pull_refresh_layout);
ActionBarPullToRefresh.from(this)
    .allChildrenArePullable()
    .listener(new OnRefreshListener() {
        @Override
        public void onRefreshStarted(View view) {
            //add your refreshing code.
        }
    })
    .setup(pullRefreshLayout);
```
<!-- more -->

通过以上代码，你就可以看到默认的下拉刷新效果了，如果需要自定义效果，请看下面。

## 2. 自定义PullToRefresh的效果

```java
...
ActionBarPullToRefresh.from(this)
        .options(Options.create()
        // 重点是这儿，pulltorefresh_header里面自定义效果
        .headerLayout(R.layout.pulltorefresh_header)
        .build())
        .allChildrenArePullable()
...
```

pulltorefresh_header.xml代码如下：

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android" 
    android:layout_width="match_parent" 
    android:layout_height="wrap_content">

    <FrameLayout
        android:id="@id/ptr_content" 
        android:layout_width="match_parent" 
        android:layout_height="48dp" 
        android:background="?android:attr/colorBackground">

        <TextView
            android:id="@id/ptr_text" 
            android:layout_width="match_parent" 
            android:layout_height="match_parent" 
            android:textAppearance="?android:attr/textAppearanceMedium" 
            android:gravity="center" />

    </FrameLayout>

    //上面的代码一般不作修改，主要的修改在下面，定义SmoothProgressBar的效果
    <fr.castorflex.android.smoothprogressbar.SmoothProgressBar
        xmlns:app="http://schemas.android.com/apk/res-auto" 
        android:id="@id/ptr_progress" 
        android:layout_width="match_parent" 
        android:layout_height="wrap_content" 
        app:spb_stroke_separator_length="0dp" 
        app:spb_colors="@array/refresh_pb_colors" 
        android:minHeight="3dp"/>

</RelativeLayout>
```

自定义SmoothProgressBar的效果
```xml
<fr.castorflex.android.smoothprogressbar.SmoothProgressBar
    xmlns:android="http://schemas.android.com/apk/res/android" 
    xmlns:app="http://schemas.android.com/apk/res-auto" 
    android:layout_width="match_parent" 
    android:layout_height="wrap_content" 
    app:spb_sections_count="4"      //颜色条的个数
    app:spb_color="#FF0000"         //进度条的颜色，单色
    app:spb_colors="@array/refresh_pb_colors" //进度条的颜色，多色
    app:spb_speed="2.0"             //滚动速度，此处为2倍的速度
    app:spb_reversed="false"        //是否反向滚动
    app:spb_mirror_mode="false"     //是否为镜像显示模式
    app:spb_stroke_width="4dp"      //颜色条的宽度
    app:spb_stroke_separator_length="4dp"      //颜色条之间的间距
    app:spb_progressiveStart_activated="true"  //默认的滚动的
    app:spb_progressiveStart_speed="1.5"       //开始点的速度
    app:spb_progressiveStop_speed="3.4"        //结束点的速度
    app:spb_interpolator="spb_interpolator_accelerate"   //插值器
    />
```
