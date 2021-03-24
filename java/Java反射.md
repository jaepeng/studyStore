# Java反射

[Java反射技术详解_黄林晴-CSDN博客_java反射](https://blog.csdn.net/huangliniqng/article/details/88554510)

## 获取目标类

三种获取目标了类的方法

1. Class.forName

   ```java
    Class clazz=Class.forName("reflecttest.Apple");
   ```

2. 通过类获得

   ```java
   Class clazz= Apple.class;
   ```

3. 通过类对象获得

   ```java
   Apple apple1 = new Apple();
   Class clazz=apple1.getClass();
   ```

## 获取实例对象

1. 通过 Constructor 对象创建类对象可以选择特定构造方法,下面的代码就调用了一个有参数的构造方法进行了类对象的初始化。

   ```java
   Constructor constructor=clazz.getConstructor(String.class);
   Apple apple = (Apple) constructor.newInstance("大苹果");
   ```


以下两种方法都是默认调用无参构造函数

1. 通过Class对象的newInstance()方法

   ```java
    Apple apple = (Apple) clazz.newInstance();
   ```

3. 通过 Constructor 对象的 newInstance() 方法

 ```java
Constructor constructor=clazz.getConstructor();
Apple apple1 = (Apple) constructor.newInstance();
 ```

## 通过反射获取类属性、方法、构造器

1. 无参构造的对象

   ```java
   Method setName = clazz.getMethod("setName", String.class);
   Method getName = clazz.getMethod("getName");
   setName.invoke(apple,"小苹果");
   System.out.println("getname:"+getName.invoke(apple));
   ```

2. 有参的反射构造出来的实例对象

   ```java
   Constructor constructor= clazz.getConstructor(String.class);//参数类型
   Apple apple1 = (reflecttest.Apple) constructor.newInstance("大苹果");//参数
   System.out.println(apple1.getName());
   ```

3. 获得私有的方法

   ```java
   Method method = clazz.getDeclaredMethod("talk");//talk就是Apple中的私有方法
   method.setAccessible(true);//不设置就会报错
   method.invoke(apple1);//调用apple对象的talk方法
   //大苹果会说话
   ```

4. 获得、修改私有属性：

   ```java
   //私有属性:address name
   Field address = clazz.getDeclaredField("address");
   address.setAccessible(true);
   //设置属性
   address.set(apple1,"苹果树下");
   Field name = clazz.getDeclaredField("name");
   //允许修改私有属性,不设置就会报错
   name.setAccessible(true);
   
   //获得apple1对象的name属性值
   System.out.println(name.get(apple1).toString());
   
   //修改私有属性值
   name.set(apple1,"红苹果");
   //打印name
   Method getName = clazz.getMethod("getName");
   System.out.println(getName.invoke(apple1));
   ```

5. 公有属性的修改和设置操作步骤一样，不过不需要`name.setAccessible(true);`这一步

## 







