# Java 小知识点

[TOC]

## Runnable和Callable的相同与区别

[参考文章](https://blog.csdn.net/const_/article/details/88105721)

### 相同

1. 都是接口
2. 都用start()方法启动
3. 都是编写多线程程序

### 不同

1. Runnable接口没有返回值,Callable接口有(是个泛型 V),和Future、FutureTask配合可以用来获取异步执行的结果
2. Callable的call方法可以抛出异常,Runnable的run方法只能内部消化
3. Callalble接口支持返回执行结果，需要调用FutureTask.get()得到，此方法会阻塞主进程的继续往下执行，如果不调用不会阻塞。

### Runnable 和Callable区别

| 方法名   | ExecutorService的执行方法 | ExecutorService.submit()返回值 | 返回的Future调用get()方法 | 线程体 |        取消执行         |
| -------- | ------------------------- | ------------------------------ | ------------------------- | ------ | :---------------------: |
| Runnable | execute和submit           | Future                         | null                      | run()  |          不能           |
| Callable | 只能是submit              | Future                         | Future定义的泛型T         | call() | Future.cancel能取消执行 |

### ExcutorService中的excutor和submit方法的区别

两者都是将一个线程任务添加到线程池中并执行；
1、excutor**没有返回值**，submit**有返回值**，并且返回执行结果**Future对象**
2、excutor不能提交Callable任务，只能提交Runnable任务，submit两者任务都可以提交
3、在submit中提交Runnable任务，会返回执行结果Future对象，**但是Future调用get方法将返回null**（Runnable没有返回值）

### 使用场景：

使用多线程任务校验被保人数据；
1、每个被保人都是一个线程任务，
2、每个线程任务执行完都需要告诉主线程执行成功还是失败
3、这里需要submit提交Callable任务返回Future对象，并通过Future.get方法来获取执行结果

### FutureTask

FutureTask实现了RunnableFuture，RunnableFuture既实现了Runnbale又实现了Futrue这两个接口；它两者的结合体
FutureTask又包装了Callable（如果是Runnable最终会转化为Callable）；
FutureTask可以通过Thread包装来直接执行，也可以提交给ExcutorService来执行，并可以直接通过get方法来获取执行结果；

### ExecutorCompletionService

使用ExecutorCompletionService提交任务后会将执行结果放到阻塞队列中，使用take方法会得到结果，哪个任务先执行完成就先获取到这个任务的执行结果;
原理：在ExecutorCompletionService维护了一个**QueueingFuture（队列任务）**，当通过ExecutorCompletionService提交的任务执行完成后，将结果放入QueueingFuture中；然后通过take和poll方法获取执行结果时会阻塞线程，直到当QueueingFuture中有结果时就会立即返回；
如果**自己维护一个list**来存放future执行结果，会导致的**问题**是：这样通过future.get方法来获取执行结果只能一个一个阻塞取出执行结果，如果后面的任务可能会先执行完成，后面的任务只有等待前面的任务执行完成得到结果后才能获取到结果，这样就浪费一定的时间在等待执行任务的结果上了。可以用ExecutorCompletionService来解决这个问题的。
总结：
1、自己创建一个集合来保存Future存根并循环调用其返回结果的时候，主线程并不能保证首先获得的是最先完成任务的线程返回值。它只是按加入线程池的顺序返回。因为take方法是阻塞方法，后面的任务完成了，前面的任务却没有完成，主程序就那样等待在那儿，只到前面的完成了，它才知道原来后面的也完成了。
2、使用CompletionService来维护处理线程的返回结果时，**主线程总是能够拿到最先完成的任务的返回值**，而不管它们加入线程池的顺序。
3、CompletionService的实现是维护了一个保存Future的BlockingQueque。只有当这个Future的任务状态是结束的时候，才会加入到这个Queque中，take()方法其实就是Producer-Consumer中的Consumer。它会从Queue中取出Future对象，如果Queue是空的，就会阻塞在那里，直到有完成的Future对象加入到Queue中。也就是先完成的必定先被取出，这样就减少了不必要的等待时间。
**注意**：

1. 使用Future获取多个任务的执行结果时，如果其中**一个**任务**出现异常**，**就会直接中断不会获取后面任务的执行结果了**，但是不会中断其他任务，**其他任务会正常运行，只是结果无法获取了**；
2. 如果需要**中断其他任务**就需要在捕获到异常后执行ExecutorService.shutdownNow方法来关闭线程，也可以通过Future.cancle方法来中断

#### ExecutorService中shutdownNow和shutdown的区别：

**shutdownNow**表示**立刻关闭**线程池并中断正在执行的任务；
**shutdown**表示线程池中的任务**全部执行完成**后再关闭线程池





### Callable接口优势:

多线程返回执行结果是很有用的一个特性，因为多线程相比单线程更难、更复杂的一个重要原因就是因为多线程充满着未知性，某条线程是否执行了？某条线程执行了多久？某条线程执行的时候我们期望的数据是否已经赋值完毕？无法得知，我们能做的只是等待这条多线程的任务执行完毕而已。而**Callable+Future/FutureTask却可以获取多线程运行的结果**，可以在等待时间太长没获取到需要的数据的情况下**取消**该线程的任务，真的是非常有用。

### example1:

```java
public class CallableAndFuture {
    public static void main(String[] args) {
        Callable<Integer> callable = new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                Thread.sleep(6000);
                return new Random().nextInt();
            }
        };
        FutureTask<Integer> future = new FutureTask<>(callable);
        new Thread(future).start();
        try {
            Thread.sleep(1000);
            System.out.println("hello begin");
            System.out.println(future.isDone());
            System.out.println(future.get());
            System.out.println(future.isDone());
            System.out.println("hello end");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}

//result:
有返回值的线程 0
有返回值的线程 2
有返回值的线程 4
有返回值的线程 6
有返回值的线程 8
子线程的返回值10
```

### eample2:

```java
public class CallableAndFuture {
    public static void main(String[] args) {
        Callable<Integer> callable = new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                Thread.sleep(6000);
                return new Random().nextInt();
            }
        };
        FutureTask<Integer> future = new FutureTask<>(callable);
        new Thread(future).start();
        try {
            Thread.sleep(1000);
            System.out.println("hello begin");
            System.out.println(future.isDone());
//            future.cancel(false);
            if (!future.isCancelled()) {
                System.out.println(future.get());
                System.out.println(future.isDone());
                System.out.println("hello end");
            } else {
                System.out.println("cancel~");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}	
```

## java多态实现机制

[java多态实现机制——参考文章](https://www.cnblogs.com/kaleidoscope/p/9790766.html)

[更好的参考](https://blog.csdn.net/github_37130188/article/details/89931885)

[更更更好的参考](https://www.jianshu.com/p/68ddb5484ca2)

### 多态的实现原理

1. Java 里对象方法的调用是依靠类信息里的方法表实现的。
2. 总体而言，当调用对象某个方法时，JVM查找该对象类的方法表以确定该方法的直接引用地址，有了地址后才真正调用该方法。

子类继承父类的方法，如果不**重写**该方法，那么调用时会指向父类的方法。如果**重写**该方法，那么指向该类的**代码区**是子类存有的**父类**的方法区

调用对象某个方法时，JVM查找该对象类的方法表以确定该方法的直接引用地址，有了地址后才真正调用该方法

### 多态又分为编译时多态和运行时多态。

**编译时多态:**比如重载
**运行时多态：**比如重写

### 多态原理：

#### 简单版：

原理也很简单，父类或者接口定义的引用变量可以指向子类或者具体实现类的实例对象，由于程序调用方法是在**运行期才动态绑定**的，那么引用变量所指向的**具体实例对象在运行期才确定**。所以这个对象的方法是运行期正在内存运行的这个对象的方法而不是引用变量的类型中定义的方法。

#### 学术版

我们将引入Java静态分派和动态分派这个概念。

- 静态分派:所有依赖静态类型来定位方法执行版本的分派动作。静态分派发生在编译阶段，因此确定静态分派的动作实际上不是由虚拟机来执行的，而是由编译器来完成。（编译时多态）
- 动态分派：在运行期根据实际类型确定方法执行版本的分派动作。（运行时多态）

## Java访问权限

> public>protected>default>private

**protected**:

1. 同一个包下的类
2. 子类
3. 同一个类

**default**：

1. 同类
2. 同包

## 子类实现父类后的代码执行顺序

>父类静态代码块 ->子类静态代码块 ->父类非静态代码块 -> 父类构造函数 -> 子类非静态代码块 -> 子类构造函数。

## 枚举（enum）类型

1. 是Java 5新增的特性，它是一种新的类型，允许用常量来表示特定的数据片断，而且全部都以类型安全的形式来表示，是特殊的类，可以拥有成员变量和方法。
2. 是一种类，自然就不是原始数据类型啦

## 局部内部类

> 就像是局部方法里面的变量，是不能用 public protected、private、default等来修饰的，**static**也不行