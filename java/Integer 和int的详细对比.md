# Integer 和int的详细对比

> 可以类比其他包装类型和基本类型

[参考链接](https://blog.csdn.net/i6223671/article/details/88873163?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.edu_weight&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.edu_weight)

**注意:**Intger的范围在（-128~127）是非常需要注意的临界点

##  1.int与Ingteger的基本使用对比

1. Integer是int的包装类；int是基本数据类型；
2. Integer变量必须实例化后才能使用；int变量不需要；
3. Integer实际是对象的引用，指向此new的Integer对象**(存放在堆中）**；int是直接存储数据值 ；
4. Integer的**默认值**是null；int的默认值是0。

## 2.详细对比

1. 两个**new出来**的Integer对象是永远不会相等的，因为都是两个对象（存放在堆中）

2. **Integer对象和int比较**，只要两个变量的值是向等的，则结果为true（因为包装类Integer和基本数据类型int比较时，**java会自动拆包装为int，然后进行比较**，实际上就**变为两个int变量的比较**）

3. 非new的Integer和new的Integer对象相比，绝对不同。因为非new生成的Integer变量指向的是java**常量池**中的对象，而new Integer()生成的变量指向**堆**中新建的对象，两者在内存中的地址不同）

4. 对于两个非new生成的Integer对象，进行比较时，如果两个变量的值在区间**-128到127**之间，则比较结果为true，如果两个变量的值不在此区间，则比较结果为false

   ​	**4解释：**java在编译Integer i = 100 ;时，会翻译成为Integer i = Integer.valueOf(100)。而java API中对Integer类型的valueOf的定义如下，对于-128到127之间的数，会进行缓存，Integer i = 127时，会将127进行缓存，下次再写Integer j = 127时，就会直接从缓存中取，就不会new了。

## int 的存储位置与声明位置有关

【★★★★★】Java中的**变量和基本类型的值**(局部变量)存放于**栈内存**,如果是**类变量**或者是**成员变量**则是在**堆**中(详细请参考【Integer 和int的详细对比】)。new出来的**对象实例**本身存放于**堆**内存,而指向对象的**引用**还是存放在**栈内存**.

```java
public class Test {
    public static int a = 1;// 基本类型 类变量 存放在堆中
    public int b = 2;// 基本类型 成员变量,存放在堆中  对象都放在堆里面，b和a这两个类的数值自然也放在堆中
    public void method() {//这是一个方法，该方法放在栈帧中，当方法结束，出栈后，它所带的局部变量也就消失了。
        Test test=new Test();//这个test放在栈中，而new Test()放在堆里面
        int c = 3;// //基本类型 局部变量 存放在栈中，当方法出栈时，该变量消亡
        int d = 3;// 局部变量
        c = 4;
    }
}

```

