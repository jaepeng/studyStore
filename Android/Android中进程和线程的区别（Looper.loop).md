# Android中进程和线程的区别（Looper.loop)

按照操作系统中的描述。线程是 CPU 调度的最小单元，同时线程也是一种有限的资源。而进程一般指一个执行单元，在 PC 和移动设备上指一个程序或者一个应用。一个进程可以包含多个线程。

**进程是资源分配的最小单位，线程是CPU调度的最小单位**

[参考链接](https://www.zhihu.com/question/34652589/answer/90344494)

## 进程

1. 每个app运行时前首先创建一个进程，该进程是由**Zygote fork出来的**，用于承载App上运行的各种Activity/Service等组件。进程对于上层应用来说是完全透明的，这也是google有意为之，让App程序都是运行在Android Runtime。大多数情况一个App就运行在一个进程中，除非在AndroidManifest.xml中配置Android:process属性，或通过native代码fork进程。

2. 组件运行在哪个进程中，是在 AndroidManifest 文件中进行设置的，activity、service、receiver 和 provider 均支持 `android:process` 属性，此属性可以指定该组件应在哪个进程运行。我们可以设置此属性，使每个组件均在各自的进程中运行。

3. Android 进程的五种优先级**（具体查看Android开发小知识点中的[进程状态](D:\个人文件\studyStore\Android\关于Android小知识.md)部分）**

   **1、前台进程 — Foreground process**

   **2、可见进程 — Visible process**

   **3、服务进程 — Service process**

   **4、后台进程 — Background process**

   **5、空进程 — Empty process**

## 线程

1. 线程对应用来说非常常见，比如每次new Thread().start都会创建一个新的线程。该线程与App所在进程之间资源共享，**从Linux角度来说进程与线程除了是否共享资源外，并没有本质的区别**，都是一个task_struct结构体**，在CPU看来进程或线程无非就是一段可执行的代码，CPU采用CFS调度算法，保证每个task都尽可能公平的享有CPU时间片**。
2. 线程在 Android 中是一个很重要的概念，从用途上来说，线程分为主线程和子线程，主线程的作用是「运行四大组件以及处理它们和用户的交互」，而子线程的作用则是「执行耗时任务，比如网络请求、I/O 操作等」，由于 Android 的特性，如果在主线程中执行耗时操作那么就会导致程序无法及时地响应。因此耗时操作必须放在子线程中执行。
3. 除了 Thread 本身以外，在 Android 中可以扮演线程角色的还有很多，比如 AsyncTask 和 IntentService，同时 HandlerThread 也是一种特殊的线程。尽管 AsyncTask、IntentService 以及 HandlerThread 的「表现形式」都有别于传统的线程，但是它们的本质仍然是传统的线程。

## 如何保证线程一直被执行下去

> **简单做法就是可执行代码是能一直执行下去的，死循环便能保证不会被退出**

1. 真正会卡死主线程的操作是在回调方法onCreate/onStart/onResume等操作时间过长，会导致掉帧，甚至发生ANR，looper.loop本身不会导致应用卡死。
2. ActiviyThread.main()中的`Looper.prepareMainLooper();`就是创建一个死循环
3. **ActivityThread实际上并非线程**，不像HandlerThread类，ActivityThread并没有真正继承Thread类，只是往往运行在主线程，该人以线程的感觉，其实承载ActivityThread的主线程就是由Zygote fork而创建的进程。

### 主线程的死循环一直运行是不是特别消耗CPU资源呢？

其实不然，这里就涉及到**Linux pipe/epoll机制**，简单说就是在主线程的MessageQueue没有消息时，便**阻塞在loop的queue.next()中的nativePollOnce()方法里**，详情见[Android消息机制1-Handler(Java层)](https://link.zhihu.com/?target=http%3A//www.yuanhh.com/2015/12/26/handler-message-framework/%23next)，此时**主线程会释放CPU资源进入休眠状态**，直到下个消息到达或者有事务发生，通过往pipe管道写端写入数据来唤醒主线程工作。这里采用的epoll机制，是一种IO多路复用机制，可以同时监控多个描述符，当某个描述符就绪(读或写就绪)，则立刻通知相应程序进行读或写操作，本质同步I/O，即读写是阻塞的。 **所以说，主线程大多数时候都是处于休眠状态，并不会消耗大量CPU资源。**

### Activity的生命周期是怎么实现在死循环体外能够执行起来的？

1. ActivityThread的内部类H继承于Handler，通过handler消息机制，简单说Handler机制用于同一个进程的线程间通信。
2. **Activity的生命周期都是依靠主线程的Looper.loop，当收到不同Message时则采用相应措施：**
   1. 在H.handleMessage(msg)方法中，根据接收到不同的msg，执行相应的生命周期。
   2. 比如收到msg=H.LAUNCH_ACTIVITY，则调用ActivityThread.handleLaunchActivity()方法，最终会通过反射机制，创建Activity实例，然后再执行Activity.onCreate()等方法；
   3. 再比如收到msg=H.PAUSE_ACTIVITY，则调用ActivityThread.handlePauseActivity()方法，最终会执行Activity.onPause()等方法。

