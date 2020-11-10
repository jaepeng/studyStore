# 关于Android小知识

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



