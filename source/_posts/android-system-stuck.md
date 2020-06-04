---
title: Android系统卡顿如何优化
date: 2020-06-04 19:51:04
tags: [Android, System]
categories: android
---

## 1. 问题背景

在某次对Android系统测试中，出现系统响应比较慢，操作卡顿的情况。

操作卡顿，第一反应当然是看看CPU和内存的消耗是否正常，根据命令打印显示，CPU偶尔占用比较高，内存消耗也不算太高，分析了本地的log记录，也没有看出什么问题。

只能试着重现问题，看出现问题的概率高不高，如果不高的话，可能考虑此问题先放一放。

## 2. 问题复现 — monkey压测

压测使用的工具是monkey，它可以对整个系统或指定的单个应用进行随机的压力测试。

如下的脚本是分别对白板和设置进行monkey压测

- -p 指定测试的App包名
- -- ignore开头的参数，是测试过程中忽略掉对应的事件
- -- throttle 是执行每次测试的间隔时间，太快的话，可能会导致系统响应不过来，容易引起anr
- -v -v -v 三个v，表示记录最详细的日志
- 最后面的数字代表要测试的次数
- nohup 最后加一个&，是指脚本在后台运行

```powershell
# 比如对设置压测
nohup monkey -p com.android.settings --ignore-crashes --ignore-timeouts --ignore-native-crashes --ignore-security-exceptions --throttle 100 -v -v -v 1000000 &
```

通过使用monkey压测，确实能复现系统卡顿的问题，但接下来不知道怎么分析。遂去咨询系统开发的同事，他们分析了一些系统参数后，结合以往的经验，提出是因为系统产生了大量的内存碎片，这些内存碎片虽然是可用内存，但是内存空间不连续，无法被系统直接使用，导致内存不足，系统卡顿。

## 3. 内存分布 — buddyinfo

系统开发的同事提出是因为系统产生了大量的内存碎片导致卡顿，但需要进一步验证才能确定。
通过`cat /proc/buddyinfo` 可以查看系统当前的内存分布情况

<!-- more -->

```powershell
# buddyinfo 描述了当前可用内存的分布情况

cat /proc/buddyinfo 
Node 0, zone      DMA      4      4      3      3      3      3      2      0      1      1      2 
```

DMA 行第3列表示有4个 `2^2 × page_size` 的内存块可以用。
以此类推，越是往后的空间，就越是连续，数目越多，就代表这个大小的连续空间越多，当大的连续空间很少的时候，也就说明，内存碎片已经非常多了，到一定程度就可能引起系统卡顿。

为了证实确实是内存碎片引起的，就需要验证在系统从流畅到卡顿的过程中，记录系统内存分布情况，看它的变化趋势，是不是系统越卡顿，内存碎片越多，大块内存越少，也就是buddyinfo的前面的数据越来越大，后面的数据越来越小。

配合上面的压测代码，同时结合下面的脚本来记录buddyinfo内存分布变化

```powershell
#!/bin/bash

LOG_FILE="/sdcard/buddyinfo.log"

while true
do
    # 记录时间
    date >> $LOG_FILE
    # 记录buddyinfo
    cat /proc/buddyinfo >> $LOG_FILE
    # 换行
    echo >> $LOG_FILE
    # 间隔10分钟
    sleep 600
done
```

经过压测和buddyinfo的记录发现，确实是系统越卡顿，内存碎片越多，也就验证了问题是内存碎片引起的。

既然是内存碎片引起的卡顿，第一反应是要解决内存碎片的产生，看看是哪些应用产生了大量内存碎片。但是要排查内存碎片的产生，又是个非常大的工程，系统的每一行代码都有可能产生内存碎片，没法直接通过修改代码来解决内存碎片的问题。

## 4. 内存整理和优化

既然没法从局部解决问题，就只能从整体来考虑来优化，看有没有什么方法能对整个系统进行内存整理和优化。

经过研究发现，Android系统可以向`drop_caches `和`compact_memory `文件写数据，实现对内存的整理和优化。

- drop_caches 释放缓存
- compact_memory 合并低阶内存页来创造出足够的高阶内存页

具体使用如下

```powershell
# 同时释放 页、目录、索引节点缓存
echo 3 > /proc/sys/vm/drop_caches
# 释放目录和索引节点缓存
echo 2 > /proc/sys/vm/drop_caches
# 释放页缓存
echo 1 > /proc/sys/vm/drop_caches
# 内存压缩
echo 1 > /proc/sys/vm/compact_memory
```

## 5. 最终方案

最终的方案是把上面的内存整理和优化的代码加到Launcher中，每次回到Launcher时，进行一次内存优化。经过多次压测，对比使用了此方案之前的效果，整个系统流畅了很多，也很少出现卡顿的情况。

```kotlin
override fun onStart() {
    super.onStart()
    // 增加了如下代码
    var result = ShellCommandUtils.executeCommand("cat proc/buddyinfo")
    Logger.d("cat proc/buddyinfo, result=" + result.result + ",error=" + result.errorMsg + ",success=" + result.successMsg)
    result = ShellCommandUtils.executeCommand("echo 3 > /proc/sys/vm/drop_caches")
    Logger.d("echo 3 > /proc/sys/vm/drop_caches, result=" + result.result + ",error=" + result.errorMsg + ",success=" + result.successMsg)
    result = ShellCommandUtils.executeCommand("echo 2 > /proc/sys/vm/drop_caches")
    Logger.d("echo 2 > /proc/sys/vm/drop_caches, result=" + result.result + ",error=" + result.errorMsg + ",success=" + result.successMsg)
    result = ShellCommandUtils.executeCommand("echo 1 > /proc/sys/vm/drop_caches")
    Logger.d("echo 1 > /proc/sys/vm/drop_caches, result=" + result.result + ",error=" + result.errorMsg + ",success=" + result.successMsg)
    result = ShellCommandUtils.executeCommand("echo 1 > /proc/sys/vm/compact_memory")
    Logger.d("echo 1 > /proc/sys/vm/compact_memory, result=" + result.result + ",error=" + result.errorMsg + ",success=" + result.successMsg)
}
```
