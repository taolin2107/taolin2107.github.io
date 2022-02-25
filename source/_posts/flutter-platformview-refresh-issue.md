---
title: Flutter PlatformView避坑指南 —— UI刷新
date: 2022-02-25 19:05:12
tags: [Flutter]
categories: flutter
---


本文记录在flutter中嵌入原生系统时view遇到的刷新问题和尝试过的解决方案。

### 1. 什么是Platform view ？

> 由于 Flutter 诞生于 Android 、iOS 非常成熟的时代背景，为了能让一些现有的 native 控件直接引用到 Flutter app 中，Flutter 团队提供了 AndroidView 、UIKitView 两个 widget 来满足需求，比如说 Flutter 中的 Webview、MapView，这样无需使用 Flutter 重新开发一套。 

其实 Platform view 就是 AndroidView 和 UIKitView 的总称，允许将 native view 嵌入到了 flutter widget 体系中，完成 Dart 代码对 native view 的控制。

### 2. 使用Platform view遇到的问题

项目中需要实现视频通话的功能，效果见下图。由于使用的视频通话sdk没有flutter版本的，且sdk只提供渲染后的视图view，不提供视频数据。所以只能考虑在flutter中嵌入原生系统Platform view。

<img src="/2022/02/25/flutter-platformview-refresh-issue/01.jpg" width="300" height="627">

实现步骤以Android平台为例，我们需要先对视频通话sdk进行一次封装以便flutter层能调用。
首先在原生系统层将视频通话视图列表缓存到一个列表中，列表的每个item包含一个通话视图、视图id及其他信息，通话视图通过Platformview的方式暴露给flutter层。
然后把每个视图的id列表，通过MethodChannel的方式传递给flutter层。flutter层获取到视图的id列表，通过给Platformview传递视图id来获取某个item的通话视图，再将该视图显示到具体的位置。

实际测试发现，视频通话列表可以正常显示，但在列表数量变化时（比如有人加入了视频通话），视图列表出现错乱。

<img src="/2022/02/25/flutter-platformview-refresh-issue/02.jpg" width="300" height="627">

异常log如下：

{% asset_img 03.png %}

从log可以看到，报错是因为flutter加载原生view时，该原生view还被另一个父view引用；Android中规定一个view只能有一个父view，所以这里报错。

<!-- more -->

### 3. 尝试过的解决方案

#### 3.1 解决方案一
分析：由于flutter层显示视频通话列表是一个listview，listview组件为了提高性能会有复用机制，考虑可能是这个机制导致加载原生view时错乱。

所以首先尝试给listview的每个item加一个UniqueKey，确保每个item是唯一的，这样由于每个item的key不同，来避免复用的问题，实际测试发现不起作用。

进一步方案，给每个item尺寸额外增加一个padding，padding值是一个0到1的随机数，当列表数量变化时，listview视图重新加载，每个item的尺寸有变化，视图大小来确保item不会复用，实际测试发现不起作用。

#### 3.2 解决方案二

分析：报错是因为flutter加载原生view时，该原生view还被另一个父view引用。所以尝试在获取原生view时，先判断该view是否有父view，如果有就先移除该view，(view.getParent()).removeView(view)，确保该view没有被其他父view引用。
实际测试发现，flutter在加载原生view时，会2次获取原生view，第2次获取时，原生view有父view，此时移除有父view后，flutter会显示异常。

修改方案，当第一个加载原生view时，移除其父view，第2次加载不移除。
测试发现，flutter会显示异常，而且flutter层会不停的获取原生view，进入死循环。
此方案破坏了flutter视图绘制的流程，不可行。

#### 3.3 解决方案三

从flutter widget生命周期着手分析，根据官方文档提供的widget生命周期如下，widget在deactivate时从渲染树中移除，但不会马上销毁，framework可能会根据情况重新复用这个widget；widget在dispose后，就会被彻底移除然后销毁。

<img src="/2022/02/25/flutter-platformview-refresh-issue/04.png" width="400" height="581">

<img src="/2022/02/25/flutter-platformview-refresh-issue/05.png" width="500" height="260">

<img src="/2022/02/25/flutter-platformview-refresh-issue/06.png" width="500" height="260">

写一个demo验证widget刷新时，生命周期的变化。

<img src="/2022/02/25/flutter-platformview-refresh-issue/07.png" width="381" height="210">

<img src="/2022/02/25/flutter-platformview-refresh-issue/08.png" width="400" height="100">

根据log可以看出Flutter widget刷新时，后一个widget和前一个widget的生命周期存在部分重叠。
由此可知，如果刷新前后两个widget都引用了同一个原生view时，就会出现上面的异常。

{% asset_img 09.png %}


### 4. 最终结论

在Flutter中如果嵌入原生系统Platform view，要确保widget刷新前后，引用的原生view能复用，否则会出现原生view被2个parent引用而出现的异常。