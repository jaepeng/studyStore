# Android IPC机制

## 简介:

IPC:Inter-Process Communication(跨进程通信)

### 进程和线程的区别:

1. 线程是CPU调度的最小单元,同时线程是一种有限的系统资源
2. 进程一般只一个执行单元,在PC和移动设备上只一个程序或者一个应用。一个进程可以包含多个线程，因此进程和线程是**包含和被包含**的关系。

## 多进程模式

### 开启多进程模式

1. 给四大组件在AndroidMenifest中指定`android:process`属性
2. 非常规方法：通过JNI在native层去fork一个新的进程

#### 指定process

```java
<activity 
  android:name ="com.test.SecondActivity"
...
  android：process=":remote"
/>
<activity 
  android:name ="com.test.ThirdActivity"
...
  android：process=":remote"
/>




```

