---
title: kotlin协程入门
date: 2020-06-04 19:31:10
tags: [Android, kotlin]
categories: kotlin
---

## 1. 认识协程

### 1.1 进程

早期的计算机运行程序还是只能一次运行一个任务，之后进程的出现实现了近似同步的执行效果，其本质上是程序的交替执行。为了保证进程中的程序能够正常执行，还会有一些存储进程状态的保存集。随着硬件的发展和多CPU的出现，能够同时执行的进程数量逐渐增多。这就带来了一个问题，即用来存储进程状态的集合所占用的资源比一个进程可以执行的资源还要多，相当于整个系统大半的进行都是用来保存进程的状态。

### 1.2 线程

线程的提出有效的解决了这个问题。进程不再频繁的切换，而是先执行，遇到阻塞的话暂时不管，继续执行其他的任务，当其他任务执行完之后再回过头来看阻塞任务是否执行完。多线程的缺陷在于无法自主控制调度，除开一定会执行的主线程之外，其他线程的执行顺序都无法控制，在Java上是由Java虚拟机调度，其他平台大多是由系统控制。

### 1.3 协程

线程执行过程中发生线程切换的时候会损耗一定的资源，这部分资源用来保存线程的状态。执行过程中如果发生了磁盘读写或网络请求这样的IO操作的时候线程的执行会被阻塞，但同时该线程还会持有CPU资源，这就造成了一定了资源浪费。理想的情况是在发送阻塞的时候，该线程主动交出CPU给其他线程使用或者给内部的其他任务。

这种方式其实就是协程的体系。通过提升CPU利用率，减少线程切换，进而提升程序运行效率。

延伸开来协程主要有三个特性。

- 第一个是可控制，不同于线程协程能做到可被控制的发起子任务；
- 第二个是轻量级，协程非常小、占用资源比线程还少，在JVM平台上它的本质就是一次方法的调用；
- 第三个是语法糖，目前能够使用协程的语言都提供了很好的语法糖支持，使多任务或多线程切换不在使用回调语法。

<!-- more -->

### 1.4 举例：用户登录

用户登录在应用开发中可以算是一个很常见的场景，具体的逻辑是这样的，首先向第三方平台请求用户token，然后将token和自身平台上的用户账号关联起来，最后获取用户信息展示到UI界面上。

```kotlin
fun login() {
    requestToken { token ->
        requestUserInfo(token) { user ->
            setText(user.name)
        }
    }
}
```
对此最常见的做法是采用回调的形式。requestToken会先发出一次网络请求，请求返回后执行回调并传入token，回调内部又会用token作为参数向服务器发起请求获得用户信息，最终完成用户信息在UI上的改变。这种方式的问题在于嵌套层级过多，链路一旦过长看起来会非常复杂。

用协程改写

```kotlin
fun login() = runBlocking {
    val token = requestToken()
    val user = requestUserInfo(token)
    setText(user.name)
}
```

### 1.5 举例：对比线程的执行效率

运行以下代码：

```kotlin
fun testCoroutineRepeat() {
    // 启动大量的协程
    runBlocking {
        repeat(100_000) {
            launch {
                delay(1000L)
                print(".")
            }
        }
    }
}
```
它启动了 10 万个协程，并且在一秒钟后，每个协程都输出一个点。输出结果如下：

{% asset_img coroutine.png %}

再尝试使用线程来实现，会发生什么

```kotlin
fun testThreadRepeat() {
    // 启动大量的线程
    repeat(100_000) {
        Thread(Runnable {
            Thread.sleep(1000L)
            print(".")
        }).start()
    }
}
```
输出结果：

{% asset_img thread.png %}

可以看到，用线程执行花了47秒多，用协程执行只花了2秒多，**协程的执行效率比线程高20倍！**


## 2. 启动一个协程

在Kotlin中常用的启动协程的方式有三种

- 第一种是runBlocking，会阻塞调用它的线程，直到 runBlocking内部的协程执行完毕。一般用在协程和线程的交接点，也就是通常只用于启动最外层协程。
- 第二种是launch，不阻塞调用它的线程，一般用于在协程内部再启动一个协程。
- 第三种是async/await，会阻塞调用它的线程，它不仅可以启动协程，还可以得到协程里面执行的结果。


```kotlin
@Test
fun testRunBlocking() {
    runBlocking {
        delay(1000)
        println("runBlocking")
    }
    println("test end.")
}

```
执行结果如下，可以看到runBlocking会阻塞调用它的线程，直到runBlocking协程执行完毕，才开始执行线程后面的代码。
{% asset_img test_runblocking.png %}


```kotlin
@Test
fun testLaunch() = runBlocking {
    launch {
        delay(1000)
        println("launch")
    }
    println("test end.")
}

```
执行结果如下，可以看到launch启动的协程不会阻塞调用它的线程，runBlocking也会保证内部的子协程都执行完毕，才会结束runBlocking协程，当然要保证在同一个CoroutineScope（协程上下文，后面会介绍）。
{% asset_img test_launch.png %}

