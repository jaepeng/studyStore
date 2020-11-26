# final、finally、finalize

[final、finally与finalize的区别 - 涛声依旧~ - 博客园 (cnblogs.com)](https://www.cnblogs.com/ktao/p/8586966.html)

## final

[final详情](https://github.com/jaepeng/studyStore/blob/master/java/final、static、final staic 之间的区别.md)

1. final: 修饰**变量(成员变量或局部变量)，方法，类**； 
   1. **修饰变量**时表明这对象的值不可变，你不能为这个变量赋一个新的值，或者这样说：对基本类型而言，你不能改变其数值，对于引用，你不能将其指向一个新的引用（而引用自身是可以改变的）。 
   2. **修饰方法**时表明我们希望把这个方法锁定，以防止任何继承类修改它的含义，这样会确保在继承中，我们的final方法的行为不会改变，并且不会被覆盖。使用final方法的另一个考虑是效率问题：在Java早期的时候，遇到final方法，编译器会将此方法调用转为内嵌调用，如此一来以减小方法调用产生的开销。
   3.  **修饰类**的时候表明你不打算继承该类，而且也不允许别人这样做。

## finally

1. finally:是异常处理中进行收场处理的代码块，比如关闭一个数据库连接，清理一些资源占用的问题。不管有没有异常被捕获，finally子句中的代码都会被执行。（[当然也会有意外情况](#finallyQuestion)）

2. 如果try中有return语句,那么先会执行finally中的语句,要**保证finally一定要被执行**

   1. ```java
      public static int sout(){
              int a=0;
              try {
                  a++;
                  System.out.println("try："+a);
                  return a;
              }finally {
                  a+=10;
                  System.out.println("finally:"+a);
      		//return a;
              }
          }
      //输出结果：
      try：1
      finally:11
      main:1
      ```

3. 如果try中有数据处理,比如(retrun a++),则会先**将(a++)计算出来**进行保存,供后面的语句进行执行,然后去执行finally中的语句,这时候如果finally中的语句有retrun 则直接返回了.

   1. ```java
       public static int sout(){
              int a=0;
              try {
                  a++;
                  System.out.println("try"+a);
                  return a;
              }finally {
                  a+=10;
                  System.out.println("finally:"+a);
                  return a;
              }
      }
      //输出结果
      try1
      finally:11
      main:11
      ```

4. 如果catch中也有返回值,finally中也有返回值,则finally中的返回值会替代catch中的语句,因为catch中的语句保存在一个临时区中，就算catch语句中有return，也不会执行，而是去执行finally中的return语句（如果finallly中没有return语句，则会执行catch中的语句）。

```java
public static int sout(){
        int a=0;
        try {
            System.out.println("try"+a);
            a/=0;
            return a;
        }catch (Exception e){
            System.out.println("catch:"+a);
            return -1;
        }finally {
            a+=10;
            System.out.println("finally:"+a);
//            return a;
        }
    }
//输出语句：
try0
catch:0
finally:10
main:-1
```

```java
public static int sout(){
        int a=0;
        try {
            System.out.println("try"+a);
            a/=0;
            return a;
        }catch (Exception e){
            System.out.println("catch:"+a);
            return -1;
        }finally {
            a+=10;
            System.out.println("finally:"+a);
            return a;
        }
    }
//输出语句
try0
catch:0
finally:10
main:10
```

<span id="finallyQuestion">**finally不执行的情况**</span>

1. 如果还没运行到try中就发生意外，程序终止的话，finally语句当然不会被执行，一定要在try执行后，finally语句才有机会执行
2. try中意外终止，catch中意外终止同理。

```java
 public static int sout(){
        int a=0;
        try {
            System.out.println("try:"+a);
            System.exit(0);
            a/=0;
            return a;
        }finally {
            a+=10;
            System.out.println("finally:"+a);
            return a;
        }
    }
//输出结果
try:0
```

1. finally 语句块没有执行，为什么呢？因为我们在 try 语句块中执行了 System.exit (0) 语句，终止了 Java 虚拟机的运行。
2. 当一个线程在执行 try 语句块或者 catch 语句块时被打断（interrupted）或者被终止（killed），与其相对应的 finally 语句块可能不会执行。
3. 还有更极端的情况，就是在线程运行 try 语句块或者 catch 语句块时，突然死机或者断电，finally 语句块肯定不会执行了。可能有人认为死机、断电这些理由有些强词夺理，没有关系，我们只是为了说明这个问题。



## finalize

1. finalize:finalize出现的原因在于： 我们一定需要进行清理动作。Java没有用于释放对象的，如同C++里的delete调用，的方法，而是使用垃圾回收器（GC）帮助我们释放空间。当垃圾回收器准备释放对象占用的存储空间的时候，将首先调用其finalize()方法。
2. finalize()是在java.lang.Object里定义的，也就是说每一个对象都有这么个方法。这个方法在gc启动，该对象被回收的时候被调用。其实gc可以回收大部分的对象（凡是new出来的对象，gc都能搞定，一般情况下我们又不会用new以外的方式去创建对象），所以一般是不需要程序员去实现finalize的。 
3. 特殊情况下，需要程序员实现finalize，当对象被回收的时候释放一些资源，比如：一个socket链接，在对象初始化时创建，整个生命周期内有效，那么就需要实现finalize，关闭这个链接。 
4. 使用finalize还需要注意一个事，调用super.finalize();
5. 一个对象的finalize()方法只会被调用一次，而且finalize()被调用不意味着gc会立即回收该对象，**所以有可能调用finalize()后，该对象又不需要被回收了，然后到了真正要被回收的时候，因为前面调用过一次，所以不会调用finalize()，产生问题**。 所以，推荐不要使用finalize()方法，它跟析构函数不一样。

