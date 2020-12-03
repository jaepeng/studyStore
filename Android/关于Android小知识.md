# 关于Android小知识

[TOC]



## AlertDialog

1. AlertDialog.Builder的create() 和show()方法都返回AlertDialog对象
2. show()方法创建并显示对话框
3. AlertDialog不能直接用new关键字构建对象,而必须使用其内部类Builder



## ContentProvider

1. ContentProvider通过Binder机制实现应用间数据共享
2. ContentProvider通过URI来区分外界要访问的数据集合
3. 一个应用可以提供多个ContentProvider

## android中使用SQLiteOpenHelper这个辅助类时，可以生成一个数据库，并可以对数据库进行管理的方法可以是?

1. getReadableDatabase()
2. getWriteableDatabase()

>Android使用 getWritableDatabase() 和getReadableDatabase()方法都可以获取一个用于操作数据库的SQLiteDatabase实例。(getReadableDatabase()方法中会调用getWritableDatabase()方法)getReadableDatabase()并不是以只读方式打开数据库，而是**先执行getWritableDatabase()**，失败的情况下才以只读方式打开数据库.。但getWritableDatabase()方法以读写方式打开数据库，一旦数据库的磁盘空间满了，数据库就只能读而不能写，getWritableDatabase()打开数据库就会出错。getReadableDatabase()方法先以读写方式打开数据库，倘若使用如果数据库的磁盘空间满了，就会打开失败，当打开失败后会继续尝试以只读方式打开数据库.

## 关于IntentService与Service的关系描述错误的是

1. IntetnService继承自Service，自然也继承了Service的所有方法。
2. 启动方式都是startService
3. IntentService是继承Service的，那么它包含了Service的全部特性，当然也包含service的生命周期，那么与service不同的是，IntentService在执行onCreate操作的时候，内部开了一个线程，去你执行你的耗时操作。
4. IntentService任务执行完后会自动停止，service不会自动停止
5. 

## 关于res/raw目录说法正确的是？

1. 这里的文件是原封不动的存储到设备上，不会转换为二进制的格式
2. 这里的文件没有目录结构
3. 这里的文件都会在R.java中生成唯一ID

## Android 的MVC

 Android中界面部分也采用了当前比较流行的MVC框架，在Android中： 
　　1. **视图层（View）**：一般采用XML文件进行界面的描述，使用的时候可以非常方便的引入。当然，如何你对Android了解的比较的多了话，就一定可以想到在Android中也可以使用JavaScript+HTML等的方式作为View层，当然这里需要进行Java和JavaScript之间的通信，幸运的是，Android提供了它们之间非常方便的通信实现。     
　　2. **控制层（Controller）**：Android的控制层的重任通常落在了众多的Acitvity的肩上，这句话也就暗含了不要在Acitivity中写代码，要通过Activity交割Model业务逻辑层处理，这样做的另外一个原因是Android中的Acitivity的响应时间是5s，如果耗时的操作放在这里，程序就很容易被回收掉。
　　3. **模型层（Model）**：对数据库的操作、对网络等的操作都应该在Model里面处理，当然对业务计算等操作也是必须放在的该层的。就是应用程序中二进制的数据。
      在Android SDK中的数据绑定，也都是采用了与MVC框架类似的方法来显示数据。在控制层上将数据按照视图模型的要求（也就是Android SDK中的Adapter）封装就可以直接在视图模型上显示了，从而实现了数据绑定。比如显示Cursor中所有数据的ListActivity，其视图层就是一个ListView，将数据封装为ListAdapter，并传递给ListView，数据就在ListView中现实。 

## 退出Activity的方式

1. **对于单一Activity：**
   1. 调用finish()
   2. killProcess()和System.exit()这样的方法。
2. **对于多个Activity：**
   1. 抛异常强制退出：
   2. 发送特定广播
   3. 递归退出
3. 除了以上，也可以建立一个集合类用于同一管理所有的活动

## 创建Menu需要重写的方法

上下文菜单（通过在某元素上长按，来呼出菜单）
选项菜单（通过按手机上的菜单按钮，来呼出菜单） 

重写 **onCreateContextMenu** 用以创建上下文菜单
重写 **onContextItemSelected** 用以响应上下文菜单 

重写 **onCreateOptionsMenu** 用以创建选项菜单
重写 **onOptionsItemSelected** 用以响应选项菜单

当每次Menu显示时，会调用方法**onPrepareOptionsMenu**，也可以在菜单每次被调用时，对菜单中的项重新生成，通过重载**onPrepareOptionsMenu**来实现,由于每次调用时都要重新生成，对于那些不经常变化的菜单，效率就会比较低。

