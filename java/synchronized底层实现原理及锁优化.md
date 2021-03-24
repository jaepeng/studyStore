# synchronized底层实现原理及锁优化

[synchronized底层实现原理及锁优化 - 简书 (jianshu.com)](https://www.jianshu.com/p/c8f997e7f75c)

## 一、简述

[自旋锁、适应性自旋锁、锁消除、锁粗化、偏向锁、轻量级锁_公众号@代码艺术-CSDN博客](https://blog.csdn.net/weixin_46288569/article/details/106582151)

1️⃣synchronized的作用
 ①原子性：synchronized 保证语句块内操作是原子的。
 ②可见性：synchronized 保证可见性(通过“在执行unlock之前，必须先把此变量同步回主内存”实现)。
 ③有序性：synchronized 保证有序性(通过“一个变量在同一时刻只允许一条线程对其进行lock操作”)。

## 二、实现原理

[JVM](https://www.jianshu.com/p/b3728c26f29b) 的同步(synchronized)基于**进入和退出管程(Monitor)对象实现**(管程结构确保每次只有一个进程在管程内处于活动状态)， 无论是显式同步(有明确的 monitorenter 和 monitorexit 指令，即同步代码块)还是隐式同步都是如此。

1️⃣方法级的同步
 在 Java 语言中，同步用的最多的地方可能是被 synchronized 修饰的同步方法。**方法级的同步是隐式，即无需通过字节码指令来控制的**，它实现在方法调用和返回操作之中。JVM 可以从方法常量池中的方法表结构(method_info Structure)中的 ACC_SYNCHRONIZED 访问标志区分一个方法是否同步方法。当方法调用时，调用指令将会检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先持有 monitor (虚拟机规范中用的是管程一词)，然后再执行方法，最后在方法完成(无论是正常完成还是非正常完成)时释放 monitor。

2️⃣代码块的同步
 代码块的同步是利用 monitorenter 和 monitorexit 这两个字节码指令。[synchronized](https://www.jianshu.com/p/1d936f93570a) 是 [JVM](https://www.jianshu.com/p/b3728c26f29b) 实现的一种互斥同步访问方式，**底层是基于每个对象的监视器(monitor)来实现的**。被 synchronized 修饰的代码，被编译器编译后在前后加上了一组字节指令。
 ①在代码开始加入了 monitorenter，在代码后面加入了 monitorexit，这两个字节码指令配合完成了 synchronized 关键字修饰代码的互斥访问。
 ②在虚拟机执行到 monitorenter 指令的时候，会请求获取对象的 monitor 锁，基于 monitor 锁又衍生出一个锁计数器的概念。
 ③当执行 monitorenter 时，若对象未被锁定时，或者当前线程已经拥有了此对象的 monitor 锁，则锁计数器+1，该线程获取该对象锁。
 ④当执行 monitorexit 时，锁计数器-1，当计数器为0时，此对象锁就被释放了。那么其它阻塞的线程则可以请求获取该 monitor 锁。
 ⑤如果获取monitor对象失败，该线程则会进入阻塞状态，直到其他线程释放锁。

这里要注意：
①synchronized 是[可重入的](https://www.jianshu.com/p/5212b58373bb)，所以不会自己把自己锁死
②synchronized 锁一旦被一个线程持有，其他试图获取该锁的线程将被阻塞。

# ReentrantLock可重入锁的使用

> 何为可重入锁？当一个线程获得了当前实例的锁，并进入方法a，这个线程在没有释放这把锁的时候，可以再次进入方法a。

## 一、简介

### 和synchronized对比

| synchronized                                               | ReentrantLock                                                |
| ---------------------------------------------------------- | ------------------------------------------------------------ |
| 是排他锁，加锁和解锁的过程自动进行，易于操作，但不够灵活。 | 也是排他锁锁，加锁和解锁的过程需要手动进行，不易操作，但非常灵活。 |
| 可重入，因为加锁和解锁自动进行，不必担心最后是否释放锁。   | 也可重入，但加锁和解锁需要手动进行，且次数需一样，否则其他线程无法获得锁。 |
| 不可响应中断，一个线程获取不到锁就一直等着。               | 可以响应中断。                                               |

**最重要的点:**

synchronized所没有的，一个最主要的就是 ReentrantLock 可以实现**公平锁机制**。什么叫公平锁呢？**也就是在锁上等待时间最长的线程将获得锁的使用权。通俗的理解就是谁排队时间最长谁先执行获取锁。**

## 使用

1️⃣简单使用：一个最基础的使用案例，也就是实现锁的功能。



```csharp
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockDemo {
    private static final Lock lock = new ReentrantLock();//定义一个可重入锁
    public static void main(String[] args) {
        new Thread(() -> test(), "线程A").start();
        new Thread(() -> test(), "线程B").start();
    }
    public static void test() {
        for (int i = 0; i < 2; i++) {
            lock.lock();//锁上
            System.out.println(Thread.currentThread().getName() + "获取了锁");
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();//解锁
            }
        }
    }
}
```

这里定义了一个 ReentrantLock，然后在 test 方法中分别 lock 和 unlock，运行一遍就可以实现功能。这就是最简单的功能实现，代码很简单。

2️⃣公平锁实现
 对于公平锁的实现，要结合可重入性质。

```csharp
public class ReentrantLockDemo {
    private static final Lock lock = new ReentrantLock(true);
    public static void main(String[] args) {
        new Thread(() -> test(), "线程A").start();
        new Thread(() -> test(), "线程B").start();
        new Thread(() -> test(), "线程C").start();
        new Thread(() -> test(), "线程D").start();
        new Thread(() -> test(), "线程E").start();
    }
    public static void test() {
        for (int i = 0; i < 2; i++) {
            lock.lock();
            System.out.println(Thread.currentThread().getName() + "获取了锁");
            try {
                TimeUnit.SECONDS.sleep(2);
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }
    }
}
```

new 一个 ReentrantLock 的时候参数为 **true**，表明实现公平锁机制。在这里多定义几个线程ABCDE，然后**在 test 方法中循环执行了两次加锁和解锁的过程：**

![img](https://gitee.com/pengjae/pic/raw/master/img/20201203200840.webp)

3️⃣非公平锁实现



非公平锁就是随机的获取，谁运气好，cpu时间片轮到哪个线程，那个线程就能获取锁。和公平锁的区别很简单，**就在于先 new 一个 ReentrantLock 的时候参数为 false，当然也可以不写，默认就是 false。**测试结果：

![img](https://gitee.com/pengjae/pic/raw/master/img/20201203200922.webp)

4️⃣响应中断
**响应中断就是一个线程获取不到锁，不会傻傻的一直等下去，ReentrantLock 会给予一个中断回应。**在这里举一个死锁的案例：

```java
public class ReentrantLockDemo {
    private static final Lock lockA = new ReentrantLock();
    private static final Lock lockB = new ReentrantLock();

    public static void main(String[] args) {
        Thread threadA = new Thread(new ThreadDemo(lockA, lockB));
        Thread threadB = new Thread(new ThreadDemo(lockA, lockB));
        threadA.start();
        threadB.start();
        threadA.interrupt();//第一个线程中断，这样B线程就会继续执行完。
    }

    static class ThreadDemo implements Runnable {
        Lock firstLock;
        Lock secondLock;
        public ThreadDemo(Lock firstLock, Lock secondLock) {
            this.firstLock = firstLock;
            this.secondLock = secondLock;
        }
        @Override
        public void run() {
            try {
                firstLock.lockInterruptibly();
                secondLock.lockInterruptibly();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                firstLock.unlock();
                secondLock.unlock();
                System.out.println(Thread.currentThread().getName() + "获取到了资源，正常结束！");
            }
        }
    }
}
```

![image-20201203211942449](https://gitee.com/pengjae/pic/raw/master/img/20201203211942.png)

# Synchronized和Lock的区别

[Synchronized和lock区别 - 简书 (jianshu.com)](https://www.jianshu.com/p/1d936f93570a)

| 类别     | synchronized                                                 | Lock                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 存在层次 | Java的关键字，在JVM层面上                                    | 是一个类                                                     |
| 锁的释放 | 1. 以获取锁的线程执行完同步代码，释放锁<br />2.线程执行发生异常，Jvm会让线程释放锁 | 在finally中必须释放锁，不然容易造成线程死锁                  |
| 锁的获取 | 假设A线程获得锁，B线程等待。如果A线程阻塞，B线程会一直等待   | 分情况而定，Lock有多个锁获取的方式。可以尝试获得锁，线程可以不用一直等待 |
| 锁状态   | 无法判断                                                     | 可以判断                                                     |
| 锁类型   | 可重入<br />不可中断<br />非公平                             | 可重入<br />可中断<br />可公平（两者皆可）                   |
| 性能     | 少量同步                                                     | 大量同步                                                     |

## 用法区别

1️⃣synchronized：在需要同步的对象中加入此控制，synchronized 可以加在方法上，也可以加在特定代码块中，括号中表示需要锁的对象。

2️⃣lock：**需要显示指定起始位置和终止位置**。一般使用 ReentrantLock 类做为锁，多个线程中必须要使用一个 ReentrantLock 类做为对象才能保证锁的生效。且在加锁和解锁处需要通过 lock() 和 unlock() 显示指出。所以要在 finally块中写 unlock() 以防死锁。

## 性能区别

synchronized 是托管给 [JVM](https://www.jianshu.com/p/b3728c26f29b) 执行的，而 lock 是 Java 写的控制锁的代码。在 Java1.5 中，synchronized 是性能低效的。因为这是一个重量级操作，需要调用操作接口，导致有可能加锁消耗的系统时间比加锁以外的操作还多。相比之下使用 Java 提供的 Lock 对象，性能更高一些。但是到了Java1.6，发生了变化。synchronized 在语义上很清晰，可以进行很多优化，有适应自旋，锁消除，锁粗化，轻量级锁，偏向锁等等。导致在 Java1.6 上 synchronized 的性能并不比 Lock 差。官方也表示，他们也更支持 synchronized，在未来的版本中还有优化余地。

说到这里，还是想提一下这两种机制的具体区别。synchronized 原始采用的是CPU[悲观锁机制](https://www.jianshu.com/p/d2ac26ca6525)，即线程获得的是[排他锁](https://www.jianshu.com/p/b4731a7d255a)。排他锁意味着其他线程只能依靠阻塞来等待线程释放锁。**而在 [CPU](https://www.jianshu.com/p/45c6bcb85934) 转换线程阻塞时会引起线程上下文切换，当有很多线程竞争锁的时候，会引起 CPU 频繁的上下文切换导致效率很低。**

**而 Lock 用的是[乐观锁机制](https://www.jianshu.com/p/d2ac26ca6525)。**所谓乐观锁就是，每次不加锁而是假设没有冲突而去完成某项操作，如果因为冲突失败就重试，直到成功为止。乐观锁实现的机制就是 [CAS 操作(Compare and Swap)](https://www.jianshu.com/p/98220486426a)。进一步研究 [ReentrantLock](https://www.jianshu.com/p/5212b58373bb) 的源码，会发现其中比较重要的获得锁的一个方法是 compareAndSetState。这里其实就是调用的 CPU 提供的特殊指令。

现代的 CPU 提供了指令，可以自动更新共享数据，而且能够检测到其他线程的干扰，而 compareAndSet() 就用这些代替了锁定。这个算法称作非阻塞算法，意思是一个线程的失败或者挂起不应该影响其他线程的失败或挂起的算法。

##  用途区别

1️⃣某个线程在等待一个锁的控制权的这段时间需要中断。
 2️⃣需要分开处理一些 wait-notify，ReentrantLock 里面的 Condition 应用，能够控制 notify 哪个线程。
 3️⃣具有公平锁功能，每个到来的线程都将排队等候。

先说第一种情况，[ReentrantLock](https://www.jianshu.com/p/5212b58373bb)  的 lock 机制有2种，忽略中断锁和响应中断锁，这带来了很大的灵活性。比如：如果A、B 两个线程去竞争锁，A线程得到了锁，B线程等待，但是A线程这个时候实在有太多事情要处理，就是一直不返回，B线程可能就会等不及了，想中断自己，不再等待这个锁了，转而处理其他事情。这个时候 ReentrantLock 就提供了两种机制：可中断/可不中断：
 ①B线程中断自己(或者别的线程中断它)，但是 ReentrantLock 不去响应，继续让B线程等待，你再怎么中断，我全当耳边风(synchronized 原语就是如此)；
 ②B线程中断自己(或者别的线程中断它)，ReentrantLock 处理了这个中断，并且不再等待这个锁的到来，完全放弃。

## 总结

1️⃣来源：lock 是一个接口，而 synchronized 是 Java 的一个关键字，synchronized 是内置的语言实现。

2️⃣异常是否释放锁：synchronized 在发生异常时候会自动释放占有的锁，因此不会出现死锁；而 lock 发生异常时候，不会主动释放占有的锁，必须手动 unlock 来释放锁，可能引起死锁的发生。(所以最好将同步代码块用 [try catch](https://www.jianshu.com/p/384d2ec90c41) 包起来，finally 中写入 unlock，避免死锁的发生。)

3️⃣是否响应中断
 lock 等待锁过程中可以用 interrupt 来中断等待，而 synchronized 只能等待锁的释放，不能响应中断。

4️⃣是否知道获取锁：Lock 可以通过 **trylock** 来知道有没有获取锁，而 synchronized 不能。

5️⃣Lock 可以提高多个线程进行读操作的效率。(可以通过 [ReadWriteLock ](https://www.jianshu.com/p/377ac92ac10c) 实现读写分离)

6️⃣在性能上来说，如果竞争资源不激烈，两者的性能是差不多的，而当竞争资源非常激烈时(即有大量线程同时竞争)，此时 Lock 的性能要远远优于 synchronized。所以说，在具体使用时要根据适当情况选择。

7️⃣synchronized 使用 [Object](https://www.jianshu.com/p/aeb8430fe799) 对象本身的 wait 、notify、notifyAll 调度机制，而 Lock 可以使用 Condition 进行线程之间的调度。

# volatile和synchronized的区别

[volatile和synchronized的区别_suchahaerkang的博客-CSDN博客_volatile和synchronized](https://blog.csdn.net/suchahaerkang/article/details/80456085)

