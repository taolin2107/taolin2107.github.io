---
title: Android java代码如何混淆
date: 2016-07-18 20:53:18
tags: [Android, java, proguard, 代码混淆]
ategories: android
---

## 1. 前期准备
本篇文章中介绍的混淆技术都是基于Android Studio的，Eclipse的用法也基本类似，但是就不再为Eclipse专门做讲解了。 
我们要建立一个Android Studio项目，并在项目中添加一些能够帮助我们理解混淆知识的代码。这里我准备好了一些，我们将它们添加到Android Studio当中。 
首先新建一个MyFragment类，代码如下所示：
```java
public class MyFragment extends Fragment {

    private String toastTip = "toast in MyFragment";

    @Override
    public View onCreateView(LayoutInflater inflater, Nullable ViewGroup container, Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_layout, container, false);
        methodWithGlobalVariable();
        methodWithLocalVariable();
        return view;
    }

    public void methodWithGlobalVariable() {
        Toast.makeText(getActivity(), toastTip, Toast.LENGTH_SHORT).show();
    }

    public void methodWithLocalVariable() {
        String logMessage = "log in MyFragment";
        logMessage = logMessage.toLowerCase();
        System.out.println(logMessage);
    }

}
```

可以看到，MyFragment是继承自Fragment的，并且MyFragment中有一个全局变量。onCreateView()方法是Fragment的生命周期函数，这个不用多说，在onCreateView()方法中又调用了methodWithGlobalVariable()和methodWithLocalVariable()方法，这两个方法的内部分别引用了一个全局变量和一个局部变量。 
<!-- more -->

接下来新建一个Utils类，代码如下所示：
```java
public class Utils {

    public void methodNormal() {
        String logMessage = "this is normal method";
        logMessage = logMessage.toLowerCase();
        System.out.println(logMessage);
    }

    public void methodUnused() {
        String logMessage = "this is unused method";
        logMessage = logMessage.toLowerCase();
        System.out.println(logMessage);
    }

}
```

这是一个非常普通的工具类，没有任何继承关系。Utils中有两个方法methodNormal()和methodUnused()，它们的内部逻辑都是一样的，唯一的据别是稍后methodNormal()方法会被调用，而methodUnused()方法不会被调用。 
下面再新建一个NativeUtils类，代码如下所示：
```java
public class NativeUtils {

    public static native void methodNative();

    public static void methodNotNative() {
        String logMessage = "this is not native method";
        logMessage = logMessage.toLowerCase();
        System.out.println(logMessage);
    }

}
```

这个类中同样有两个方法，一个是native方法，一个是非native方法。 
最后，修改MainActivity中的代码，如下所示：
```java
public class MainActivity extends AppCompatActivity {

    private String toastTip = "toast in MainActivity";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        getSupportFragmentManager().beginTransaction().add(R.id.fragment, new MyFragment()).commit();
        Button button = (Button) findViewById(R.id.button);
        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                methodWithGlobalVariable();
                methodWithLocalVariable();
                Utils utils = new Utils();
                utils.methodNormal();
                NativeUtils.methodNative();
                NativeUtils.methodNotNative();
                Connector.getDatabase();
            }
        });
    }

    public void methodWithGlobalVariable() {
        Toast.makeText(MainActivity.this, toastTip, Toast.LENGTH_SHORT).show();
    }

    public void methodWithLocalVariable() {
        String logMessage = "log in MainActivity";
        logMessage = logMessage.toLowerCase();
        System.out.println(logMessage);
    }

}
```

可以看到，MainActivity和MyFragment类似，也是定义了methodWithGlobalVariable()和methodWithLocalVariable()这两个方法，然后MainActivity对MyFragment进行了添加，并在Button的点击事件里面调用了自身的、Utils的、以及NativeUtils中的方法。注意调用native方法需要有相应的so库实现，不然的话就会报UnsatisefiedLinkError，不过这里其实我也并没有真正的so库实现，只是演示一下让大家看看混淆结果。点击事件的最后一行调用的是LitePal中的方法，因为我们还要测试一下引用第三方Jar包的场景，到LitePal项目的主页去下载最新的Jar包，然后放到libs目录下即可。 
完整的build.gradle内容如下所示：
```xml
apply plugin: 'com.android.application'

android {
    compileSdkVersion 23
    buildToolsVersion "23.0.2"

    defaultConfig {
        applicationId "com.example.guolin.androidtest"
        minSdkVersion 15
        targetSdkVersion 23
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:23.2.0'
}
```

