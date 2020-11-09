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

## 关于res/raw目录说法正确的是？

1. 这里的文件是原封不动的存储到设备上，不会转换为二进制的格式
2. 这里的文件没有目录结构
3. 这里的文件都会在R.java中生成唯一ID

