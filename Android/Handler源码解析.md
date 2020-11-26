# Handler源码解析

[补充参考链接](https://github.com/Omooo/Android-Notes/blob/master/blogs/Android/Handler%20%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6.md)

![image-20201120102826616](https://gitee.com/pengjae/pic/raw/master/img/handler_1.png)

## Handelr 作用

1. 处理延时任务

> 设定将来某个时刻处理某件事情

2. 线程间通信

> 主线程和子线程通信

### Handelr 构成

![](https://gitee.com/pengjae/pic/raw/master/img/20201123092441.png)



### Handler的主要函数

![image-20201123140915195](https://gitee.com/pengjae/pic/raw/master/img/20201123140917.png)

由此可见，只要是与发送消息有关的函数，最终都是调用sendMessageAtTime()，而这个函数最后调用的则是MessageQueue的,enqueueMessage.
	
### sendMessage()——添加消息

1. 调用顺序

sendMessaged——>sendMessageDelayed(msg,0)——>sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);——>enqueueMessage(queue, msg, uptimeMillis);——>**queue**.enqueueMessage(msg, uptimeMillis);

​			1. 发送消息，调用的就是**MessageQueue.enqueueMessage()**方法，

2. ![image-20201123141528093](https://gitee.com/pengjae/pic/raw/master/img/20201123141530.png)

   ```java
    for (;;) {
       prev = p;
       p = p.next;
       if (p == null || when < p.when) {
               break;
       }
       if (needWake && p.isAsynchronous()) {
            needWake = false;
        }
   }
   msg.next = p; // invariant: p == prev.next
   prev.next = msg;
   ```

   ​     1. 使用for循环，用when（执行所需事件）做比较，找到需要插入的地方，进行插入

#### 那么谁来获取这个Message对象呢？MessageQueue.next()——获取Message对象

使用next方法，循环获取当前可以执行的Message(不为空），这个是否可执行还要看他们的when属性和now（当前时间）做对比。**[**if（now>=msg.when)**]**只有到了时间的才能将这个Message **return**出去，将其 删除（msg.next=null)

```java
MessageQueue.next()方法中：
    
nativePollOnce(ptr, nextPollTimeoutMillis);//由此进行队列的等待,是个native方法
//再寻找msg之前执行。确定是否需要等待
...
mBlocked=false/true;//根据是否还有消息进行判断

MessageQueue.enqueueMessage()方法中：
needWake=mBlocked;//由此判断是否还需要唤醒
唤醒的话会将上面的natviePollOnce唤醒。不用继续等待
Message中有一个target时Handler的实例。将消息入队时会赋值给msg

```

#### 	**那么在哪里去使用这个返回来的消息呢？**Looper.loop。开启这个不断取Message对象的传送带

**Looper.loop()**中，也有一个for循环，不断调用MessageQueue的**next**方法，去拿Message

注意：loop方法中没有returen和break语句，因为这是一个开关，打开后就是一直运行，不会关闭（最多等待一会）。所以，他这里不对消息进行return，而是交给` msg.target.dispatchMessage(msg);`将其分发出去执行。

#### 那么谁启动这个开关呢？在ActivityThread（线程）的main函数中 Looper.loop()

先调用`prepareMainLooper`方法，准备好一个Looper，再在main函数中调用`Looper.loop();`方法，启动传送带。而且每一个Android对象，有且只有这一个main函数，也就是程序的入口。

```java
public static void prepareMainLooper() {
        prepare(false);//最主要的准备方法
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
}
```

```java
 private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {//sThreadLocal是ThreadLocal的实例对象
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));//将Loop设置到了ThreadLocal中去，key是当前线程
}
```

#### 什么是ThreadLocal？

1.  是一个Map的弱引用。

2. 在这个Map中的**get**方法

   1. ```java
      public T get() {
              Thread t = Thread.currentThread();
              ThreadLocalMap map = getMap(t);//t(线程id)就是key
              if (map != null) {
                  ThreadLocalMap.Entry e = map.getEntry(this);//this指的就是当前线程
                  if (e != null) {
                      @SuppressWarnings("unchecked")
                      T result = (T)e.value;
                      return result;
                  }
              }
              return setInitialValue();
          }
      ```

   2. 先**获得当前线程**，再获得当前线程的ThreadLocalMap对象

   3. 使用map.getEntry(this)得到当前map的Entry对象，并通过它拿到值

3. 注意：Handler中的ThreadLocal里面存放的就是Loop对象(Looper中的源码定义)

   1. ```java
       static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
      ```

4. ThreadLocal的**set**方法

   1. ```java
      public void set(T value) {
              Thread t = Thread.currentThread();
              ThreadLocalMap map = getMap(t);
              if (map != null)
                  map.set(this, value);//将当前线程作为Key进行设置。
              else
                  createMap(t, value);
          }
      ```

5. 由于是用线程作为key的，所以达到了**线程隔离**的目的

### 调用Message总结

ActivityThread中调用Looper.prepareMainLooper准备一个loop，再在其中调用Looper.loop()获得当前线程的Looper对象，再通过这个对象获得当前的MessageQueue`final MessageQueue queue = me.mQueue;`。然后loop()里面还会不断的调用Message.next取出消息给loop，这个给时候loop就会调用dispatchMessage方法将其分发出去



### Loop

如果想要loop运行起来，要满足以下两个条件：

1. Looper.prepare()准备好一个Loop
2. Looper.loop();开启一个Loop

**提问：**

```java
final Looper me = myLooper();
mQueue = looper.mQueue;
```

1. 为什么Hanlder中使用Looper.myLooper获取Looer对象，而不是直接new一个呢？
2. 为什么Handler中使用looper.mQueue;获取MessageQueue对象，而不是直接new一个？
3. 如何保证hanlder.sendMessage(msg)和handler.handleMessage(msg)这个时候的handler是同一个呢？

>  答：因为线程是唯一的，那么当前的Looper是唯一的。（ThreadLocal保证），又因为内Looper唯一，所以MessageQueue唯一。由于MessageQueue是唯一的，所以handler发送的消息到这个queue中是唯一的MessageQueue，而在发送该消息到MessageQueue中时，会将当前（this）Handler赋值给`msg.target=this`

<img src="https://gitee.com/pengjae/pic/raw/master/img/20201123161407.png" alt="image-20201123161403880" style="zoom:70%;float:left" />



###  MessageQueue

> 典型的生产者消费者模式
>
> MessageQueue就是仓库，enqueueMessage()是生产者，next()是消费者
>
> 也就是因为是从子线程添加，主线程提取，也就实现了从子线程到主线程的通信（线程间通信）

<img src="https://gitee.com/pengjae/pic/raw/master/img/20201123161112.png" alt="image-20201123161044889" style="zoom:70%;float:left" />

1. **入队**时，如果队列满了，则会阻塞。直到调用next方法取出消息。当next方法被调用时，通知MessageQueue可以进行消息入队。
2. **出队**时，启动Looper.loop轮询器，对queue进行轮询。当消息到达执行时间就取出来。当messageQueue为空，则会阻塞，等待消息队列调用enqueueMessage的时候通知队列，可以取出消息，停止阻塞。

### enqueueMessage()

```java
public void enqueueMessage(Message msg){
    msg.target=this;//相当于给msg这个货物打上了标签，这样才能确定是哪个工人给我搬过来的
    messageQueue.enqueueMessage(msg);
}
```

## Handler问题

1. **Looper.loop 死循环不会造成应用卡死嘛？**

   如果按照 Message.next 方法的注释来解释的话，如果返回的 Message 为空，就说明消息队列已经退出了，这种情况下只能说明应用已经退出了。这也正符合我们开头所说的，Android 本身是消息驱动，所以没有消息几乎是不可能的事；如果按照源码分析，Message.next() 方法可能会阻塞是因为如果消息需要延迟处理（sendMessageDelayed等），那就需要阻塞等待时间到了才会把消息取出然后分发出去。然后这个 ANR 完全是两个概念，ANR 本质上是因为消息未得到及时处理而导致的。同时，从另外一方面来说，对于 CPU 来说，线程无非就是一段可执行的代码，执行完之后就结束了。而对于 Android 主线程来说，不可能运行一段时间之后就自己退出了，那就需要使用死循环，保证应用不会退出。这样一想，其实这样的安排还是很有道理的。

   [参考链接](https://www.zhihu.com/question/34652589/answer/90344494)

   [自我总结部分](https://github.com/jaepeng/studyStore/blob/master/Android/Android%E4%B8%AD%E8%BF%9B%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B%E7%9A%84%E5%8C%BA%E5%88%AB%EF%BC%88Looper.loop.md)



### 如何避免Handler内存泄漏

1. Handler 允许我们发送延时消息，如果在延时期间用户关闭了 Activity，那么该 Activity 泄漏。这是因为内部类默认持有外部类的引用。

   **解决办法**就是：将 Handler 定义为**静态内部类的形式，在内部持有 Activity 的弱引用，并及时移除所有消息。**

   ```java
   public class MainActivity extends AppCompatActivity {
   
       private MyHandler mMyHandler = new MyHandler(this);//虚引用的Activiy
   
       @Override
       protected void onCreate(Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           setContentView(R.layout.activity_main);
       }
   
       private void handleMessage(Message msg) {
   
       }
   
       static class MyHandler extends Handler {
           private WeakReference<Activity> mReference;
   
           MyHandler(Activity reference) {//内部持有Activiy的虚引用
               mReference = new WeakReference<>(reference);
           }
   
           @Override
           public void handleMessage(Message msg) {
               MainActivity activity = (MainActivity) mReference.get();
               if (activity != null) {
                   activity.handleMessage(msg);
               }
           }
       }
   
       @Override
       protected void onDestroy() {
           mMyHandler.removeCallbacksAndMessages(null);
           super.onDestroy();
       }
   }
   ```

   

## 整体总结

1. prepare一定要调用
2. 为什么使用ThreadLocal而不是HashMap
   1. HashMap太过臃肿（默认初始大小为8），而我们的ThreadLocal是线程为key的不需要那么大
   2. 线程是唯一的变量，更方便管理
3. 为什么使用ObtainMessage而不是直接创建的Message
   1. **享元设计模式**
   
      