好的，到这里准备工作就已经基本完成了，接下来我们就开始对代码进行混淆吧。

## 2. 混淆APK

在Android Studio当中混淆APK实在是太简单了，借助SDK中自带的Proguard工具，只需要修改build.gradle中的一行配置即可。可以看到，现在build.gradle中minifyEnabled的值是false，这里我们只需要把值改成true，打出来的APK包就会是混淆过的了。如下所示：
```xml
release {
    minifyEnabled true
    proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
}
```

其中minifyEnabled用于设置是否启用混淆，proguardFiles用于选定混淆配置文件。注意这里是在release闭包内进行配置的，因此只有打出正式版的APK才会进行混淆，Debug版的APK是不会混淆的。当然这也是非常合理的，因为Debug版的APK文件我们只会用来内部测试，不用担心被人破解。 
那么现在我们来打一个正式版的APK文件，在Android Studio导航栏中点击Build->Generate Signed APK，然后选择签名文件并输入密码，如果没有签名文件就创建一个，最终点击Finish完成打包，生成的APK文件会自动存放在app目录下。除此之外也可以在build.gradle文件当中添加签名文件配置，然后通过gradlew assembleRelease来打出一个正式版的APK文件，这种方式APK文件会自动存放在app/build/outputs/apk目录下。 
那么现在已经得到了APK文件，接下来就用上篇文章中学到的反编译知识来对这个文件进行反编译吧，结果如下图所示： 
{% asset_img 01.png %}

很明显可以看出，我们的代码混淆功能已经生效了。 
下面我们尝试来阅读一下这个混淆过后的代码，最顶层的包名结构主要分为三部分，第一个a.a已经被混淆的面目全非了，但是可以猜测出这个包下是LitePal的所有代码。第二个android.support可以猜测出是我们引用的android support库的代码，第三个com.example.guolin.androidtest则很明显就是我们项目的主包名了，下面将里面所有的类一个个打开看一下。 
首先MainActivity中的代码如下所示： 
{% asset_img 02.png %}

可以看到，MainActivity的类名是没有混淆的，onCreate()方法也没有被混淆，但是我们定义的方法、全局变量、局部变量都被混淆了。 
再来打开下一个类NativeUtils，如下所示： 
{% asset_img 03.png %}

NativeUtils的类名没有被混淆，其中声明成native的方法也没有被混淆，但是非native方法的方法名和局部变量都被混淆了。 
接下来是a类的代码，如下所示： 
{% asset_img 04.png %}

很明显，这个是MainActivity中按钮点击事件的匿名类，在onClick()方法中的调用代码虽然都被混淆了，但是调用顺序是不会改变的，对照源代码就可以看出哪一行是调用的什么方法了。 
再接下来是b类，代码如下所示： 
{% asset_img 05.png %}

虽然被混淆的很严重，但是我们还是可以看出这个是MyFragment类。其中所有的方法名、全局变量、局部变量都被混淆了。 
最后再来看下c类，代码如下所示： 
{% asset_img 06.png %}

