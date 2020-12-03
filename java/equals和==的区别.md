# equals和==的区别

## hashCode:

1. 在 Java 应用程序执行期间，在同一对象上多次调用 hashCode 方法时，必须一致地返回相同的整数，前提是对象上 equals 比较中所用的信息没有被修改。
2. 从某一应用程序的一次执行到同一应用程序的另一次执行，该整数无需保持一致。
3. 如果根据 equals(Object) 方法，两个对象是相等的，那么在两个对象中的每个对象上调用 hashCode 方法都必须生成相同的整数结果。
4. 以下情况不是必需的：如果根据 equals(java.lang.Object) 方法，两个对象不相等，那么在两个对象中的任一对象上调用 hashCode 方法必定会生成不同的整数结果。但是，程序员应该知道，为不相等的对象生成不同整数结果可以提高哈希表的性能。
5. 实际上，由 Object 类定义的 hashCode 方法确实会针对不同的对象返回不同的整数。（这一般是通过将该对象的内部地址转换成一个整数来实现的，但是 Java编程语言不需要这种实现技巧。）  
6. 当equals方法被重写时，通常有必要重写 hashCode 方法，以维护 hashCode 方法的常规协定，该协定声明相等对象必须具有相等的哈希码。

如果两个对象的hashCode相等,则equalse的方法会执行,**不一定**返回true。

​	**解释：**如果两个hashCode相等，那最多说明他们发生了冲突，不能说明他们就是同一个对象。但是如果两个对象执行equals方法后，返回true，则两个对象必定相等，这是规定。因为如果返回equals方法相等，则他们调用hashcode方法生成的整数对象，一定要是相等的。	

## equals和==

### equals：

+ 对于基本类型。八大类：byte，short，int，long，float，double，char，boolean；
  + ta们之间的比较，应用双等号（==）,比较的是ta们的**值**。 
  + Integer和int的比较无论是`new Integer(10).equals(10)还是new Integer(10)==10`返回都为true
+ 复合数据类型(类) 
  + 当ta们用（==）进行比较的时候，比较的是ta们在内存中的存放地址，
     所以，除非是同一个new出来的对象，ta们的比较后的结果为true，否则比较后结果为false。
+ **如果不重写equals方法的话**，由于每个类继承自Object的equals方法。他们的底层都是用`==`来比较的。

### 一些特性：

+ 自反性：对于任何非空引用值 x，x.equals(x) 都应返回 true。 
+ 对称性：对于任何非空引用值 x 和 y，当且仅当 y.equals(x) 返回 true 时，x.equals(y) 才应返回 true。 
+ 传递性：对于任何非空引用值 x、y 和 z，如果 x.equals(y) 返回 true，并且 y.equals(z) 返回 true，那么 x.equals(z) 应返回 true。 
+ 一致性：对于任何非空引用值 x 和 y，多次调用 x.equals(y) 始终返回 true 或始终返回 false，前提是对象上 equals 比较中所用的信息没有被修改。 
+ 对于任何非空引用值 x，x.equals(null) 都应返回 false。 
+ **重写equals方法时请必须重写hashcode，以保证equals方法相等时两个对象hashcode返回相同的值。**