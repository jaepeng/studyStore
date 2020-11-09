## StringBuilder、StringBuffer、String

### String

> [参考链接](https://www.nowcoder.com/profile/891455099/test/39167738/3731#referAnchor) 该参考链接有误，String常量放在常量池中而不是栈中。且栈是私有的，不共享的。

#### 一些概念

1. 字符串String不是一个基本类型但是有时又和基本类型差不多,可以对变量直接赋值,而不用new一个对象(也可以new).String类是**不可变类**

```java
String str="abc"
```

2. 一些简单的定义（详情请见【Integer和int的详细对比】

   1. ```java
      int i=1;//i如果是局部变量的话，i和1都存放在栈中。
      ```

   2. ```java
      String s=new String("hello");//“hello”存放在堆中，s在栈中指向“hello”
      String s2="world"//s2在栈中，“world”在常量池中
      ```

   3. 这里的i,s,s2,1,都存放在栈中,而“hello”,存放在堆中,并且s,指向自己的字符串。而”world”放在常量池中，s2指向常量池中的该对象。

   

3. 如果都是new的呢?则这两个“hello world”都存在于堆内存中.并且是不一样的对象

   ```java
   //都存在于堆内存中，但是不一样的两个对象
   String a=new String("hello world");
   String b=new String("hello world");
   a==b;//false;
   ```

   1. 如果**字符串相加**（两个可以确定的字符串，而不是变量）,在**编译时**直接将字符串**合并**，而不是等到运行时再合并。所以是指向**常量池**中的数据.

      ```java
      String s="taobao";//放在常量池中
      public static final String MESSAGE="taobao";//放在常量池中
      String a =  "tao" + "bao";//放在常量池中
      System.out.println(a==s);//true
      System.out.println(a==MESSAGE);//true
      ```

   2. 那对于**字符串变量相加**呢?只能等到**运行时**才能判定是什么字符串，编译器不会优化.害怕你会对值进行改变,所以一定要等到运行时才能确定,防止你后来运行时改变了它所指向的字符串。所以其实s对象是放再堆内存中的。

      ```java
      public static final String MESSAGE="taobao";
      String a="tao";//栈内存
      String b="bao";//栈内存
      String s=a+b;//放在堆内存中相当于new String(a+b),和MESSAGE(常量池中)不是同一个对象
      System.out.println(MESSAGE==s);//false
      ```

      ==tips:==String的“**+**”号**拼接字符串的操作**是靠**StringBuffer**实现的.

      > 先构造一个StringBuffer里面存放”tao”,然后调用append()方法追加”bao”，然后将值为”taobao”的StringBuffer转化成String对象。StringBuffer对象在堆内存中，那转换成的String对象理所应当的也是在堆内存中。

   3. intern() 方法会先检查 String 池 ( 或者说成栈内存 ) 中是否存在相同的字符串常量，如果有就返回

      ```java
      System.out.println( (b+c).intern()== MESSAGE );//true
      ```

   4. 如果将a,b都定义为final呢?因为是final变量,变量不可以再赋值了(已经确定了),所以编译器在编译时就将a,b字符合并起来,放入栈内存中

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

   



