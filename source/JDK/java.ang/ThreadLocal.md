[TOC]

## ThreadLocal特性及使用场景

1、方便同一个线程使用某一对象，避免不必要的参数传递；

2、线程间数据隔离（每个线程在自己线程里使用自己的局部变量，各线程间的ThreadLocal对象互不影响）；

3、和线程的生命周期相同

**ThreadLocal应注意：**

1、ThreadLocal并未解决多线程访问共享对象的问题；

2、ThreadLocal并不是每个线程拷贝一个对象，而是直接new（新建）一个；

## ThreadLocalMap

### 概述

**是ThreadLocal的一个内部类，是Thread的私有属性**。

**如何达到隔离：每个线程都有自己的ThreadLocalMap**

**ThreadLocalMap的key是ThreadLocal，value是Object（即我们所谓的“线程本地数据”）。**

为避免占用空间较大或生命周期较长的数据常驻于内存引发一系列问题，hash table的key是弱引用**WeakReferences**。当空间不足时，会清理未被引用的entry。

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

### 图解ThreadLocal

**每个线程可能有多个ThreadLocal，同一线程的各个ThreadLocal存放于同一个ThreadLocalMap中。**

![img](https://img-blog.csdn.net/20170125180420388?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDg4Nzc0NA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



### ThreadLocalMap与hashMap对比

#### hash的计算

HashMap的key.hash的计算方式是native、异或、无符号位移，(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16)，ThreadLocalMap的key.hash从ThreadLocal实例化时便由nextHashCode()确定。

```java
private final int threadLocalHashCode = nextHashCode();
private static final int HASH_INCREMENT=0x61c88647;
private static AtomicInteger nextHashCode = new AtomicInteger();
private static intnextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}
```

#### 存储结构

**HashMap（JDK8）底层数据结构是位桶+链表/红黑树，而ThreadLocalMap底层数据结构是Entry数组（有冲突直接覆盖原有值）**

## ThreadLocal常见方法

### get()

```java
public T get(){
    Thread t= Thread.currentThread(); // 获取当前线程
    ThreadLocalMap map= getMap(t); // 获取当前线程对应的Map
   if(map!=null){
        ThreadLocalMap.Entry e = map.getEntry(this); // 详见3.1.1
       if(e!=null){ // map不为空且当前线程有value，返回value
            @SuppressWarnings("unchecked")
            T result=(T)e.value;
           return result;
       }
   }
   return setInitialValue(); // 初始化再返回值
}
```

```java
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

```java
private T setInitialValue(){
    T value= initialValue(); //调用重写的initialValue，返回新值
    Thread t= Thread.currentThread();
    ThreadLocalMap map= getMap(t);
   if(map!=null) // 当前线程的ThreadLocalMap不为空，则直接赋值
        map.set(this, value);
   else
// 为当前线程创造一个ThreadLocalMap(this, firstValue)并赋初值，this为当前线程
        createMap(t, value);
   return value;
}
protected T initialValue() {
    return T; // 自定义返回值
};
```

```java
void createMap(Thread t, T firstValue){
    t.threadLocals=new ThreadLocalMap(this, firstValue);
}
```

### set()

```java
public void set(T value){
    Thread t= Thread.currentThread();
    ThreadLocalMap map= getMap(t);
   if(map!=null)
        map.set(this, value);
   else
        createMap(t, value);
}
```

### remove()

```java
public void remove(){
     ThreadLocalMap m= getMap(Thread.currentThread());
    if(m!=null)
         m.remove(this);
 }
```

## ThreadLocal使用

首先ThreadLocal肯定是全局共享的（static，在多个线程共享）：

```java
public class Tools
{
    public static ThreadLocal<String> t1 = new ThreadLocal<String>();
}
```

写一个线程往ThreadLocal里面塞值：

```java
public class ThreadLocalThread extends Thread
{
    private static AtomicInteger ai = new AtomicInteger();
    
    public ThreadLocalThread(String name)
    {
        super(name);
    }
    
    public void run()
    {
        try
        {
            for (int i = 0; i < 3; i++)
            {
                Tools.t1.set(ai.addAndGet(1) + "");
                System.out.println(this.getName() + " get value--->" + Tools.t1.get());
                Thread.sleep(200);
            }
        }
        catch (InterruptedException e)
        {
            e.printStackTrace();
        }
    }
}
```

写个main函数，启动三个ThreadLocalThread：

```java
public static void main(String[] args) throws Exception
    {
        ThreadLocalThread a = new ThreadLocalThread("ThreadA");
        ThreadLocalThread b = new ThreadLocalThread("ThreadB");
        ThreadLocalThread c = new ThreadLocalThread("ThreadC");
        a.start();
        b.start();
        c.start();
    }
```

看一下运行结果：

```java
ThreadA get value--->1
ThreadC get value--->2
ThreadB get value--->3
ThreadB get value--->4
ThreadC get value--->6
ThreadA get value--->5
ThreadC get value--->8
ThreadA get value--->7
ThreadB get value--->9
```

看到每个线程的里都有自己的String，并且互不影响----因为绝对不可能出现数字重复的情况。用一个ThreadLocal也可以多次set一个数据，set仅仅表示的是线程的ThreadLocal.ThreadLocalMap中table的某一位置的value被覆盖成你最新设置的那个数据而已，**对于同一个ThreadLocal对象而言，set后，table中绝不会多出一个数据**。

## 参考

<https://blog.csdn.net/u010887744/article/details/54730556>

[**ThreadLocal的使用**](https://www.cnblogs.com/xrq730/p/4854820.html)