c类中只有一个a方法，从字符串的内容我们可以看出，这个是Utils类中的methodNormal()方法。 
我为什么要创建这样的一个项目呢？因为从这几个类当中很能看出一些问题，接下来我们就分析一下上面的混淆结果。 
首先像Utils这样的普通类肯定是会被混淆的，不管是类名、方法名还是变量都不会放过。除了混淆之外Utils类还说明了一个问题，就是minifyEnabled会对资源进行压缩，因为Utils类中我们明明定义了两个方法，但是反编译之后就只剩一个方法了，因为另外一个方法没有被调用，所以认为是多余的代码，在打包的时候就给移除掉了。不仅仅是代码，没有被调用的资源同样也会被移除掉，因此minifyEnabled除了混淆代码之外，还可以起到压缩APK包的作用。 
接着看一下MyFragment，这个类也是混淆的比较彻底的，基本没有任何保留。那有些朋友可能会有疑问，Fragment怎么说也算是系统组件吧，就算普通方法名被混淆了，至少像onCreateView()这样的生命周期方法不应该被混淆吧？其实生命周期方法会不会被混淆和我们使用Fragment的方式有关，比如在本项目中，我使用的是android.support.v4.app.Fragment，support-v4包下的，就连Fragment的源码都被一起混淆了，因此生命周期方法当然也不例外了。但如果你使用的是android.app.Fragment，这就是调用手机系统中预编译好的代码了，很明显我们的混淆无法影响到系统内置的代码，因此这种情况下onCreateView()方法名就不会被混淆，但其它的方法以及变量仍然会被混淆。 
接下来看一下MainActivity，同样也是系统组件之一，但MainActivity的保留程度就比MyFragment好多了，至少像类名、生命周期方法名都没有被混淆，这是为什么呢？根据我亲身测试得出结论，凡是需要在AndroidManifest.xml中去注册的所有类的类名以及从父类重写的方法名都自动不会被混淆。因此，除了Activity之外，这份规则同样也适用于Service、BroadcastReceiver和ContentProvider。 
最后看一下NativeUtils类，这个类的类名也没有被混淆，这是由于它有一个声明成native的方法。只要一个类中有存在native方法，它的类名就不会被混淆，native方法的方法名也不会被混淆，因为C++代码要通过包名+类名+方法名来进行交互。 但是类中的别的代码还是会被混淆的。 
除此之外，第三方的Jar包都是会被混淆的，LitePal不管是包名还是类名还是方法名都被完完全全混淆掉了。 
这些就是Android Studio打正式APK时默认的混淆规则。 
那么这些混淆规则是在哪里定义的呢？其实就是刚才在build.gradle的release闭包下配置的proguard-android.txt文件，这个文件存放于<Android SDK>/tools/proguard目录下，我们打开来看一下：
```java
# This is a configuration file for ProGuard.
# http://proguard.sourceforge.net/index.html#manual/usage.html

-dontusemixedcaseclassnames
-dontskipnonpubliclibraryclasses
-verbose

# Optimization is turned off by default. Dex does not like code run
# through the ProGuard optimize and preverify steps (and performs some
# of these optimizations on its own).
-dontoptimize
-dontpreverify
# Note that if you want to enable optimization, you cannot just
# include optimization flags in your own project configuration file;
# instead you will need to point to the
# "proguard-android-optimize.txt" file instead of this one from your
# project.properties file.

-keepattributes *Annotation*
-keep public class com.google.vending.licensing.ILicensingService
-keep public class com.android.vending.licensing.ILicensingService

# For native methods, see http://proguard.sourceforge.net/manual/examples.html#native
-keepclasseswithmembernames class * {
    native <methods>;
}

# keep setters in Views so that animations can still work.
# see http://proguard.sourceforge.net/manual/examples.html#beans
-keepclassmembers public class * extends android.view.View {
   void set*(***);
   *** get*();
}

# We want to keep methods in Activity that could be used in the XML attribute onClick
-keepclassmembers class * extends android.app.Activity {
   public void *(android.view.View);
}

# For enumeration classes, see http://proguard.sourceforge.net/manual/examples.html#enumerations
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

-keepclassmembers class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator CREATOR;
}

-keepclassmembers class **.R$* {
    public static <fields>;
}

# The support library contains references to newer platform versions.
# Dont warn about those in case this app is linking against an older
# platform version.  We know about them, and they are safe.
-dontwarn android.support.**
```

