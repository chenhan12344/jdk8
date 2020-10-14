# JAVA引用

Java当中引用分为以下四种：

- 强引用（StrongReference）
- 软引用（SoftReference）
- 弱引用（WeakReference）
- 虚引用（PhantomReference）

引用于对象的生命周期密切相关，并且可以通过引用来追踪对象的声明周期，判断对象是否被回收，有助于排查内存泄露等问题

Java提供引用的目的主要是：

- 可以让开发人员通过代码的方式来决定对象的声明周期
- 有利于JVM进行垃圾回收

## 1.强引用

强引用是最常见的引用，即代码中对一个对象的引用即为强引用。只要一个对象存在着强引用，即使JVM内存不足时，垃圾收集器也不会去回收具有强引用的对象，而是直接抛出OOM。

```java
// 这里的ref即为强引用
Object ref = new Object();
```

强引用只有在超出对象声明周期范围（作用域）或者被显示地赋值为null时，强引用才会失效，此时对象就可以被垃圾收集器进行回收

```java
{
    // 声明强引用
    Object ref = new Object();
    // 显示地赋值为null，强引用失效
    ref = null;
}
// 此处已经超出了声明周期范围（作用域），强引用也会失效
// ref = null
```

## 2.软引用

如果一个对象具有软引用，当JVM内存空间**足够**时，垃圾收集器不会回收具有软引用的对象，当JVM内存**不足**时，具有软引用的对象就会被垃圾收集器回收。

基于软引用性质，软引用常常可以用来做内存敏感型的高速诉缓存，这样当内存足够时，可以将空闲的内存用于缓存暂时不用的对象，当需要的时候可以立即获取并使用，避免了重新创建对象带来的开销。而当内存不足时，这些缓存对象可以被回收释放占用的内存空间。

Java当中提供了```SoftReference<T>```来标记一个对象的软引用

```java
SoftReference<String> ref = new Softreference<>(new String("123"));
System.out.println(ref.get()); 	// 输出123
System.gc(); 	// 通知JVM应当进行垃圾回收
Sysyem.out.println(ref.get());	// 此时JVM内存充足，软引用不会被回收，因此输出123
```

## 3.弱引用

弱引用也是用来描述非必需的对象，当一个对象只具有弱引用时，**不论JVM内存是否足够**，只要进行垃圾回收，具有弱引用的对象**都会被回收**。

Java当中提供了```WeakReference<T>```来标记一个对象的弱引用

```java
WeakReference<String> ref = new WeakReference<>(new String("123"));
System.out.println(ref.get()); 	// 输出123
System.gc(); 	// 通知JVM应当进行垃圾回收
Sysyem.out.println(ref.get());	// 不管JVM内存是否充足，弱引用都会被，因此输出null
```

弱引用常常和引用队列```ReferenceQueue```相关联合使用，当一个只具有弱引用的对象被垃圾收集回收时，JVM就会将这个对象的弱引用加入到与之关联的引用队列当中。

通过联用弱引用与引用队列，就可以检测内存泄露，LeakCanary就是通过弱引用来跟踪Activity的生命周期，并结合引用队列来判断Activity是否被垃圾收集器及时回收，进而检测内存泄露。

## 4.虚引用

虚引用是4中引用类型中最弱的一种，虚引用并不能决定对象的生命周期，一个对象是否有虚引用不对其声明周期构成任何影响，并且也无法通过虚引用来获得一个对象的实例，因为如果能够通过虚引用获取一个对象的实例，就可以给该对象设置强引用，从而影响了对象的生命周期。

Java当中提供了```PhatomReference```来标记一个对象的虚引用。由于通过虚引用无法获取到对象实例，因此虚引用必须和引用队列关联使用。

```java
ReferenceQueue<String> queue = new ReferenceQueue<>(); // 虚引用必须关联引用队列
PhantomReference<String> ref = new PhantomReference(new String("123"), queue);
System.out.println(ref.get());    // 输出null，无法从虚引用中获取对象实例
System.gc();
System.out.println(ref.get());  // 输出null，无法从虚引用中获取对象实例
```

虚引用的作用在于，当垃圾收集器准备回收一个对象时，如果这个对象还有虚引用，垃圾收集器在回收完这个对象后将这个对象的虚引用添加到引用队列中，并且在与之关联的引用队列该虚引用出队前，不会彻底销毁该对象。因此，可以通过检查引用队列中的虚引用是否还存在来判断对象是否被回收了。

# java.lang.Reference.ReferenceHandler

```java
// ReferenceHanlder继承自Thread类，因此是一个可运行的线程
private static class ReferenceHandler extends Thread {
    
    private static void ensureClassInitialized(Class<?> clazz) {
        try {
            Class.forName(clazz.getName(), true, clazz.getClassLoader());
        } catch (ClassNotFoundException e) {
            throw (Error) new NoClassDefFoundError(e.getMessage()).initCause(e);
        }
    }
    static {
        // pre-load and initialize InterruptedException and Cleaner *
        // classes
        // so that we don't get into trouble later in the run loop if there's
        // memory shortage while loading/initializing them lazily.
        ensureClassInitialized(InterruptedException.class);
        ensureClassInitialized(Cleaner.class);
    }
    
    //构造函数
    ReferenceHandler(ThreadGroup g, String name) {
        super(g, name);
    }
    // ReferenceHandler的主要作用，就是一直执行tryHandlePending
    public void run() {
        while (true) {
            tryHandlePending(true);
        }
    }
}

static boolean tryHandlePending(boolean waitForNotify) {
    Reference<Object> r;
    Cleaner c;
    try {
        synchronized (lock) {
            if (pending != null) {
                r = pending;
                // 'instanceof' might throw OutOfMemoryError sometimes
                // so do this before un-linking 'r' from the 'pending' chain...
                c = r instanceof Cleaner ? (Cleaner) r : null;
                // unlink 'r' from 'pending' chain
                pending = r.discovered;
                r.discovered = null;
            } else {
                // The waiting on the lock may cause an OutOfMemoryError
                // because it may try to allocate exception objects.
                if (waitForNotify) {
                    lock.wait();
                }
                // retry if waited
                return waitForNotify;
            }
        }
    } catch (OutOfMemoryError x) {
        // Give other threads CPU time so they hopefully drop some live references
        // and GC reclaims some space.
        // Also prevent CPU intensive spinning in case 'r instanceof Cleaner' above
        // persistently throws OOME for some time...
        Thread.yield();
        // retry
        return true;
    } catch (InterruptedException x) {
        // retry
        return true;
    }
    // Fast path for cleaners
    if (c != null) {
        c.clean();
        return true;
    }
    ReferenceQueue<? super Object> q = r.queue;
    if (q != ReferenceQueue.NULL) q.enqueue(r);
    return true;
}
```

