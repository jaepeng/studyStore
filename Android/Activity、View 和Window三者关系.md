# Activity、View 和Window三者关系

[Activity、View、Window的理解一篇文章就够](https://blog.csdn.net/zane402075316/article/details/69822438?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control)

1. Activity中新建一个PhoneWindow对象。

2. Window是一个抽象的概念，主要是管理 View 的创建，以及与 ViewRootImpl 的交互，将 Activity 与 View 解耦。
3. View的显示，是以Activity为载体的。

3. PhoneWindow是**Window**的具体实现类，Activity和Dialog中的Window对象都是PhoneWindow。同时得到(关联)一个WindowManager对象，WIndowManager是一个抽象类，它的具体实现是在WindowManagerImpl 。**每个 Activity 会有一个 WindowManager 对象**，这个 mWindowManager 就是和 [WindowManagerService](#WMS) 进行通信，也是 WindowManagerService 识别 View 具体属于那个 Activity 的**关键**，创建时传入 IBinder 类型的 mToken。这个 Activity 的 mToken，这个 mToken 是一个 IBinder，WindowManagerService 就是通过这个 IBinder 来管理 Activity 里的 View。

4. 一个 Activity 对应一个 Window 也就是 PhoneWindow，**一个 PhoneWindow 持有一个 DecorView 的实例，DecorView 本身是一个 FrameLayout。**

![这里写图片描述](https://gitee.com/pengjae/pic/raw/master/img/20201126174401.png)

1. Activity是在ActivityThread的performLaunchActivity中进行创建的，在创建完成之后就会调用其 attach方法

2. attach 方法，这个方法是优先于 onCreate、onStart 等生命周期函数的，attach 方法所做的就是 new 一个 PhoneWindow 并关联 WindowManager。

3. onCreate()这一步就是把我们的布局文件解析成 View 塞到 DecorView 的一个 id 为 R.id.content 的 mContentView 中。

   1. DecorView 本身是一个 FrameLayout，它还承载了 StatusBar 和 NavigationBar。
   2. 然后在 handleResumeActivity 中，通过 WindowManagerImpl 的 addView 方法把 DecorView 添加进去，实际实现是 WindowManagerGlobal 的 addView 方法，
   3. addView 它里面管理着所有的 DecorView 及其对应的 ViewRootImpl
   4. ViewRootImpl 是 DecorView 的管理者，它负责 View 树的测量、布局、绘制，以及通过 Choreographer 来控制 View 的刷新。

4. <span id="WMS">**WindowManagerService**（WMS）</span>是所有 **Window 窗口的管理者**，它负责 Window 的添加和删除、Surface 的管理和事件分发等等，因此每一个 Activity 中的 PhoneWindow 对象如果需要显示等操作，就需要要与 WMS 交互才能进行。这一步是在 ViewRootImpl 的 setView 方法中，会调用 requestLayout，并且通过 WindowSession 的 addToDisplay 与 WMS 进行交互，WMS 会为每一个 Window 关联一个 WindowState。除此之外，ViewRootImpl 的 setView 还做了一件重要的事就是注册 InputEventReceiver，这和 View 事件分发有关。

   ![img](https://gitee.com/pengjae/pic/raw/master/img/20201202140502.webp)

