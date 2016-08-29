---
title: Android中shape图形的用法详解
date: 2014-03-19 12:32:58
tags: [Android, shape, Android图形]
categories: android
---

Android可以通过代码生成一些简单的图片资源，节省空间，还方便后续修改。
下面介绍用shape生成图形的方法

## 1. shape图形的新建与引用
- 在res/drawable下新建一个图片资源，如ic_back.xml；
- 在代码中引用这个图片资源，如：`android:src="@drawable/ic_back"`。

## 2. shape图形的语法

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    //共有4种类型，矩形（默认）/椭圆形/直线形/环形
    android:shape=["rectangle" | "oval" | "line" | "ring"]
    // 以下4个属性只有当类型为环形时才有效
    android:innerRadius="dimension"   //内环半径
    android:innerRadiusRatio="float"  //内环半径相对于环的宽度的比例，比如环的宽度为50，比例为2.5，那么内环半径为20
    android:thickness="dimension"     //环的厚度
    android:thicknessRatio="float"    //环的厚度相对于环的宽度的比例
    android:useLevel="boolean">       //如果当做是LevelListDrawable使用时值为true，否则为false.

    //定义圆角
    <corners
        android:radius="dimension"          //全部的圆角半径
        android:topLeftRadius="dimension"   //左上角的圆角半径
        android:topRightRadius="dimension"  //右上角的圆角半径
        android:bottomLeftRadius="dimension"      //左下角的圆角半径
        android:bottomRightRadius="dimension" />  //右下角的圆角半径

    //定义渐变效果
    <gradient
        //共有3中渐变类型，线性渐变（默认）/放射渐变/扫描式渐变
        android:type=["linear" | "radial" | "sweep"]
        android:angle="integer"     //渐变角度，必须为45的倍数，0为从左到右，90为从上到下
        android:centerX="float"     //渐变中心X的相当位置，范围为0～1
        android:centerY="float"     //渐变中心Y的相当位置，范围为0～1
        android:startColor="color"      //渐变开始点的颜色
        android:centerColor="color"     //渐变中间点的颜色，在开始与结束点之间
        android:endColor="color"        //渐变结束点的颜色
        android:gradientRadius="float"  //渐变的半径，只有当渐变类型为radial时才能使用
        //使用LevelListDrawable时就要设置为true。设为false时才有渐变效果
        android:useLevel=["true" | "false"] />

    //内部边距
    <padding
        android:left="dimension"
        android:top="dimension"
        android:right="dimension"
        android:bottom="dimension" />

    //自定义的图形大小
    <size
        android:width="dimension"
        android:height="dimension" />

    //内部填充颜色
    <solid
        android:color="color" />

    //描边
    <stroke
        android:width="dimension"   //描边的宽度
        android:color="color"       //描边的颜色
        // 以下两个属性设置虚线
        android:dashWidth="dimension"      //虚线的宽度，值为0时是实线
        android:dashGap="dimension" />     //虚线的间隔
</shape>
```
<!-- more -->

## 3. shape图形示例

### 3.1. 圆角矩形，扫描式渐变

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle"
    android:useLevel="false" >

    <corners
        android:topLeftRadius="10dp"
        android:topRightRadius="10dp"
        android:bottomLeftRadius="10dp"
        android:bottomRightRadius="10dp" />

    <gradient
        android:type="sweep"
        android:endColor="@android:color/holo_blue_bright"
        android:startColor="@android:color/holo_green_dark"
        android:centerColor="@android:color/holo_blue_dark"
        android:useLevel="false" />

    <size android:width="60dp"
        android:height="60dp" />
</shape>

```
**结果如下：**
{% asset_img 01.png %}

### 3.2. 圆形，线性渐变

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval"
    android:useLevel="false" >

    <gradient
        android:type="linear"
        android:angle="45"
        android:startColor="@android:color/holo_green_dark"
        android:centerColor="@android:color/holo_blue_dark"
        android:endColor="@android:color/holo_red_dark"
        android:useLevel="false" />

    <size android:width="60dp"
        android:height="60dp" />

    <stroke android:width="1dp"
        android:color="@android:color/white" />

</shape>

```
**结果如下：**
{% asset_img 02.png %}

### 3.3. 虚线
```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="line" >

    <size android:width="60dp"
        android:height="60dp" />

    <stroke android:width="2dp"
        android:color="@android:color/holo_purple"
        android:dashWidth="10dp"
        android:dashGap="5dp" />
</shape>
```

**结果如下：**
{% asset_img 03.png %}

### 3.4. 环形，放射型渐变
```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="ring"
    android:useLevel="false"
    android:innerRadius="40dp"
    android:thickness="3dp">

    <gradient android:type="radial"
        android:gradientRadius="150"
        android:centerY="0.1"
        android:centerX="0.2"
        android:startColor="@android:color/holo_red_dark"
        android:centerColor="@android:color/holo_green_dark"
        android:endColor="@android:color/white" />

    <size android:width="90dp"
        android:height="90dp" />

</shape>
```

**结果如下：**
{% asset_img 04.png %}

官方文档：https://developer.android.com/guide/topics/resources/drawable-resource.html#Shape
