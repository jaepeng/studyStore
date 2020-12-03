# Android启动模式

## Standard

+ 标准启动模式,不管栈内部是否有该Actitvity,都会重新实例化一个Activity起来
+ 新建的Activity都放在栈顶.
+ 从onCreate开始,一直调到onResume结束

## SingleTop

+ 栈顶复用模式: 如果栈顶部就是目标Activity的话,就会复用该Activity,否则就去新建一个
+ onCreate和onStart方法不会被再次调用
+ onNewIntent会被调用
+ 假设你在当前的Activity中又要启动同类型的Activity，此时建议将此类型Activity的启动模式指定为SingleTop

## SingleTask

+ 栈内复用模式:如果栈内有目标Activity的话,就将目标Activity上面的所有Activity全部出栈
+ 仅回调onNewIntent,像onCreate和onStart都不会调用
+ 最典型的样例就是应用中展示的主页（Home页）。

## SingleInstance

+ 全局单例模式:是一种**加强的SingleTask模式**。它除了具有它所有特性外，还加强了一点：具有此模式的Activity仅仅能单独位于一个任务栈中。
+ 这个经常使用于系统中的应用，比如Launch、锁屏键的应用等等，整个系统中仅仅有一个

## 启动方式

1.  在 Manifest.xml中指定Activity启动模式

   ```html
      <activity android:name="..activity.MultiportActivity" android:launchMode="singleTask"/>
   ```

   

2. 启动Activity时。在Intent中指定启动模式去创建Activity

   ```java
           Intent intent = new Intent();
           intent.setClass(context, MainActivity.class);
           intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
           context.startActivity(intent);
   ```

## onNewIntent

调用前提：ActivityA已经启动过,处于当前应用的Activity堆栈中; 