这个就是默认的混淆配置文件了，我们来一起逐行阅读一下。 
`-dontusemixedcaseclassnames` 表示混淆时不使用大小写混合类名。 
`-dontskipnonpubliclibraryclasses` 表示不跳过library中的非public的类。 
`-verbose` 表示打印混淆的详细信息。 
`-dontoptimize` 表示不进行优化，建议使用此选项，因为根据proguard-android-optimize.txt中的描述，优化可能会造成一些潜在风险，不能保证在所有版本的Dalvik上都正常运行。 
`-dontpreverify` 表示不进行预校验。这个预校验是作用在Java平台上的，Android平台上不需要这项功能，去掉之后还可以加快混淆速度。 
```java
-keepattributes *Annotation* 表示对注解中的参数进行保留。

-keep public class com.google.vending.licensing.ILicensingService
-keep public class com.android.vending.licensing.ILicensingService
```

表示不混淆上述声明的两个类，这两个类我们基本也用不上，是接入Google原生的一些服务时使用的。
```java
-keepclasseswithmembernames class * {
    native <methods>;
}
```

表示不混淆任何包含native方法的类的类名以及native方法名，这个和我们刚才验证的结果是一致的。
```java
-keepclassmembers public class * extends android.view.View {
   void set*(***);
   *** get*();
}
```

表示不混淆任何一个View中的setXxx()和getXxx()方法，因为属性动画需要有相应的setter和getter的方法实现，混淆了就无法工作了。
```java
-keepclassmembers class * extends android.app.Activity {
   public void *(android.view.View);
}
```

表示不混淆Activity中参数是View的方法，因为有这样一种用法，在XML中配置android:onClick=”buttonClick”属性，当用户点击该按钮时就会调用Activity中的buttonClick(View view)方法，如果这个方法被混淆的话就找不到了。
```java
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}
```

表示不混淆枚举中的values()和valueOf()方法，枚举我用的非常少，这个就不评论了。
```java
-keepclassmembers class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator CREATOR;
}
```

表示不混淆Parcelable实现类中的CREATOR字段，毫无疑问，CREATOR字段是绝对不能改变的，包括大小写都不能变，不然整个Parcelable工作机制都会失败。
```java
-keepclassmembers class **.R$* {
    public static <fields>;
}
```

表示不混淆R文件中的所有静态字段，我们都知道R文件是通过字段来记录每个资源的id的，字段名要是被混淆了，id也就找不着了。 
`-dontwarn android.support.**` 表示对android.support包下的代码不警告，因为support包中有很多代码都是在高版本中使用的，如果我们的项目指定的版本比较低在打包时就会给予警告。不过support包中所有的代码都在版本兼容性上做足了判断，因此不用担心代码会出问题，所以直接忽略警告就可以了。 
好了，这就是proguard-android.txt文件中所有默认的配置，而我们混淆代码也是按照这些配置的规则来进行混淆的。经过我上面的讲解之后，相信大家对这些配置的内容基本都能理解了。不过proguard语法中还真有几处非常难理解的地方，我自己也是研究了好久才搞明白，下面和大家分享一下这些难懂的语法部分。 
proguard中一共有三组六个keep关键字，很多人搞不清楚它们的区别，这里我们通过一个表格来直观地看下：

| 关键字 |       描述       |
| :----- | :--------------- |
| keep |	保留类和类中的成员，防止它们被混淆或移除。|
| keepnames |	保留类和类中的成员，防止它们被混淆，但当成员没有被引用时会被移除。|
| keepclassmembers |	只保留类中的成员，防止它们被混淆或移除。|
| keepclassmembernames |	只保留类中的成员，防止它们被混淆，但当成员没有被引用时会被移除。|
| keepclasseswithmembers |	保留类和类中的成员，防止它们被混淆或移除，前提是指名的类中的成员必须存在，如果不存在则还是会混淆。|
| keepclasseswithmembernames |	保留类和类中的成员，防止它们被混淆，但当成员没有被引用时会被移除，前提是指名的类中的成员必须存在，如果不存在则还是会混淆。|

除此之外，proguard中的通配符也比较让人难懂，proguard-android.txt中就使用到了很多通配符，我们来看一下它们之间的区别：

