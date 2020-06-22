[TOC]

## 概述

如何使用线程，线程的两个重要属性（线程优先级，守护线程），线程状态，线程安全暂停，等待通知模式，安全取消线程

## 分析

### 1、线程优先级

```java
 /**
  * The minimum priority that a thread can have.
  */
 public final static int MIN_PRIORITY = 1;

/**
  * The default priority that is assigned to a thread.
  */
 public final static int NORM_PRIORITY = 5;

 /**
  * The maximum priority that a thread can have.
  */
 public final static int MAX_PRIORITY = 10;
```

最低为1，最高为10，默认优先级为5. 

### 2、守护线程

```java
public final void setDaemon(boolean on) {
  checkAccess();
  if (isAlive()) {
      throw new IllegalThreadStateException();
  }
  daemon = on;
}

/**
* Tests if this thread is a daemon thread.
*
* @return  <code>true</code> if this thread is a daemon thread;
*          <code>false</code> otherwise.
* @see     #setDaemon(boolean)
*/
public final boolean isDaemon() {
  return daemon;
}
```

### 3、线程状态

```java
public enum State {
  NEW,
  RUNNABLE,
  BLOCKED,
  WAITING,
  TIMED_WAITING,
  TERMINATED;
}
```

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20170409174042869?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGg1MTM4Mjg1NzA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**阻塞分为同步阻塞与IO阻塞等**

### 4、sleep-yield

**sleep方法**：sleep相当于让线程睡眠，交出CPU，让CPU去执行其他的任务。sleep方法不会释放锁
**yield方法：**调用yield方法会让当前线程交出CPU权限，让CPU去执行其他的线程。它跟sleep方法类似，同样不会释放锁。 

**废弃的暂停线程的方法**

**stop()**强行把执行到一半的线程终止，不正确释放资源
**挂起(suspend)**：不会释放资源，导致死锁

### 5、wait - notify - notifyAll方法

**这三个方法为java.lang.Object类为线程提供的用于实现线程间通信的同步控制方法。因此，可以说每一个对象实例都存在一个等待队列。**

```java
 //源码中，可以看到wait的使用方法推荐如下
  synchronized (obj) {
              while (condition does not hold){
                obj.wait();
              }
              .....     
         }
 //使用wait时，必须先持有当前对象的锁，否则会抛错InterruptedException
```

### 6. join方法

**本质是调用wait**

```java
 //名词解释：例如 main(){ ***；  t.join()；  ****； }
 //当前‘主线程’ 指的是当前执行t.join()的main主线程
 //当前‘线程’指的就是t
 public final synchronized void join(long millis)
    throws InterruptedException {
        //获取当前时间
        long base = System.currentTimeMillis();
        long now = 0;

        if (millis < 0) {
            throw new IllegalArgumentException("timeout value is negative");
        }
        //mills等于0时，当前‘主线程’对象存活时，会一直阻塞，直到当前‘线程’run执行完毕或抛出异常终止
        if (millis == 0) {
            while (isAlive()) {
                wait(0);
            }
        } else {
            while (isAlive()) {
                long delay = millis - now;
                if (delay <= 0) {
                    break;
                }
                 //当前‘主线程’阻塞millis毫秒
                wait(delay);
                now = System.currentTimeMillis() - base;
            }
        }
    }

   //线程真正退出之前，系统会调用当前方法，给它一个清理的机会
   private void exit() {
        if (group != null) {
            //通知线程组，该线程已经死亡
            //满足某些条件下，该处会执行执行this.notifyAll（）方法
            group.threadTerminated(this);
            group = null;
        }
        /* Aggressively null out all reference fields: see bug 4006245 */
        target = null;
        /* Speed the release of some of these resources */
        threadLocals = null;
        inheritableThreadLocals = null;
        inheritedAccessControlContext = null;
        blocker = null;
        uncaughtExceptionHandler = null;
    }


```

### 7、中断

**线程中断**并不会使线程立即退出，而是给线程发送一个通知，告知目标线程，有人希望你退出啦！至于目标线程接到通知后如何处理，则完全由目标线程自行 决定。这点很重要，如果中断后，线程立即无条件退出，就会遇到stop同样的问题。

```java
public void interrupt() // 线程中断
public boolean isInterrupted() // 判断是否被中断
public static boolean interrupted() // 判断是否被中断，并清除当前中断状态
```

**非阻塞方法通过调用isInterrupted方法判断是否中断来处理中断请求**

**阻塞方法中断时会跑出异常，必须捕获异常来处理中断**

```java
public class HasInterrputException {
        private static  class UseThread extends  Thread{
            public UseThread(String name){
                super(name);
            }

            @Override
            public void run() {
                while (!isInterrupted()){
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        System.out.println("the  flag  is "+isInterrupted());
                        e.printStackTrace();
                        interrupt();
                        System.out.println("the  flag2  is "+isInterrupted());
                    }
                }
            }
//        线程调用sleep方法后进入sleep状态，而sleep方法中java在实现的时候支持对中断标志位的检查，
//        一旦sleep方法检查到了中断标志位为true，就会终止sleep，并抛出这个InterruptedException。
//        方法里如果抛出InterruptedException,
//        线程的中断标志位会被复位成false，如果确定是需要中断线程，
//        要求我们自己在catch语句块里再次调用interrupt()
//        InterruptedException表示一个阻塞被中断了，阻塞包括sleep(),wait()

            public static void main(String[] args) throws InterruptedException {
                Thread endThread = new UseThread("HasInterrputEx");
                endThread.start();
                Thread.sleep(500);
// 为什么加上Thread.sleep(500)，就会有异常发生，注释掉就没有呢
// 因为调用interrupt的时候，子线程甚至还么来的及初始化完成
                endThread.interrupt();
            }
        }
}
```

## 参考

<https://blog.csdn.net/m0_37276008/article/details/79947787>

<https://blog.csdn.net/pf1234321/article/details/80373227>

<https://blog.csdn.net/lh513828570/article/details/60144598>