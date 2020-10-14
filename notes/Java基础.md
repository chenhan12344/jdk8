# java.lang.Object

## 1.常用方法

> hashCode();
> equals();
> toString();
> clone();
> wait();
> notify();
> notifyAll();
> getClass();

## 2. Object.equals()方法和Object.hashCode()方法

在没有重写的情况下hashCode方法默认返回对象在JVM中的内存地址，而equals方法比较的是两个对象的内存地址

## 3. == 和Object.equals()的区别

equals方法的默认实现是：

```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

在没有重写equals方法的情况下，其作用和==相同，都是比较两个对象在JVM的内存地址是否相同

## 4.为什么重写equals()方法一定要重写hashCode()方法

>  1为了维护hashCode方法的常规协定，协定声明相等两个对象其hashCode必须具有相同的hashCode

> 2许多散列数据存储结构如HashMap，都是根据hashCode来判断两个对象是否相同，如果不重写hashCode方法可能会导致存放两个等值的对象，产生混淆

# java.lang.String

## **1.String 对象的存放**
JDK1.6及之前字符串常量池存放在永久代中（属于方法区）
JDK1.7及以后字符串常量池存放在堆区中

## **2.String 对象的创建**

```java
//假设初始时字符串"a"和"b"均不在字符串常量池当中
//s1的创建过程：先在常量池中创建字符串"a"，然后在堆区中创建s1对象。
String s1 = new String("a");
//s2的创建过程：先在常量池中创建字符串"b"，然后s2直接复制常量池中的"b"的引用。
String s2 = "b";
```

对于创建未在常量池的中字符串，将会生成两个对象，即常量池中的字符串对象与堆区中的字符串对象

## 3.String.intern()

> 当调用字符串s的intern()方法时，如果常量池中有对象s1满足s1.equals(s)==true，则返回t的地址；否则，将s的引用（即地址）加入常量池并返回s的引用

## 4.常用方法

> equals()
> indexOf()
> lastIndexOf()
> startsWith()
> endsWith()
> join()
> toUpperCase()
> toLowerCase()
> trim()
> format()
> valueOf()
> replace()
> length()
> isEmpty()
> charAt()
> getBytes()

## 5.String、StringBuffer、StringBuilder的区别

> String是字符串常量，一旦赋值将无法改变，而StringBuffer和StringBuilder都是字符串变量，都是可以更改的
>
> StringBuilder是线程不安全的，而StringBuffer是线程安全的
>
> String适用于少量的字符串操作的情况；StringBuilder适用于单线程下在字符缓冲区进行大量操作的情况；StringBuffer适用多线程下在字符缓冲区进行大量操作的情况

# java.lang.Integer

## 1.Interger的缓存机制（Character、Btye、Short、Long类均有相同的缓存机制）

java.lang.Integer类内部维护了一个缓存池，默认保存了-128\~127之间的整型对象，在进行自动装箱的时候，会默认调用Integer.valueOf()方法，内部实现为：

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
}
```

如果被包装的int值在-128\~127之间，则会默认从缓存池中获取

缓存池的大小可以通过JVM启动参数进行设置：-XX:AutoBoxingCacheMax=256（将缓存池上限设置为256）

## 2.使用 == 和 euqals() 方法比较整型数值

### 2.1 == 比较

> 如果==的有一边是基本类型，包装类的一边将会进行自动拆包进行数值比较。如果两边都是包装类，将会进行地址比较。由于缓存池的机制，使用==比较可能会出现两个值相同的整型对象结果为false，如下面这种情况：
>
> ```java
> new Integer(10) == Integer.valueOf(10)	\\false
> ```

### 2.2 euqals比较

> Integer类重写了equals方法，在比较的时候会进行数值比较，因此可以放心使用

# java.lang.Throable

## 1.  java异常体系结构

![](img\Java异常体系.png)

java异常体系中位于顶层的是Throwable类，异常一共分为两种，一种是Error类，一种是Exception类

Error一般是程序无法处理的错误，由JVM产生和抛出，发生Error时，JVM一般会选择线程终止

Exception是程序本身可处理的异常，主要分为运行时异常Runtime Exception以及非运行时异常（如：I/O Exception，SQL Exception）

### 1.1 检查异常（checked exceptions）

检查异常就指编译器要求强制处理的异常，编译器在编译的时候会检查异常进行扫描，如果不对检查异常处理，那么编译器将无法通过编译。发生检查异常的代码往往不存在逻辑错误，只是会因为某些无法预料的情况才会发生错误，如：打开一个不存在的文件，文件被写保护等。当调用的方法可能抛出检查异常时，我们就要使用try catch块来对异常进行处理或者继续向上抛出。**除了RuntimeException及其子类，以及错误Error外，其他绝大多数异常都是检查异常。**



### 1.2 非检查异常（unchecked execptions）

非检查异常则是编译器不要求强制处理的异常，非检查异常往往是只有在运行时才有可能发生的异常，比如IndexOutOfBoundException越界异常，NPE空指针异常等。非检查异常往往都是代码逻辑引起的，比如没有进行空指针判断、没有对数组越界进行判断等。**Error及其子类和Runtime Exception及其子类都属于非检查异常。**

## 2. try、catch、finally语句块

使用try、catch块可进行异常捕获，如果try块中可能发生多种异常，则可以由多个catch语句块来捕获不同的异常。当有多个catch语句可以捕获同一异常时，只会匹配最近的catch语句块。因此如果try块中的多个异常存在继承关系，则应先从子类开始进行捕获。

finally语句块不论try块中是否发生了异常都会执行，它是异常处理的统一出口，通常在finally语句块中进行一些资源回收的操作（如：关闭打开的文件、删除临时资源等）。如果try块和catch块中均有返回语句，则finally的执行时刻是在try块或catch块执行完，返回之前执行。

如果finally中有返回语句，则将会覆盖try块和catch块中的返回语句，因此不建议在finally语句中进行返回。

## 3. 异常捕获

可以使用try、catch块进行异常捕获，也可直接使用throw关键字来抛出异常，但需要在方法签名中使用throws关键字声明会抛出的异常类型。

## 4. 异常处理

如果是用了try、catch块，建议在try、catch块中进行异常处理，否则在方法签名中声明异常的类型并交给调用者进行处理