| 通配符 |      	描述      |
| :----- | :--------------- |
| <field> | 匹配类中的所有字段 |
| <method> | 匹配类中的所有方法 |
| <init> | 匹配类中的所有构造函数 |
| * |	匹配任意长度字符，但不含包名分隔符(.)。比如说我们的完整类名是com.example.test.MyActivity，使用com.*，或者com.exmaple.*都是无法匹配的，因为*无法匹配包名中的分隔符，正确的匹配方式是com.exmaple.*.*，或者com.exmaple.test.*，这些都是可以的。但如果你不写任何其它内容，只有一个*，那就表示匹配所有的东西。 |
| ** |	匹配任意长度字符，并且包含包名分隔符(.)。比如proguard-android.txt中使用的-dontwarn android.support.**就可以匹配android.support包下的所有内容，包括任意长度的子包。 |
| *** |	匹配任意参数类型。比如void set*(***)就能匹配任意传入的参数类型，*** get*()就能匹配任意返回值的类型。 |
| … |	匹配任意长度的任意类型参数。比如void test(…)就能匹配任意void test(String a)或者是void test(int a, String b)这些方法。 |

虽说上面表格已经解释的很详细了，但是很多人对于keep和keepclasseswithmembers这两个关键字的区别还是搞不懂。确实，它们之间用法有点太像了，我做了很多次试验它们的结果都是相同的。其实唯一的区别就在于类中声明的成员存不存在，我们还是通过一个例子来直接地看一下，先看keepclasseswithmember关键字：
```java
-keepclasseswithmember class * {
    native <methods>;
}
```

这段代码的意思其实很明显，就是保留所有含有native方法的类的类名和native方法名，而如果某个类中没有含有native方法，那就还是会被混淆。 
但是如果改成keep关键字，结果会完全不一样：
```java
-keep class * {
    native <methods>;
}
```

使用keep关键字后，你会发现代码中所有类的类名都不会被混淆了，因为keep关键字看到class *就认为应该将所有类名进行保留，而不会关心该类中是否含有native方法。当然这样写只会保证类名不会被混淆，类中的成员还是会被混淆的。 
比较难懂的用法大概就这些吧，掌握了这些内容之后我们就能继续前进了。 
回到Android Studio项目当中，刚才打出的APK虽然已经成功混淆了，但是混淆的规则都是按照proguard-android.txt中默认的规则来的，当然我们也可以修改proguard-android.txt中的规则，但是直接在proguard-android.txt中修改会对我们本机上所有项目的混淆规则都生效，那么有没有什么办法只针对当前项目的混淆规则做修改呢？当然是有办法的了，你会发现任何一个Android Studio项目在app模块目录下都有一个proguard-rules.pro文件，这个文件就是用于让我们编写只适用于当前项目的混淆规则的，那么接下来我们就利用刚才学到的所有知识来对混淆规则做修改吧。 
这里我们先列出来要实现的目标：

- 对MyFragment类进行完全保留，不混淆其类名、方法名、以及变量名。
- 对Utils类中的未调用方法进行保留，防止其被移除掉。
- 对第三方库进行保留，不混淆android-support库，以及LitePal库中的代码。

下面我们就来逐一实现这些目标。 
首先要对MyFragment类进行完全保留可以使用keep关键字，keep后声明完整的类名，然后保留类中的所有内容可以使用*通配符实现，如下所示：
```java
-keep class com.example.guolin.androidtest.MyFragment {
    *;
}
```

然后保留Utils类中的未调用方法可以使用keepclassmembers关键字，后跟Utils完整类名，然后在内部声明未调用的方法，如下所示：
```java
-keepclassmembers class com.example.guolin.androidtest.Utils {
    public void methodUnused();
}
```

最后不要混淆第三方库，目前我们使用了两种方式来引入第三方库，一种是通过本地jar包引入的，一种是通过remote引入的，其实这两种方式没什么区别，要保留代码都可以使用**这种通配符来实现，如下所示：
```java
-keep class org.litepal.** {
    *;
}

-keep class android.support.** {
    *;
}
```

所有内容都在这里了，现在我们重新打一个正式版的APK文件，然后再反编译看看效果： 
{% asset_img 07.png %}

