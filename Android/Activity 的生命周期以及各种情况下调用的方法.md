# Activity 的生命周期以及各种情况下调用的方法

[TOC]



### 生命周期介绍

1. **onCreate**:**表示页面（Activity）的创建**。（生命周期第一个阶段）功能：完成初始化工作，如：加载页面布局资源、初始化数据。
2. **onStart** : **表示页面（Activity）正在被启动**，即将开始。功能：页面为可见状态，但是无法与用户交互。
3. **onResume** : **表示页面（Activity）出现在前台**。功能：与 onStart 相比，onStart 处于后台，OnResume 才显示到前台。
4. **onPause** : **表示页面（Activity）正在停止**。功能：页面处于后台，正常情况下，onStop 紧接着执行。此时会做一些数据存储、停止动画不太耗时的工作。onPause 执行完新的页面（Activity）的 onResume 才会执行。
5. **onStop** : **表示页面（Activity）即将停止**。功能：页面为不可见状态，做稍微轻量级的不太耗时的回收工作。
6. **onDestroy** : **表示页面（Activity）即将销毁**。（生命周期最后一个阶段）功能：回收工作和资源的释放。
7. **onRestart** : **表示页面（Activity）重新启动**。功能：页面从不可见状态转化为可见状态时会调用此方法。如：Home 键切换页面（打开新的 Activity），然后回到页面过程中。

#### 还有两个保存页面数据和恢复页面数据的方法

1. onSaveInstanceState()保存页面数据
2. onRestoreInstanceState() 恢复页面数据

### 启动一个Activity经历的生命周期

> onCreate——>onStart——>onResume
>
> tips:这里没有进行数据保存的操作

#### 不同情况下的生命周期

1. 这个时候直接退出(按返回键退出)：

> onPause——>onStop——>onDestory

2. 这个时候翻转屏幕：

> onPause——>onStop——>onSaveInstanceState——>onDestory——>onCreate——>onStart——>onRestoreInstanceState——>onResume

​		由此可见，如果在一个Activity页面进行翻转页面的话，会将这个Activity**销毁**掉，然后**重新创建**。并且在onStop和onDestory方法之间会调用onSaveInstanceState()方法进行数据保存。在onStart和onResume方法之间还会将调用onRestoreInstanceState方法将数据恢复。

3. 翻转屏幕后如果是同侧的翻转（180°）是不会调用任何方法的

4. 翻转屏幕后翻转回来（90°）：

> onPause——>onStop——>onSaveInstanceState——>onDestory——>onCreate——>onStart——>onRestoreInstanceState——>onResume

   与上面的第一次翻转是一样的。

2.从Activity界面直接返回主界面

> onPause——>onStop——>onSaveInstanceState

###### 这里有个特殊情况 如果这个时候直接退出(应用被划掉),那走到这里就结束了,后面的onDestory是不会被执行的,和用back键退出不一样.

3.从主界面返回来

> onRestart——>onStart——>onResume

### 启动一个Activity 再启动第二个Activity

对第一个Activity的启动是一样的：

> onCreate——>onStart——>onResume

#### 启动第二个Activity

> Main.onPause——>Second.onCreate——>Second.onStart——>Second.onResume——>Main.onStop——>Main.onSavveInstanceState

##### 不同情况下的生命周期

1. 直接返回到第一个Activity

> Second.onPause——>Main.onRestart——>Main.onStart——>Main.onResume——>Second.onStop
>
> ——>Second.onDestory

2.再第二个Activity界面直接返回主界面

> 与上同

3.从主界面返回来

> 与上同

### 启动DialogActivity

为什么它**特殊**?因为它是一个弹出框类型的Activity,下面的Activity还是可见的.

> Main.onPasue——>Dialog.onCreate——>Dialog.onStart——>Dialog.onResume

#### 不同情况下生命周期

1. 返回第一个activity(点击屏幕空白区域,或back键)

> Dialog.onPause——>Main.onResume——>Dialog.onStop——>Dialog.onDestory

2.这个时候翻转屏幕

> DiaologActivity和普通Activity一样的翻转生命周期
>
> onPause——>onStop——>onSaveInstanceState——>onDestory——>onCreate——>onStart——>onRestoreInstanceState——>onResume

##### 这个的MainActivity的生命周期却大有不同

> 等DialogActivity的翻转生命周期执行完毕后,MainActivity才会开始重新来
>
> 由于MainActivity本身就处于onPause状态了,这个时候MainActivity就从onStop开始执行
>
> onStop——>onSaveInstanceState——>onDestory——>onCreate——>onStart——>onRestoreInstanceState——>onResume——>onPasue
>
> 很有意思的是,它最后又会回到**onPause状态.**

3. 这个时候直接回到主界面

> Dialog.onPause——>Main.onStop——>Main.onSaveInstanceState——>Dialog.onStop——>onSaveInstanceState

### 如何将数据返回到启动你的Activity

如果一个activity要返回数据到启动它的那个activity，可以调用**setResult()**方法，在activity**销毁(onDestory)**之前需要调用