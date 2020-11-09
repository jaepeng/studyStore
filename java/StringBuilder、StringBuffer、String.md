## StringBuilder、StringBuffer、String

### String

> [参考链接](https://www.nowcoder.com/profile/891455099/test/39167738/3731#referAnchor)

#### 一些概念

1. 字符串String不是一个基本类型但是有时又和基本类型差不多,可以对变量直接赋值,而不用new一个对象(也可以new).String类是**不可变类**

```java
String str="abc"
```

2. Java中的变量和基本类型的值存放于**栈内存**,new出来的**对象**本身存放于**堆**内存,而指向对象的**引用**还是存放在**栈内存**.

   1. ```java
      int i=1;
      ```

   2. ```java
      String s=new String("hello");
      String s2="world"
      ```

   3. 这里的i,s,s2,1,”world”都存放在栈中,而“hello”存放在堆中,并且s指向该字符串

3. 栈内存的一个特点是数据共享，这样设计是为了减小内存消耗

   1. 比如说再定义一个j=1 ,将j放入栈内存后会开始寻找栈中是否有1,如果有j就指向1,如果没有则向栈内存中存放一个1.

      此时这个j和i指向都就是同一个对象.

      ```java
      int j=1;
      i==j;//true
      ```

   2. **也就是如果常量在栈内存中，就将变量指向该常量，如果没有就在该栈内存增加一个该常量，并将变量指向该常量。**

   3. 下面这个情况,这时指向的变量并不会改变，而是在栈内寻找新的常量（比原来的常量大1），如果栈内存有则指向它，如果没有就在栈内存中加入此常量并将j指向它。

      ```java
      j++;
      ```

   4. 如果都是new的呢?则这两个“hello world”都存在于堆内存中.并且是不一样的对象

      ```java
      String a=new String("hello world");
      String b=new String("hello world");
      a==b;//false;
      ```

   5. 如果字符串相加,在**编译时**直接将字符串**合并**，而不是等到运行时再合并。所以是指向栈内存中的数据.

      ```java
      String s="taobao";
      public static final String MESSAGE="taobao";
      String a =  "tao" + "bao";
      System.out.println(a==s);//true
      System.out.println(a==MESSAGE);//true
      ```

   6. 那对于字符串变量相加呢?只能等到**运行时**才能判定是什么字符串，编译器不会优化.害怕你会对值进行改变,所以一定要等到运行时才能确定,防止你后来运行时改变了它所指向的字符串。所以其实s对象是放再堆内存中的。

      ```java
      public static final String MESSAGE="taobao";
      String a="tao";//栈内存
      String b="bao";//栈内存
      String s=a+b;//放在堆内存中
      System.out.println(MESSAGE==s);//false
      ```

      ==tips:==String的“**+**”号**拼接字符串的操作**是靠**StringBuffer**实现的.

      > 先构造一个StringBuffer里面存放”tao”,然后调用append()方法追加”bao”，然后将值为”taobao”的StringBuffer转化成String对象。StringBuffer对象在堆内存中，那转换成的String对象理所应当的也是在堆内存中。

   7. intern() 方法会先检查 String 池 ( 或者说成栈内存 ) 中是否存在相同的字符串常量，如果有就返回

      ```java
      System.out.println( (b+c).intern()== MESSAGE );//true
      ```

   8. 如果将a,b都定义为final呢?因为是final变量,变量不可以再赋值了(已经确定了),所以编译器在编译时就将a,b字符合并起来,放入栈内存中

      ```java
      final String  a="tao";
      final String b="bao";
      String c=a+b;
      String s="taobao";
      System.out.println(c==s);//true
      ```

      ==tips:==要是有一个不是final,就不会这么优化,还是会返回false

   ### StringBuffer&StringBuilder

   #### StringBuffer 和 StringBuilder 类的对象能够被多次的修改，并且**不产生新的未使用对象**。

   1. StringBuilder 的方法不是线程安全的,StringBuffer线程安全.

   2. 正因为StringBuffer线程安全,所以效率较低,StringBuilder有速度优势.

   3. 初始化时,默认容量为16+value.length=19.如果只是实例化一个空的stringBuffer/stringBuilder对象出来,那么默认容量为16.

      ```java
      StringBuffer sb=new StringBuffer("123");
       System.out.println(stringBuffer.capacity());//19
      
      ```

   4. 扩容方式2*n+2;当超过Integer.MAX_VALUE时,就只能扩容到MAX_VALUE这个值,并且会抛出异常OutOfMemoryError

   