可以看到，现在android-support包中所有代码都被保留下来了，不管是包名、类名、还是方法名都没有被混淆。LitePal中的代码也是同样的情况： 
{% asset_img 08.png %}

再来看下MyFragment中的代码，如下所示： 
{% asset_img 09.png %}

可以看到，MyFragment中的代码也没有被混淆，按照我们的要求被完全保留下来了。 
最后再来看一下Utils类中的代码： 
{% asset_img 10.png %}

很明显，Utils类并没有被完全保留下来，类名还是被混淆了，methodNormal()方法也被混淆了，但是methodUnused()没有被混淆，当然也没有被移除，因为我们的混淆配置生效了。 
经过这些例子的演示，相信大家已经对Proguard的用法有了相当不错的理解了，那么根据自己的业务需求来去编写混淆配置相信也不是什么难事了吧？ 
Progaurd的使用非常灵活，基本上能够覆盖你所能想到的所有业务逻辑。这里再举个例子，之前一直有人问我使用LitePal时的混淆配置怎么写，其实真的很简单，LitePal作为开源库并不需要混淆，上面的配置已经演示了如何不混淆LitePal代码，然后所有代码中的Model是需要进行反射的，也不能混淆，那么只需要这样写就行了：
```java
-keep class * extends org.litepal.crud.DataSupport {
    *;
}
```

因为LitePal中所有的Model都是应该继承DataSupport类的，所以这里我们将所有继承自DataSupport的类都进行保留就可以了。 
关于混淆APK的用法就讲这么多，如果你还想继续了解关于Proguard的更多用法，可以参考官方文档：http://proguard.sourceforge.net/index.html#manual/usage.html

## 3. 混淆Jar

在本篇文章的第二部分我想讲一讲混淆Jar包的内容，因为APK不一定是我们交付的唯一产品。就比如说我自己，我在公司是负责写SDK的，对于我来说交付出去的产品就是Jar包，而如果Jar包不混淆的话将会很容易就被别人反编译出来，从而泄漏程序逻辑。 
实际上Android对混淆Jar包的支持在很早之前就有了，不管你使用多老版本的SDK，都能在 <Android SDK>/tools目录下找到proguard这个文件夹。然后打开里面的bin目录，你会看到如下文件： 
{% asset_img 11.png %}

其中proguardgui.bat文件是允许我们以图形化的方式来对Jar包进行混淆的一个工具，今天我们就来讲解一下这个工具的用法。 
在开始讲解这个工具之前，首先我们需要先准备一个Jar包，当然你从哪里搞到一个Jar包都是可以的，不过这里为了和刚才的混淆逻辑统一，我们就把本篇文章中的项目代码打成一个Jar包吧。 
Eclipse中导出Jar包的方法非常简单，相信所有人都会，可是Android Studio当中就比较让人头疼了，因为Android Studio并没有提供一个专门用于导出Jar包的工具，因此我们只能自己动手了。 
我们需要知道，任何一个Android Studio项目，只要编译成功之后就会在项目模块的build/intermediates/classes/debug目录下生成代码编译过后的class文件，因此只需通过打包命令将这些class文件打包成Jar包就行了，打开cmd，切换到项目的根目录，然后输入如下命令：
```
jar -cvf androidtest.jar -C app/build/intermediates/classes/debug .
```

在项目的根目录下就会生成androidtest.jar这个文件，这样我们就把Jar包准备好了。 
现在双击proguardgui.bat打开混淆工具，如果是Mac或Ubuntu系统则使用sh proguardgui.sh命令打开混淆工具，界面如下图所示： 
{% asset_img 12.jpg %}

