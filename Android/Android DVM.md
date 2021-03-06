# Android DVM

## 基础知识

>  DVM指dalivk的虚拟机。每一个Android应用程序都在它自己的进程中运行，都拥有一个独立的Dalvik虚拟机实例。而每一个DVM都是在Linux 中的一个进程，所以说可以认为是同一个概念

1. Dalvik是Google公司自己设计用于Android平台的Java虚拟机,每一个Dalvik应用作为一个**独立的Linux 进程**执行。**独立的进程可以防止在虚拟机崩溃的时候所有程序都被关闭**.

2. **Dalvik** **专门针对同时高效地运行多个虚拟机进行优化，因此** **Android** **系统以方便的实现对应用程序进行隔离。**
3. Linux的进程是程序的具体实现，是执行程序的过程

## Dalvik和Java运行环境的区别：

1. Dalvik主要是完成**对象生命周期管理**，堆栈管理，线程管理，安全和异常管理，以及垃圾回收等等重要功能。
2. Dalvik负责进程隔离和线程管理，每一个Android应用在**底层**都会**对应一个独立的Dalvik虚拟机实例**，其代码在虚拟机的解释下得以执行。
3. 不同于Java虚拟机运行java字节码，Dalvik虚拟机运行的是其专有的文件格式Dex 　　	4:dex文件格式可以减少整体文件尺寸，提高I/o操作的类查找速度。
4. odex是为了在运行过程中进一步提高性能，对dex文件的进一步优化。
5. 所有的Android应用的线程都对应一个Linux线程，虚拟机因而可以更多的依赖操作系统的线程调度和管理机制
6. 有一个特殊的虚拟机进程Zygote，他是**虚拟机实例的孵化器**。它在系统启动的时候就会产生，它会**完成虚拟机的初始化，库的加载，预制类库和初始化的操作**。如果系统需要一个新的虚拟机实例，它**会迅速复制自身，以最快的数据提供给系统**。对于一些只读的系统库，所有虚拟机实例都和Zygote共享一块内存区域。