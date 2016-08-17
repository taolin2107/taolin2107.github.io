---
title: Android res资源文件如何混淆
date: 2016-08-15 20:47:56
tags: [Android, resource, proguard, 代码混淆]
categories: android
---

由于apk的反编译比较简单，发布没有混淆过的apk，基本就等于把源码公开了。
对于release版的app，对代码进行混淆是有必要的，市场上发布的apk大部分也都进行了混淆，不过一般都只是对java代码进行混淆。通过没有混淆的资源文件，代码也很容易被破解，为了提高apk的破解难度，res资源文件也应该进行混淆。

下面介绍如何用AndResGuard对资源文件进行混淆

### 1. 添加依赖包
在Project根目录下build.gradle中添加dependency：

```xml
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.1.0'
        //添加AndResGuard依赖
        classpath 'com.tencent.mm:AndResGuard-gradle-plugin:1.1.9'
    }
}
```
<!-- more -->

在app目录下的build.gradle中添加AndResGuard插件：

```xml
apply plugin: 'com.android.application'
apply plugin: 'AndResGuard'    //添加AndResGuard插件
...
```

### 2. 配置AndResGuard
在app目录下的build.gradle中添加：

```xml
andResGuard {
    //使用指定的映射文件进行混淆，详细解释见下面注释
    mappingFile = file("../mapping/resource_mapping_name.txt")
    if (!mappingFile.exists()) {
        mappingFile = null    //为null时表示不keepmapping，否则keepmapping
    }
    use7zip = true     //是否使用7z重新压缩签名后的apk包
    useSign = true     //是否使用签名
    keepRoot = false   //是否将res/drawable混淆成r/s
    // because the launcher will get the icon with his name
    whiteList = [      //混淆白名单，比如通过getIdentifier加载的资源是不能混淆的
            "R.drawable.ic_launcher",
            "R.string.app_name"
    ]
    compressFilePattern = [    //需要压缩的文件类型
            "*.png",
            "*.jpg",
            "*.jpeg",
            "*.gif",
            "resources.arsc"
    ]
    sevenzip {       //指定使用的7z命令
        artifact = 'com.tencent.mm:SevenZip:1.1.9'
        //path = "/usr/local/bin/7za"
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
...

注：keepmapping 可以保持所有版本混淆的一致性，比如资源ic_back混淆后映射为d。
如果不keepmapping，那么再次混淆可能映射为e，为了兼容之前的版本，会产生一些映射冗余。
通过AndResGuard 代码混淆完后会生成一个 resource_mapping开头的文件，将此文件保存下来。
比如我上面指定的文件位置"../mapping/resource_mapping_name.txt"，下次代码混淆就会以这个映射文件为基础。
```

### 3. 运行混淆命令
在Project根目录下执行命令：
```
./gradlew resguard

在AndResGuard开头的目录下会生成如下6个文件：

appname_unsigned.apk        //混淆完，未签名
appname_signed.apk          //签名后的apk
appname_signed_7zip.apk     //签名压缩后的apk
appname_signed_aligned.apk         //签名对齐后的apk
appname_signed_7zip_aligned.apk    //签名压缩对齐后的apk，可供发布
resource_mapping_name.txt      //生成的映射文件，请保留，下次混淆指定mappingFile为此文件
```

详情参考：https://github.com/shwenzhang/AndResGuard