其实从主界面上我们就能看出，这个Proguard工具支持Shrinking、Optimization、Obfuscation、Preverification四项操作，在左侧的侧边栏上也能看到相应的这些选项。Proguard的工作机制仍然还是要依赖于配置文件，当然我们也可以通过proguardgui工具来生成配置文件，不过由于配置选项太多了，每个都去一一设置太复杂，而且大多数还都是我们用不到的配置。因此最简单的方式就是直接拿现有的配置文件，然后再做些修改就行了。 
那么我们从<Android SDK>/tools/proguard目录下将proguard-android.txt文件复制一份出来，然后点击主界面上的Load configuration按钮来加载复制出来的这份proguard-android.txt文件，完成后点击Next将进入Input/Output界面。 
Input/Output界面是用于导入要混淆的Jar包、配置混淆后文件的输出路径、以及导入该Jar包所依赖的所有其它Jar包的。我们要混淆的当然就是androidtest.jar这个文件，那么这个Jar包又依赖了哪些Jar包呢？这里就需要整理一下了。

- 首先我们写的都是Java代码，Java代码的运行要基于Jre基础之上，没有Jre计算机将无法识别Java的语法，因此第一个要依赖的就是Jre的rt.jar。
- 然后由于我们导出的Jar包中有Android相关的代码，比如Activity、Fragment等，因此还需要添加Android的编译库，android.jar。
- 除此之外，我们使用的AppCompatActivity和Fragment分别来自于appcompat-v7包和support-v4包，那么这两个Jar包也是需要引入的。
- 最后就是代码中还引入了litepal-1.3.1.jar。

整理清楚了之后我们就来一个个添加，Input/Output有上下两个操作界面，上面是用于导入要混淆的Jar包和配置混淆后文件的输出路径的，下面则是导入该Jar包所依赖的所有其它Jar包的，全部导入后结果如下图所示： 
{% asset_img 13.jpg %}

这些依赖的Jar包所存在的路径每台电脑都不一样，你所需要做的就是在你自己的电脑上成功找到这些依赖的Jar包并导入即可。 
不过细心的朋友可能会发现，我在上面整理出了五个依赖的Jar包，但是在图中却添加了六个。这是我在写这篇文章时碰到的一个新的坑，也是定位了好久才解决的，我觉得有必要重点提一下。由于我平时混淆Jar包时里面很少会有Activity，所以没遇到过这个问题，但是本篇文章中的演示Jar包中不仅包含了Activty，还是继承自AppCompatActivity的。而AppCompatActivity的继承结构并不简单，如下图所示： 
{% asset_img 14.png %}

其中AppCompatActivity是在appcompat-v7包中的，它的父类FragmentActivity是在support-v4包中的，这两个包我们都已经添加依赖了。但是FragmentActivity的父类就坑爹了，如果你去看BaseFragmentActivityHoneycomb和BaseFragmentActivityDonut这两个类的源码，你会发现它们都是在support-v4包中的： 

```java
package android.support.v4.app;
import ...

/**
 * Base class for {@code FragmentActivity} to be able to use v11 APIs.
 */
abstract class BaseFragmentActivityHoneycomb extends BaseFragmentActivityDonut {
```

```java
package android.support.v4.app;
import ...

/**
 * Base class for {@code FragmentActivity} to be able to use Donut APIs.
 */
abstract class BaseFragmentActivityDonut extends Activity {

```

可是如果你去support-v4的Jar包中找一下，你会发现压根就没有这两个类，所以我当时一直混淆报错就是因为这两个类不存在，继承结构在这里断掉了。而这两个类其实被规整到了另外一个internal的Jar包中，所以当你要混淆的Jar包中有Activity，并且还是继承自AppCompatActivity或FragmentActivity的话，那么就一定要记得导入这个internal Jar包的依赖，如下图所示： 
{% asset_img 17.jpg %}

接下来点击Next进入Shrink界面，这个界面没什么需要配置的东西，但记得要将Shrink选项钩掉，因为我们这个Jar包是独立存在的，没有任何项目引用，如果钩中Shrink选项的话就会认为我们所有的代码都是无用的，从而把所有代码全压缩掉，导出一个空的Jar包。 
继续点击Next进入Obfuscation界面，在这里可以添加一些混淆的逻辑，和混淆APK时不同的是，这里并不会自动帮我们排除混淆四大组件，因此必须要手动声明一下才行。点击最下方的Add按钮，然后在弹出的界面上编写排除逻辑，如下图所示： 
{% asset_img 18.jpg %}

