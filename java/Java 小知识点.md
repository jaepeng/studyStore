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

```java
public class Parent {

    public Parent() {
        System.out.println("父类构造器的方法");
    }
    static{
        System.out.println("父类静态代码块的方法");
    }
    {
        System.out.println("父类的代码块的方法");
    }

}
```

**tips:**如果父类的非静态代码块去掉括号是不能执行的，编译错误。

## 枚举（enum）类型

1. 是Java 5新增的特性，它是一种新的类型，允许用常量来表示特定的数据片断，而且全部都以类型安全的形式来表示，是特殊的类，可以拥有成员变量和方法。
2. 是一种类，自然就不是原始数据类型啦

## 局部内部类

> 就像是局部方法里面的变量，是不能用 public protected、private、default等来修饰的，**static**也不行

## TreeSet的方法

**sub(from,true,to,true):**true表示要包括边界值(即from,to)



## java的复制数组的效率

>  复制的效率System.arraycopy>clone>Arrays.copyOf>for循环

## ceil、floor、round

**ceil：**天花板，找比自己大的整数

floor：地板，找比自己小的整数

round：正数四舍五入，负数，五舍六入。

## Collection和Collections

**java.util.Collection** 是一个**集合接口**。它提供了对集合对象进行基本操作的通用接口方法。Collection接口在Java 类库中有很多具体的实现。Collection接口的意义是为各种具体的集合提供了最大化的统一操作方式。

**java.util.Collections** 是一个**包装类**。它包含有各种有关集合操作的静态多态方法。此类不能实例化，就像一个工具类，服务于Java的Collection框架。

## ThreadLocal

1. **用于创建线程的本地变量**，该变量是线程之间不共享的
2. ThreadLocal存放的值是线程封闭，**线程间互斥**的，主要用于**线程内共享**一些数据，**避免通过参数来传递**
3. 线程的角度看，每个线程都保持一个对其线程局部变量副本的隐式引用，只要线程是活动的并且 ThreadLocal 实例是可访问的；在线程消失之后，其线程局部实例的所有副本都会被垃圾回收
4. 在Thread中有一个成员变量ThreadLocals，该变量的类型是ThreadLocalMap(是ThreadLoacl的内部类),也就是一个Map(但不是继承和implementMap接口)，它的键是threadLocal，值为就是变量的副本。
5. 对于多线程资源共享的问题，同步机制采用了“以时间换空间”的方式，而ThreadLocal采用了“以空间换时间”的方式
6. 通过ThreadLocal的get()方法可以获取该线程变量的本地副本，在get方法之前要先set,否则就要重写initialValue()方法。

## ThreadLoaclMap

1. 这个map使用[开放地址法](https://cloud.tencent.com/developer/article/1361248)解决冲突

## 进程和线程

1. 一个**进程**是一个**独立的运行环境**，可以被看做一个程序或者一个应用。而**线程**是在进程中**执行的一个任务**。Java运行环境是一个包含了不同的类和程序的**单一进程**。线程可以被称为**轻量级进程**。线程需要较少的资源来创建和驻留在进程中，并且可以共享进程中的资源

   

## 包(package)的作用

1. 为了更好地组织类，Java提供了包机制。包是类的容器，用于分隔类名空间。如果没有指定包名，所有的示例都属于一个默认的无名包。Java中的包一般均包含相关的类，java是跨平台的，所以java中的包和操作系统没有任何关系，java的包是**用来组织文件的一种虚拟文件系统**。
2. import语句并没有将对应的java源文件拷贝到此处**仅仅是引入**，告诉编译器有使用外部文件，编译的时候要去读取这个外部文件。
3. Java提供的包机制与IDE没有关系。
4. 定义在同一个包（package）内的类可以不经过import而直接相互使用。

## java集合框架(Collection)

![img](https://raw.githubusercontent.com/jaepeng/PicGo/master/img/java%E9%9B%86%E5%90%88%E6%A1%86%E6%9E%B6%E5%9B%BE?token=AGMTLFNSZJYT4CXXP25CZ2S7VMXY4)

## 引用传递和值传递

### 引用传递(call by reference)

1. 用传递**不可以**改变原变量的**地址**，但可以改变原变量的内容-,**原因**是当副本的引用改变时，原变量 的引用并没有发生变化，当副本改变内容时，由于副本引用指向的是原变量的地址空间，所以，原变量的内容发生变化。
2. 对象(数组自然也是)

### 值传递(call by value)

1. 值传递**不可以**改变原变量的**内容和地址**,**原因**是java方法的形参传递都是传递**原变量的副本**，在方法中改变的是副本的值，而不适合原变量的
2. 基本类型

### String

1. 虽然String看似是一个基本类型,实际上它是一个final的char类型数组,所以传递String字符的时候其实是对副本的改变,因为String本身是final的,所以是不能再次赋值的。
2. 如果是String数组的话，由于是一个对象，所以是可以改变的。