调用Menu.**addSubMenu**()方法，为某个菜单项添加子菜单

## android NDK

1. NDK是一系列工具的集合
2. NDK 提供了一份稳定、功能有限的 API 头文件声明
3. 使 “Java+C” 的开发方式终于转正，成为官方支持的开发方式

## Handler

[参考链接](https://blog.csdn.net/weixin_34209406/article/details/87997113?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.edu_weight&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.edu_weight)

1. 一个线程只能有一个Handler和Looper(通过ThreadLocal保证只有一个looper)
2. Looper(Looper.prepare())创建MessageQueue
3. Looper(Looper.loop())创建Loop
4. Handler内部有一个空的Runnable叫做Callable
   1. 如果是空的构造函数Hanlder(),则Callable为null,
   2. 如果postDelayed()方法,则会将传入的Runnable r 赋值给callable

## Android 手机安装的应用都是以apk的形式来进行的。

1. 他实质上是个压缩包。
2. src下的.java文件会被编译成dex字节码文件（代码类文件 classes.dex)
3. 还包含开发过程中的各种资源文件
4. AndroidManifest.xml

## onActivityResult和SetResult

[参考链接](https://blog.csdn.net/han_wen2015/article/details/49718475)

[参考链接——带代码实例](https://www.cnblogs.com/fuck1/p/5456337.html)

1. 需要返回数据或结果的，则使用**startActivityForResult** (Intent intent, intrequestCode)，requestCode的值是自定义的，用于识别跳转的目标Activity。

2. 跳转的目标Activity所要做的就是返回数据/结果，**setResult**(int resultCode)只返回结果不带数据，或者**setResult**(int resultCode, Intent data)两者都返回！而接收返回的数据/结果的处理函数是**onActivityResult**(intrequestCode, int resultCode, Intent data)，这里的requestCode就是startActivityForResult的requestCode，resultCode就是setResult里面的resultCode，返回的数据在data里面



## ContentProvider和ContentResolver

> 其他应用通过ContentResolver操作（query）另一个应用程序通过ContentProvider共享的数据

ContentProvider的主要目的是对外暴露数据提供其他程序查询

## 数据持久化方式:

1. sharedpreference
2. 内部存储(openFileOutput()方法打开一个文件的输入输出流)
3. SQLite Database
4. 网络连接(将数据放在服务器上)
5. 外部存储(SD卡)

## 所有线程执行到某一处,才进行后面的代码执行我们可以使用?

>CountDownLatch

## RatingBar组件不能用属性直接设置的是?

> 五角星的色彩

## 哪些方法可以指定目标组件?

` 题目解析:显示intent启动Activity有什么方法`

1. 构造方法:new Intent();

   ```java
   Intent intent = new Intent(Context packageContext,Class<?> cls);  
   //前者为启动活动的上下文，后者为想要启动的目标活动 
   Intent intent = new Intent(FirstActivity.this,SecondActivity.class);
   ```

2. setComponent方法

   ```java
   ComponentName componentName = new ComponentName(this, SecondActivity.class);  
   Intent intent = new Intent();  
   intent.setComponent(componentName);  
   startActivity(intent);
   ```

3. setClass/setClassName方法

   ```java
   Intent intent = new Intent();  
   intent.setClass(this, SecondActivity.class);  
   // 或者intent.setClassName(this, "top.longsh1z.www.SecondActivity");  
   // 或者intent.setClassName(this.getPackageName(), "top.longsh1z.www.SecondActivity");        
   startActivity(intent);
   ```

4. 总结:

   ```java
   Intent intent = new Intent(aactivity.this,bactivity.class);
   startActivity(intent); //显示指定目标组件，
   Intent intent = new Intent();
   intent.setClass(aactivity.this,bactivity.class);
   startActivity(intent);//显示指定目标组件与第一个效果一样
   intent.setClassName(包名，包名➕activity);
   startActivity(intent)//这个打开不同Application的activity；
   ```

   #### AlertDialog和ProgressDialog

   ##### AlertDialog

   1. 用create()方法创建

   2. show()和create()都返回AlertDialog对象

   ##### ProgressDialog

   1. ProgressDialog用Build来创建

## Service的启动方式

1. 用**startService() **启动的Service启动后和Activity就无关了.就算Activity销毁了,也会在后台继续执行
2. 而**bindService()**启动的Service如果Activity挂了之后,他也会挂掉.并且要记得注销掉它.不然会造成资源浪费
3. 先startService()启动再用bindService()方式绑定那如何销毁?
   1. 这样启动的优势：
      1. 服务既可以长期在后台运行，又可以与服务进行通信。
   2. 需要先进行解绑onUnbind(解绑服务，否则**无法停止服务**的（但是服务，仍然在后台运行）)
   3. 再在不需要使用时调用stopService()

## Android dvm的进程和Linux的进程

1. 它们都是进程的一种

2. dvm是android的虚拟机

3. dvm中可以创建多个进程来处理应用间的同步问题

4. 每个 Android 应用程序都运行在单独的 Dalvik 虚拟机内（即每个 Android 应用程序对应一条 Dalvik 进程）， Dalvik 专门针对同时高效地运行多个虚拟机进行优化，因此 Android 系统以方便的实现对应用程序进行隔离。每一个**DVM**都是在Linux 中的一个进程.

5. Android 运行时由两部分组成： 

   1. **Android 核心库集**:提供了 Java 语言核心库所能使用的绝大部分功能
   2. **Dalvik 虚拟机**:而虚拟机则负责运行 Android 应用程序。

6. **DVM**指dalivk的虚拟机，每一个Andriod应用程序都在它自己的进程中运行，都拥有一个独立的Dalivk虚拟机实例，而每一个DVM都是在Linux中的一个进程，所以说可以认为是同一个概念

## Android 现阶段架构

Android 架构:

1. Linux Kernel(Linux内核)、
2. Hardware Abstraction Layer(硬件抽象层)、
3. Libraries(系统运行库或者是c/c++ 核心库)、
4. Application Framework(开发框架包 )、
5. Applications(核心应用程序)



## Android 数据库打开方式

Android使用 getWritableDatabase() 和getReadableDatabase()方法都可以获取一个用于操作数据库的SQLiteDatabase实例。(getReadableDatabase()方法中会调用getWritableDatabase()方法)getReadableDatabase()并不是以只读方式打开数据库，而是**先执行getWritableDatabase()**，失败的情况下才以只读方式打开数据库.。

但getWritableDatabase()方法以读写方式打开数据库，一旦数据库的磁盘空间满了，数据库就只能读而不能写，getWritableDatabase()打开数据库就会出错。

getReadableDatabase()方法先以读写方式打开数据库，
倘若使用如果数据库的磁盘空间满了，就会打开失败，当打开失败后会继续尝试以只读方式打开数据库.

## Android 新建菜单Menue

1. 上下文菜单（通过在某元素上长按，来呼出菜单）
2. 选项菜单（通过按手机上的菜单按钮，来呼出菜单） 
3. 重写 **onCreateContextMenu** 用以创建上下文菜单
4. 重写 **onContextItemSelected** 用以响应上下文菜单 
5. 重写 **onCreateOptionsMenu** 用以创建选项菜单
6. 重写 **onOptionsItemSelected** 用以响应选项菜单

当每次Menu显示时，会调用方法**onPrepareOptionsMenu**，也可以在菜单每次被调用时，对菜单中的项重新生成，通过重载**onPrepareOptionsMenu**来实现,由于每次调用时都要重新生成，对于那些不经常变化的菜单，效率就会比较低。调用Menu.**addSubMenu()**方法，为某个菜单项添加子菜单

###  android中有三种菜单

1. 选项菜单Options menus :一个Activity只能有一个选项菜单，在按下Menu键时，显示在屏幕下方。
2. 上下文菜单Context menus :为Activity中的任何一个视图注册一个上下文菜单，“长按”出现。
3. 弹出式菜单Popup menus :依赖于Activity中的某个一个视图。

## ANR

Activity/UI线程----->5秒
Broadcast----->10秒
前台Service----->20秒

后台Service—>200秒

input事件—>5秒

### 三种常见类型

**1：** **KeyDispatchTimeout(5 seconds) --** **主要类型**

按键或触摸事件在特定时间内无响应

**2** **：** **BroadcastTimeout(10 seconds)**

BroadcastReceiver在特定时间内无法处理完成

**3：** **ServiceTimeout(20 seconds) --** **小概率类型**

Service在特定的时间内无法处理完成

## 如和防止ANR?

###  @+id/资源ID名，@id/资源ID名，@android:id/资源ID名三种id格式的区别

1. **android:id="@+id/new_name":**

   ​	开发者为当前的控件或者布局新定义一个id名称。该ID名称在R.java文件中会分配一个唯一的int型常量，用于对资源引用的索引。（如果R.java中存在这个值则直接使用存在的值，否则会在R.java文件中生成 new_name值)

2. **android:id="@id/defined_name" **
   引用一个名称为defined_name的id资源。该引用针对的是开发者自定义的id资源。（如果R.java中存在这个值，则使用此值）；否则会造成编译错误。

3. **android:id="@android:id/sys_name" **
   引用名称为sys_name的系统内部资源。

## ViewPager extends ViewGroup

## Android Handler

1. 避免了在新线程中刷新UI的操作
2. 采用队列的方式来存储Message
3. 实现不同线程间通信的一种机制

## 在 android 中使用 SQLiteOpenHelper 这个辅助类时，哪些操作可能生成一个数据库

​	getReadableDatabase(),getWritableDatabase(),创建或者打开数据库，如果数据库不存在则创建数据库，如果数据库存在直接打开数据库；默认情况下两个函数都表示打开或者创建可读可写的数据库对象，如果磁盘已满或者是数据库本身权限等情况下getReadableDatabase打开的是只读数据库

## Intent传递数据时，下列的数据类型哪些可以被传递

1. Serializable
2. CharSequence
3. Parcelable
4. Bundle

## Android的存储数据的方式

1. SharedPreferences
2. File
3. SQLite

## NotificationManager 中清除消息的方法是

1. cancel
2. cancelAll

## 进程状态

[参考链接](https://www.nowcoder.com/test/question/done?tid=39308857&qid=19988#summary)

**前台进程**

用户当前操作所必需的进程。如果一个进程满足以下任一条件，即视为前台进程：

1. 托管用户正在交互的 [Activity](https://developer.android.com/reference/android/app/Activity.html)（已调用 [Activity](https://developer.android.com/reference/android/app/Activity.html) 的 [onResume()](https://developer.android.com/reference/android/app/Activity.html#onResume()) 方法）
2. 托管某个 [Service](https://developer.android.com/reference/android/app/Service.html)，后者绑定到用户正在交互的 Activity
3. 托管正在“前台”运行的 [Service](https://developer.android.com/reference/android/app/Service.html)（服务已调用 [startForeground()](https://developer.android.com/reference/android/app/Service.html#startForeground(int, android.app.Notification))）
4. 托管正执行一个生命周期回调的 [Service](https://developer.android.com/reference/android/app/Service.html)（[onCreate()](https://developer.android.com/reference/android/app/Service.html#onCreate())、[onStart()](https://developer.android.com/reference/android/app/Service.html#onStart(android.content.Intent, int)) 或 [onDestroy()](https://developer.android.com/reference/android/app/Service.html#onDestroy())）
5. 托管正执行其 [onReceive()](https://developer.android.com/reference/android/content/BroadcastReceiver.html#onReceive(android.content.Context, android.content.Intent)) 方法的 [BroadcastReceiver](https://developer.android.com/reference/android/content/BroadcastReceiver.html)

通常，在任意给定时间前台进程都为数不多。只有在内存不足以支持它们同时继续运行这一万不得已的情况下，系统才会终止它们。 此时，设备往往已达到内存分页状态，因此需要终止一些前台进程来确保用户界面正常响应。

**可见进程**

没有任何前台组件、但仍会影响用户在屏幕上所见内容的进程。 如果一个进程满足以下任一条件，即视为可见进程：

- 托管不在前台、但仍对用户可见的 [Activity](https://developer.android.com/reference/android/app/Activity.html)（已调用其 [onPause()](https://developer.android.com/reference/android/app/Activity.html#onPause()) 方法）。例如，如果前台 Activity 启动了一个对话框，允许在其后显示上一 Activity，则有可能会发生这种情况。
- 托管绑定到可见（或前台）Activity 的 [Service](https://developer.android.com/reference/android/app/Service.html)。

可见进程被视为是极其重要的进程，除非为了维持所有前台进程同时运行而必须终止，否则系统不会终止这些进程。

**服务进程**

正在运行已使用 [startService()](https://developer.android.com/reference/android/content/Context.html#startService(android.content.Intent)) 方法启动的服务且不属于上述两个更高类别进程的进程。尽管服务进程与用户所见内容没有直接关联，但是它们通常在执行一些用户关心的操作（例如，在后台播放音乐或从网络下载数据）。因此，除非内存不足以维持所有前台进程和可见进程同时运行，否则系统会让服务进程保持运行状态。

**后台进程**

包含目前对用户不可见的 Activity 的进程（已调用 Activity 的 [onStop()](https://developer.android.com/reference/android/app/Activity.html#onStop()) 方法）。这些进程对用户体验没有直接影响，系统可能随时终止它们，以回收内存供前台进程、可见进程或服务进程使用。 通常会有很多后台进程在运行，因此它们会保存在 LRU （最近最少使用）列表中，以确保包含用户最近查看的 Activity 的进程最后一个被终止。如果某个 Activity 正确实现了生命周期方法，并保存了其当前状态，则终止其进程不会对用户体验产生明显影响，因为当用户导航回该 Activity 时，Activity 会恢复其所有可见状态。 有关保存和恢复状态的信息，请参阅 [Activity](https://developer.android.com/guide/components/activities.html#SavingActivityState)文档。

**空进程**

不含任何活动应用组件的进程。保留这种进程的的唯一目的是用作缓存，以缩短下次在其中运行组件所需的启动时间。 为使总体系统资源在进程缓存和底层内核缓存之间保持平衡，系统往往会终止这些进程。

## Activity被销毁时,调用什么方法保存截面数据?

实现 Activity 的 onSaveInstanceState（）方法,该方法不一定能保证得到使用,最好还是在onPause()中和onStop中做数据存储

### 调用什么方法恢复?

可以在onCreate()方法中判断savedInstanceState是否保存了数据onRestoreInstanceState()

## 在安卓应用程序开发中，可以通过(什么方法)获得屏幕的属性，从而取得屏幕的宽度和高度?

```java
getMetrics()
```

## 哪些情况下系统会程序抛出异常，强制退出(Force Close)

1. 应用运行时抛出了RuntimeException
2. Error
   1. 应用运行时抛出了OutOfMemoryError
   2. StackOverFlowError

## FC和ANR的区别

ANR是长时间未响应,还有响应的可能,不一定就强制退出了

## 在Android中使用IBinder进行IPC通讯时，能够传递下列哪些数据？

1. Parcleable
2. Serializable
3. Bundle

## 使用SimpleAdapter作为 ListView的适配器，行布局中支持下列哪些组件

1. 继承Checkable接口的组件(CompoundButton)
2. TextView
3. ImageView

## Android系统对下列哪些对象提供了资源池

1. Message

   1. .Message提供了消息池，有静态方法Obtain从消息池中取对象；
2. AsyncTask **线程池改造**,池里 默认提供（核数+1）个线程进行并发操作，最大支持（核数 * 2 + 1）个线程，超过后会丢弃其他任务
3. 线程池ThreadPool管理的Thread

## Broadcast Receiver程序在运行什么方法时，才会处于有效状态

```
onReceive
```

## ScrollView可以有几个直接的子控件

> 1. ScrollView只能添加一个子控件，如果添加了多个子控件，则会出现“ScrollView can host only one direct child”异常
> 2. 解决办法：子控件外面再套一层LinearLayout,注意只能是Linearlayout，布局中不能出现RelativeLayout

## SharedPreferences

1. SharedPreferences最终的存储形态是XML文件
2. SharedPreferences可以被多个应用共享访问
3. SharedPreferences可以被同一个应用的多个进程共享访问
4. sharedpreference.apply()是异步的，sharedpreference.commit()是同步的。
5. sharePreferences处理的就是key-value对
6. 属于移动存储解决方案

## 清单文件中设置不让他横竖屏切换,activity怎么知道进行了横竖屏切换

### 清单文件保证他不进行横竖屏切换

	>  **android:screenOrientation=”landscape”**

### 如何检测

> 可以使用重力传感器

[参考文章]([Activity横竖屏切换的那些事_gdutxiaoxu的博客（微信公众号 stormjun94）-CSDN博客](https://blog.csdn.net/gdutxiaoxu/article/details/62235974))

## 横竖屏切换一定会调用onDestory吗?

在清单文件中配置该属性：android:configChanges属性

总结：

1、不设置Activity的android:configChanges时，切屏会重新调用各个生命周期，切横屏时会执行一次，切竖屏时会执行两次

2、设置Activity的android:configChanges="orientation"时，切屏还是会重新调用各个生命周期，切横、竖屏时只会执行一次

3、设置Activity的android:configChanges="orientation|keyboardHidden"时，切屏不会重新调用各个生命周期，只会执行onConfigurationChanged方法

注意：自从Android 3.2（API 13），在设置Activity的android:configChanges="orientation|keyboardHidden"后，还是一样会重新调用各个生命周期的。因为screen size也开始跟着设备的横竖切换而改变。因此，阻止程序在运行时重新加载Activity，除了设置"orientation"，你还必须加上"ScreenSize"。