很简单，就是在继承那一栏写上android.app.Activity就行了，其它的组件原理也相同。 
继续点击Next进入Optimiazation界面，不用修改任何东西，因为我们本身就不启用Optimization功能。继续点击Next进入Information界面，也不用修改任何东西，因为我们也不启用Preverification功能。 
接着点击Next，进入Process界面，在这里可以通过点击View configuration按钮来预览一下目前我们的混淆配置文件，内容如下所示：

```java
-injars /Users/guolin/AndroidStudioProjects/AndroidTest/androidtest.jar
-outjars /Users/guolin/androidtest_obfuscated.jar

-libraryjars /Library/Java/JavaVirtualMachines/jdk1.7.0_79.jdk/Contents/Home/jre/lib/rt.jar
-libraryjars /Users/guolin/Library/Android/sdk/platforms/android-23/android.jar
-libraryjars /Users/guolin/AndroidStudioProjects/AndroidTest/app/build/intermediates/exploded-aar/com.android.support/appcompat-v7/23.2.0/jars/classes.jar
-libraryjars /Users/guolin/AndroidStudioProjects/AndroidTest/app/build/intermediates/exploded-aar/com.android.support/support-v4/23.2.0/jars/classes.jar
-libraryjars /Users/guolin/AndroidStudioProjects/AndroidTest/app/build/intermediates/exploded-aar/com.android.support/support-v4/23.2.0/jars/libs/internal_impl-23.2.0.jar
-libraryjars /Users/guolin/AndroidStudioProjects/AndroidTest/app/libs/litepal-1.3.1.jar

-dontshrink
-dontoptimize
-dontusemixedcaseclassnames
-keepattributes *Annotation*
-dontpreverify
-verbose
-dontwarn android.support.**


-keep public class com.google.vending.licensing.ILicensingService

-keep public class com.android.vending.licensing.ILicensingService

# keep setters in Views so that animations can still work.
# see http://proguard.sourceforge.net/manual/examples.html#beans
-keepclassmembers public class * extends android.view.View {
    void set*(***);
    *** get*();
}

# We want to keep methods in Activity that could be used in the XML attribute onClick
-keepclassmembers class * extends android.app.Activity {
    public void *(android.view.View);
}

-keepclassmembers class * extends android.os.Parcelable {
    public static final android.os.Parcelable$Creator CREATOR;
}

-keepclassmembers class **.R$* {
    public static <fields>;
}

-keep class * extends android.app.Activity

-keep class * extends android.app.Service

-keep class * extends android.content.BroadcastReceiver

-keep class * extends android.content.ContentProvider

# Also keep - Enumerations. Keep the special static methods that are required in
# enumeration classes.
-keepclassmembers enum  * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}

# Keep names - Native method names. Keep all native class/method names.
-keepclasseswithmembers,allowshrinking class * {
    native <methods>;
}
```

恩，由此可见其实GUI工具只是给我们提供了一个方便操作的平台，背后工作的原理还是通过这些配置来实现的，相信上面的配置内容大家应该都能看得懂了吧。 
接下来我们还可以点击Save configuration按钮来保存一下当前的配置文件，这样下次混淆的时候就可以直接Load进来而不用修改任何东西了。 
最后点击Process!按钮来开始混淆处理，中间会提示一大堆的Note信息，我们不用理会，只要看到最终显示Processing completed successfully，就说明混淆Jar包已经成功了，如下图所示： 
{% asset_img 19.jpg %}

混淆后的文件我将它配置在了/Users/guolin/androidtest_obfuscated.jar这里，如果反编译一下这个文件，你会发现和刚才反编译APK得到的结果是差不多的：MainActivity的类名以及从父类继承的方法名不会被混淆，NativeUtils的类名和其中的native方法名不会被混淆，Utils的methodUnsed方法不会被移除，因为我们禁用了Shrink功能，其余的代码都会被混淆。由于结果实在是太相似了，我就不再贴图了，参考本篇文章第一部分的截图即可。

本文转载自：http://blog.csdn.net/guolin_blog/article/details/50451259

