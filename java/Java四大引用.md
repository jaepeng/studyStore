# Java四大引用

> 在 Java 中一切都被视为了对象，但是我们操作的标识符实际上是对象的一个引用（reference）。

**tips**：以下**代码运行情况**出现在将 JVM 的**初始内存设为2M**，最大**可用内存为 3M**。

## 强引用(Strong)

1. Java中默认声明的就是强引用:

```java
Object obj = new Object(); //只要obj还指向Object对象，Object对象就不会被回收
obj = null;  //手动置null
```

2. 只要强引用存在，垃圾回收器将**永远不会回收被引用的对象**，哪怕内存不足时，JVM也会直接抛出OutOfMemoryError，不会去回收。如果想中断强引用与对象之间的联系，**可以显示的将强引用赋值为null，这样一来，JVM就可以适时的回收对象了。**

```java
//创建一个2M大小的数组时，系统会报OOM错误，但仍然不会被回收
byte[] buff = new byte[1024 * 1024 * 2];
```



## 软引用(Soft)

1. 软引用是用来描述一些非必需但仍有用的对象。在内存足够的时候，软引用对象不会被回收，只有在**内存不足时，系统则会回收软引用对象**，**如果回收了软引用对象之后仍然没有足够的内存，才会抛出内存溢出异常**。这种特性常常被用来实现**缓存技术**，比如网页缓存，图片缓存等。

```java
//最后一个创建成功（2M就存不下了），其他都打印为null
private static void testSoftReference() {
		for (int i = 0; i < 10; i++) {
			byte[] buff = new byte[1024 * 1024];//虽然这里是强引用，但是sr仍然是指向一个弱引用
            //Java的编译器发现了在之后的代码中, buff 已经没有被使用了, 所以自动进行了优化。buff变成了弱引用对象
			SoftReference<byte[]> sr = new SoftReference<>(buff);
			list.add(sr);
		}
		
		System.gc(); //主动通知垃圾回收
		
		for(int i=0; i < list.size(); i++){
			Object obj = ((SoftReference) list.get(i)).get();
			System.out.println(obj);
		}
		
}
```

```java
//但是对上面代码进行优化后，情况就大不相同
private static void testSoftReference() {
    byte[] buff;
    for(){
        buff= new byte[1024 * 1024];
    }
    ...
    
}
//由于buff这个强引用一直存在，所以会报OOM错误，而不会进行回收
```

2. 如果一个对象唯一剩下的引用是软引用，那么该对象是软可及的（softly reachable）。垃圾收集器并不像其收集弱可及的对象一样，尽量地收集软可及的对象，相反，它只在**真正 “需要” 内存时才收集软可及的对象**。

## 弱引用(Weak)

1. 弱引用的引用强度比软引用要更弱一些，**无论内存是否足够，只要 JVM 开始进行垃圾回收，那些被弱引用关联的对象都会被回收**。

```java
private static void testWeakReference() {
		for (int i = 0; i < 10; i++) {
			byte[] buff = new byte[1024 * 1024];
			WeakReference<byte[]> sr = new WeakReference<>(buff);
			list.add(sr);
		}
		
		System.gc(); //主动通知垃圾回收
		
		for(int i=0; i < list.size(); i++){
			Object obj = ((WeakReference) list.get(i)).get();
			System.out.println(obj);
		}
}
//打印结果都为null
```

2. 如果一个对象仅仅是偶尔使用，并且希望在使用时随时就能获取到，但又不想影响此对象的垃圾收集，那么你应该用 WeakReference 来引用该对象。

3. 弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果弱引用**所引用的对象被垃圾回收**，Java虚拟机就会把这个**弱引用**加入到与之关联的**引用队列**中。

4. 一般来说，很少直接使用WeakReference，而是使用WeakHashMap。在WeakHashMap中，内部有一个引用队列，插入的元素会被包裹成WeakReference，并加入队列中，用来做缓存再合适不过。

   1. 如果要判断哪些软引用对象被回收

      ```java
      SoftReference ref=null;
      while((ref=(SoftReference)queue.poll()!=null)){
          //清除软引用对象
      }
      ```

      

## 虚引用(Phantom)

1. 虚引用是**最弱的一种引用关系**，如果一个对象仅持有虚引用，那么它就和没有任何引用一样，它随时可能会被回收，在 JDK1.2 之后，用 PhantomReference 类来表示，通过查看这个类的源码，发现**它只有一个构造函数和一个 get() 方法**，而且它的 get() 方法仅仅是返回一个null，也就是说将**永远无法通过虚引用来获取对象**，虚引用必须要**和 ReferenceQueue 引用队列一起使用**。

```java
public class PhantomReference<T> extends Reference<T> {
    /**
     * Returns this reference object's referent.  Because the referent of a
     * phantom reference is always inaccessible, this method always returns
     * <code>null</code>.
     *
     * @return  <code>null</code>
     */
    public T get() {
        return null;//永远是null,但是weakReference没有该方法
    }
    public PhantomReference(T referent, ReferenceQueue<? super T> q) {
        super(referent, q);
    }
}
```

### 引用队列（ReferenceQueue）

```java
Object obj=new Object();
ReferenceQueue queue=new ReferenceQueue();
SoftReference softReference=new SoftReference(obj,queue);
```



1. 引用队列可以与**软引用、弱引用以及虚引用一起配合使用**，当垃圾回收器准备回收一个对象时，如果发现它还有引用，那么就会在回收对象之前，把这个引用加入到与之关联的引用队列中去。程序可以通过判断引用队列中是否已经加入了引用，来判断被引用的对象是否将要被垃圾回收，这样就可以在对象被回收之前采取一些必要的措施。
2. 如果一个和引用队列配合使用的引用,如果它指向的对象被回收,则该引用会被追加到队列尾部.
3. 与软引用、弱引用不同，**虚引用必须和引用队列一起使用。**
4. 它的本质是一个链表

## 弱引用与软引用对比



### 区别

1. **只具有弱引用**的对象拥有**更短暂的生命周期。**
2. 被垃圾回收器回收的时机不一样，在垃圾回收器线程扫描它所管辖的内存区域的过程中**，一旦发现了只具有弱引用的对象，不管当前内存空间足够与否，都会回收它的内存。**而被软引用关联的对象只有在内存不足时才会被回收。
3. 弱引用不会影响GC，而软引用会一定程度上对GC造成影响。

### 相似之处

1. 都是用来描述非必需对象的。

### 那么什么时候用SoftReference，什么时候用WeakReference呢？

1. 如果缓存的对象是比较**大**的对象，使用频率相对较高的对象，那么使用**SoftReference**会更好，因为这样能让缓存对象有更长的生命周期。

2. 如果缓存对象都是比较**小**的对象，使用频率一般或者相对较低，那么使用**WeakReference**会更合适。

3. 当然，如果实在不知道选哪个，一般而言，用作缓存时使用WeakHashMap都不会有太大问题

- 弱引用是比软引用更弱的引用类型
- **弱引用不能延长对象的生命周期，一旦对象只剩下弱引用，它就随时可能会被回收**
- **可以通过弱引用获取对象的强引用**
- 弱引用适合用作缓存