```kotlin
@Test
fun testAsync() = runBlocking {
    val getNum = GlobalScope.async {
        delay(1000)
        println("launch")
        100
    }
    println("get num=" + getNum.await())
    println("test end.")
}
```
执行结果如下，可以看到async启动的协程会阻塞调用它的线程，并且还可以通过await方法得到async里面的返回值。
{% asset_img test_async.png %}


## 3. 协程调度器

协程调度器可以将协程限制在一个特定的线程执行，或将它分派到一个线程池，亦或是让它不受限地运行。

Kotlin 提供了三个调度程序，可以使用它们来指定应在哪个线程环境运行协程：

- Dispatchers.Main - 使用此调度程序可在 Android 主线程上运行协程。此调度程序只能用于与界面交互和执行快速工作。示例包括调用 suspend 函数、运行 Android 界面框架操作，以及更新 LiveData 对象。
- Dispatchers.IO - 此调度程序经过了专门优化，适合在主线程之外执行磁盘或网络 I/O。示例包括使用 Room 组件、从文件中读取数据或向文件中写入数据，以及运行任何网络操作。
- Dispatchers.Default - 此调度程序经过了专门优化，适合在主线程之外执行占用大量 CPU 资源的工作。用法示例包括对列表排序和解析 JSON。

所有的协程启动函数都可以接收一个可选的 CoroutineContext 参数，它可以用来指定新协程运行在哪个环境。或者在协程内部使用 withContext(context CoroutineContext) 来切换运行环境。withContext() 不会增加额外的开销。此外，在某些情况下，还可以优化 withContext() 调用，使其超越基于回调的等效实现。

```kotlin
fun testCoroutineContext() = runBlocking {
    println("runBlocking working thread ${Thread.currentThread().name}")
    launch { // 运行在父协程的上下文中，即跟 runBlocking 运行的线程环境一样
        println("launch1 working thread ${Thread.currentThread().name}")
        withContext(Dispatchers.IO) {
            println("launch1 switched thread ${Thread.currentThread().name}")
        }
    }
    launch(Dispatchers.Unconfined) { // 不受限的——将工作在主线程中
        println("launch2 working thread ${Thread.currentThread().name}")
    }
    launch(Dispatchers.Default) { // 将会获取默认调度器
        println("launch3 working thread ${Thread.currentThread().name}")
    }
    launch(newSingleThreadContext("MyOwnThread")) { // 将使它获得一个新的线程
        println("launch4 working thread ${Thread.currentThread().name}")
    }    
}
```

{% asset_img coroutine_context.png %}


## 4. 协程上下文

在定义协程时还必须指定其上下文CoroutineScope。CoroutineScope 可管理一个或多个相关的协程。还可以使用 CoroutineScope 在该范围内启动新的协程。不过，与调度程序不同，CoroutineScope 不运行协程。

CoroutineScope 的一项重要功能就是在用户离开你的应用时停止执行协程，确保所有正在运行的操作都能正确停止。比如在 Android 的 activity 上下文中启动了一组协程来使用异步拉取并更新数据，所有这些协程必须在这个 activity 销毁的时候停止以避免内存泄漏。

CoroutineScope 可以通过 CoroutineScope() 创建或者通过MainScope() 工厂函数。前者创建了一个通用作用域，而后者为使用 Dispatchers.Main 作为默认调度器的 UI 应用程序 创建作用域：

```kotlin
class Activity {
    private val scop = CoroutineScope(Dispatchers.IO)

    fun refreshData() = runBlocking {
          scope.launch {
              val data = fetchData()
              withContext(Dispatchers.Main) {
                  updateViews(data)
              }
          }
    }

    fun destroy() {
        // 在activity销毁时，协程也能正确停止
        scope.cancel()
    }
    // 继续运行……

```

或者，我们可以在这个 Activity 类中实现 CoroutineScope 接口，使协程上下文与 activity 的生命周期相关联，方法如下：

```kotlin
class Activity : CoroutineScope by CoroutineScope(Dispatchers.Default) {
    fun refreshData() = runBlocking {
          // 这样我们在launch启动协程时不用指定协程上下文
          launch {
              val data = fetchData()
              withContext(Dispatchers.Main) {
                  updateViews(data)
              }
          }
    }

    fun destroy() {
        // 在activity销毁时，协程也能正确停止
        cancel()
    }
    // 继续运行……
```

## 5. 实际案例

在项目开发中遇到这个一个场景：

搜索设备这个操作是阻塞的，在搜索时，需要在UI上展示一个loading弹框，搜索结束后关闭弹窗。如果搜索结果返回太快，loading弹窗一闪而过，体验不好；如果搜索很长时间都不返回，需要加一个超时时间。搜索接口是不提供设置最短时间或超时时间，需要自己另外实现。

用协程实现起来就比较方便了，如下：

```kotlin
fun refreshDevice() = runBlocking {
    try {
        // 设置10s超时
        withTimeout(10000) {
            val scanDevices = async { scanDevices() }
            // 最短时间不小于3s
            val taskMinTime = async { delay(3000) }
            // 两个async 启动的协程是并行的，同时await是阻塞的，这样就保证了执行scanDevices最少需要3s，否则会阻塞在这里
            taskMinTime.await()
            val deviceList = scanDevices.await()
            ...
        }
    } catch (e: TimeoutCancellationException) {
        // 超时逻辑处理
        ...
    }
}

```
