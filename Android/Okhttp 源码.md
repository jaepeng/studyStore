# Okhttp 源码

多看:[OKhttp源码解析详解系列 - 简书 (jianshu.com)](https://www.jianshu.com/p/d98be38a6d3f)

## 第一条主线

> 请求发送到哪去了

### enqueue操作:

+ 最后进入到dispatch(okhttp)中,有这么一端代码.可以看到`runningAsyncCalls`这个运行队列将`call`添加进去是有条件的,当前队列数要小于maxRequests(64),`runningCallsForHost(call)`对同一个目标的连接要小于`maxrequestsPerHost`(5),才能将它添加进去.

+ 否则就将它添加到等待队列.

![image-20201130075646235](https://gitee.com/pengjae/pic/raw/master/img/20201130075702.png)



## 第二条主线

> 请求被谁处理了

+ 从上面可以看出来,一旦把请求放入到运行队列中,就会直接放到线程池(executorService())中进行执行.

![image-20201130080314056](https://gitee.com/pengjae/pic/raw/master/img/20201130080314.png)

`new SynchronousQueue:`

+ 线程池内部维护一个队列,可以将线程池来不及处理的线程放到这个队列中进行等待.但是由于现在创建的队列是一个无容量队列,所以不能存放,如果有请求进来,就一定要去创建线程.所以最大可使用线程数为`Integer.MAX_VALUE`

+ 为什么要创建一个无容量队列呢?而不是直接创建一个队列,然后设置核心线程数10,最大线程数100.?

  + 如果按照这个方案,那么一个请求发过来,最大有可能需要排三次队(发送时的等待队列,然后再进入运行队列,然后再运行队列的线程池中再排队)
  + 本来就已经用maxRequest(64个)来进行约束了,如果进了运行队列还需要等待,那还不如直接放到等待队列当中去
  + 并且,线程池内部的队列对我们来说是不可控的,所以最好让它没用

+ 执行的时候(execute)，再execute方法中会调用下面这个方法(处理请求)

  ```java
final class RealCall implements Call {
      Response getResponseWithInterceptorChain() throws IOException {
        // Build a full stack of interceptors.
          List<Interceptor> interceptors = new ArrayList<>();
          interceptors.addAll(client.interceptors());
          interceptors.add(retryAndFollowUpInterceptor);
          interceptors.add(new BridgeInterceptor(client.cookieJar()));
          interceptors.add(new CacheInterceptor(client.internalCache()));
          interceptors.add(new ConnectInterceptor(client));
          if (!forWebSocket) {
              interceptors.addAll(client.networkInterceptors());
          }
          interceptors.add(new CallServerInterceptor(forWebSocket));
  
          Interceptor.Chain chain = new RealInterceptorChain(interceptors, null, null, null, 0,
                  originalRequest, this, eventListener, client.connectTimeoutMillis(),
                  client.readTimeoutMillis(), client.writeTimeoutMillis());
  
          return chain.proceed(originalRequest);
      }
  }
  ```
  
  
  
  + 首先一次将用户添加的拦截器client.interceptors()--retryAndFollowUpInterceptor（重定向）--BridgeInterceptor--CacheInterceptor--ConnectInterceptor--networkInterceptors（非forWebSocket）--CallServerInterceptor加入到列表中。
  + 接着构造一个拦截器链index=0的链元素RealInterceptorChain
  + 最终调用第一个链元素的proceed
  
  责任链模式：一个请求是一条链子．再链子上有多个节点，去判断它是否符合条件，符合就往下走，不符合就直接返回
  
  ![img](https://gitee.com/pengjae/pic/raw/master/img/20201130084621.webp)
  
  1. 在当前链元素的proceed（）中会构造下一个链元素next，并获取到当前链元素的拦截器interceptor，调用拦截器的interceptor.intercept(next);将next传进去。
  
  2. 在intercept(next)继续会调用next的proceed（）直到最后一个CallServerInterceptor。在该拦截其中没有调用next的proceed（）
  3. 最后将Response逐级返回。
  4. 它的实现采用责任链模式

## 第三条主线

> 请求是怎么被维护的

　再线程请求完之后，会调用`client.dispatcher.finished(this)`方法

这个方法就去根据运行队列和等待队列中的数据进行处理.比如说,当运行队列将请求已经发给服务器处理完了,那么就会回头去检查运行队列和等待队列里面的数据,如果等待队列有数据,则将他放到运行队列中去,

## OkHttp系统

![img](https://gitee.com/pengjae/pic/raw/master/img/20201130084533.webp)

## 请求与封装

### 请求的封装

> 请求是由Okhttp发出，真正的请求都被封装了在了接口Call的实现类RealCall中

#### 2.1 Call接口展示

```java
public interface Call extends Cloneable {
    
  //返回当前请求
  Request request();

  //同步请求方法，此方法会阻塞当前线程知道请求结果放回
  Response execute() throws IOException;

  //异步请求方法，此方法会将请求添加到队列中，然后等待请求返回
  void enqueue(Callback responseCallback);

  //取消请求
  void cancel();

  //请求是否在执行，当execute()或者enqueue(Callback responseCallback)执行后该方法返回true
  boolean isExecuted();

  //请求是否被取消
  boolean isCanceled();

  //创建一个新的一模一样的请求
  Call clone();

  interface Factory {
    Call newCall(Request request);
  }
}
```

#### 2.2 RealCall的构造方法

```java
final class RealCall implements Call {
    
  private RealCall(OkHttpClient client, Request originalRequest, boolean forWebSocket) {
    //我们构建的OkHttpClient，用来传递参数
    this.client = client;
    this.originalRequest = originalRequest;
    //是不是WebSocket请求，WebSocket是用来建立长连接的，后面我们会说。
    this.forWebSocket = forWebSocket;
    //构建RetryAndFollowUpInterceptor拦截器(重试和重定向拦截器)
    this.retryAndFollowUpInterceptor = new RetryAndFollowUpInterceptor(client, forWebSocket);
  }
}
```

### 请求等发送

+ 同步请求

```java
@Override
public Response execute() throws IOException {
    synchronized (this) {
        if (executed) throw new IllegalStateException("Already Executed");
        executed = true;
    }
    captureCallStackTrace();
    eventListener.callStart(this);
    try {
        client.dispatcher().executed(this);
        Response result = getResponseWithInterceptorChain();
        if (result == null) throw new IOException("Canceled");
        return result;
    } catch (IOException e) {
        eventListener.callFailed(this, e);
        throw e;
    } finally {
        client.dispatcher().finished(this);
    }
}
```



+ 异步请求

```java
@Override
public void enqueue(Callback responseCallback) {
    synchronized (this) {
        if (executed) throw new IllegalStateException("Already Executed");
        executed = true;
    }
    captureCallStackTrace();
    eventListener.callStart(this);
    client.dispatcher().enqueue(new AsyncCall(responseCallback));
}
```

+ 相同点
  1. 不管是同步请求还是异步请求都是Dispatcher在处理：
  2. 在执行请求之前都会回调eventListener.callStart(this);
  3. 通过getResponseWithInterceptorChain()获取Response
+ 不同点
  1. 同步请求：直接执行，并返回请求结果
  2. 异步请求：构造一个AsyncCall，并将自己加入处理队列中
  3. 异步请求多了responseCallback

#### Dispatcher是一个任务调度器，它内部维护了三个双端队列

- readyAsyncCalls：准备运行的异步请求
- runningAsyncCalls：正在运行的异步请求
- runningSyncCalls：正在运行的同步请求

+ 同步请求就直接把请求添加到正在运行的同步请求队列runningSyncCalls中，

+ 异步请求会做个判断：
  +  如果正在运行的异步请求不超过64，而且同一个host下的异步请求不得超过5个则将请求添加到正在运行的同步请求队列中runningAsyncCalls并开始 执行请求，否则就添加到readyAsyncCalls继续等待。

## StreamAllocation

连接的创建是在**StreamAllocation**对象统筹下完成的，我们前面也说过它早在**RetryAndFollowUpInterceptor**就被创建了，StreamAllocation对象，而且也创建了Socket了。但是连接还是在**ConnectInterceptor连接拦截器**中建立的。
 主要用来管理两个关键角色：

- **RealConnection**：真正建立连接的对象，利用**Socket建立连接**。
- **ConnectionPool**：连接池，用来管理和复用连接。