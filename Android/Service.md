## Service

> **服务**是长期在后台运行的程序,首先它是一个组件，用于执行长期运行的任务，并且与用户没有交互。

服务属于服务进程,当杀死空进程和后台进程后内存还不足的话,就会杀死服务进程.但是当内存足够是,服务又会重新启动起来.

### Service的生命周期

#### **1.onCreate**

1. 如果service没被创建过，调用startService()后会执行onCreate()回调；
2. 如果service已处于运行中，调用startService()不会执行onCreate()方法。也就是说，onCreate()只会在第一次创建service时候调用，多次执行startService()不会重复调用onCreate()，此方法适合完成一些初始化工作。

#### **2.onStartCommand()**

 如果多次执行了Context的startService()方法，那么Service的onStartCommand()方法也会相应的多次调用。onStartCommand()方法很重要，我们在该方法中根据传入的Intent参数进行实际的操作，比如会在此处创建一个线程用于下载数据或播放音乐等。

#### 3.**onBind()**

Service中的onBind()方法是抽象方法，Service类本身就是抽象类，所以onBind()方法是必须重写的，即使我们用不到。

#### 4.**onDestory()**

在销毁的时候会执行Service该方法。

> 这几个方法都是回调方法，且在主线程中执行，由android操作系统在合适的时机调用。

1.startServices实例:

```java
 Intent intent = new Intent();
 intent.setClass(this, FirstService.class);
 startService(intent);//系统自带的,非自己定义的
```

2.stopService实例:

```java
Intent intent = new Intent();
intent.setClass(this, FirstService.class);
stopService(intent);
```

3. bindService实例:

   ```java
   private FirstService.MyBinder mRemoteBinder;
   public void bindServiceClick(View view){
           Intent intent = new Intent();
           intent.setClass(this, FirstService.class);
           mIsServiceBind = bindService(intent, mServiceConnection, BIND_AUTO_CREATE);
       }
       private ServiceConnection mServiceConnection=new ServiceConnection() {
   
   
   
           @Override
           public void onServiceConnected(ComponentName name, IBinder service) {
               Log.d(TAG, "onServiceConnected: ");
               mRemoteBinder = (FirstService.MyBinder) service;
           }
   
           @Override
           public void onServiceDisconnected(ComponentName name) {
               mRemoteBinder=null;//解绑
           }
       };
   ```

   

4.unbindService实例:

```java
public void unbindServiceClick(View view){
        if (mServiceConnection != null&&mIsServiceBind) {
            unbindService(mServiceConnection);//系统自带方法
        }

}
```

#### 绑定服务流程:

1.在Service中创建需要在服务中使用的方法.

```java
public void  sayHello(){
        Toast.makeText(this, "Hello world", Toast.LENGTH_SHORT).show();
}
```

2.因为需要再onBind()方法中进行绑定,所以需要声明一个Mybinder(继承自Binder)

```java
public IBinder onBind(Intent intent) {
        Log.d(TAG, "onBind: ");
        return new MyBinder();
    }
```

3.再这个MyBinder中调用需要在服务中使用的方法

```java
public class MyBinder extends Binder{
        public void callServiceInnerMethod(){//声明方法给外部实现去调用
            Log.d(TAG, "callServiceInnerMethod: ");
            sayHello();
        }

    }
```

4.在需要使用serverice中的方法的时候调用

```java
mRemoteBinder.callServiceInnerMethod();
```

### StartService和BindService

#### startService:

1. 开启服务后与Activity无关,就算应用程序退出了,Service还仍然会继续运行在后台
2. 不可以跟服务进行通讯

#### bindService:

1. 绑定服务后就与Activity绑定,应用程序退出,Service也会退出
2. 可以跟服务进行通讯

#### 混合绑定的优势

> 我们先startService，再进行bindService，这样子的话，服务可以长期于后台运行，又可以跟服务进行通讯了。

标准声明周期

onCreate(创建服务)——>onStartCommand(开启服务,使用startService()方法开启)——>onBind(绑定服务)——>onServiceConnected: (Service去建立连接,调用内部方法)——>onUnbind(解绑服务，否则**无法停止服务**的（但是服务，仍然在后台运行）)——>onDestory(彻底不需要时停止服务,使用stopService()方法)







