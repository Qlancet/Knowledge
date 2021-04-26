



# CAS

​		compareAndSwapObject(Object var1, long var2, Object var4, Object var5);

​		compareAndSwapInt(Object var1, long var2, int var4, int var5);

​		compareAndSwapLong(Object var1, long var2, long var4, long var6);

​		如果偏移量是对象就使用第一个,如果属性是int就是用第二个,如果属性是long就是用第三个

​		var1:要操作的对象

​		var2: 要操作对象中属性地址的偏移量

​		var4: 需要修改数据的期望值

​		var5: 需要修改数据的新值

​		

# 自旋锁		

​		是指当一个线程在获取锁的时候，如果锁已经被其它线程获取，那么该线程将**循环等待**，然后不断的判断锁是否能够被成功获取，直到获取到锁才会退出循环。获取锁的线程**一直处于活跃状态**，但是并没有执行任何有效的任务，使用这种锁会造成busy-waiting。

模拟自旋锁

```java
package com.czbank.queue;

import java.util.concurrent.atomic.AtomicReference;

public class SpinLockDemo {
    public static void main(String[] args) {
        AtomicReference<Thread> cas = new AtomicReference<Thread>();
        Thread thread1 = new Thread(new Task(cas));
        Thread thread2 = new Thread(new Task(cas));
        thread1.start();
        thread2.start();
    }
}

//自旋锁验证
class Task implements Runnable {
    private AtomicReference<Thread> cas;
    private spinlock slock;

    public Task(AtomicReference<Thread> cas) {
        this.cas = cas;
        this.slock = new spinlock(cas);
    }
    @Override
    public void run() {
        slock.lock(); //上锁
        for (int i = 0; i < 10; i++) {
            //Thread.yield();
            System.out.println(i);
        }
        slock.unlock();
    }
}

class spinlock {
    private AtomicReference<Thread> cas;

    spinlock(AtomicReference<Thread> cas) {
        this.cas = cas;
    }

    public void lock() {
        Thread current = Thread.currentThread();
        // 利用CAS
        while (!cas.compareAndSet(null, current)) { //为什么预期是null？？
            // DO nothing
            System.out.println("I am spinning");
        }
    }

    public void unlock() {
        Thread current = Thread.currentThread();
        cas.compareAndSet(current, null);
    }
}

```

优点:**自旋锁不会使线程状态发生切换，一直处于用户态，即线程一直都是active的；不会使线程进入阻塞状态，减少了不必要的上下文切换，执行速度快**

缺点:**如果某个线程持有锁的时间过长，就会导致其它等待获取锁的线程进入循环等待，消耗CPU。使用不当会造成CPU使用率极高**



# 可重入锁

参考: https://www.cnblogs.com/hustzzl/p/9343797.html

即当一个线程第一次已经获取到了该锁，在锁释放之前又一次重新获取该锁，第二次就不能成功获取到

```java
//可重入锁-
package com.czbank.queue;

public class MyLockTest implements Runnable {
    public synchronized void get() {
        System.out.println("2 enter thread name-->" + Thread.currentThread().getName());
        //reentrantLock.lock();
        System.out.println("3 get thread name-->" + Thread.currentThread().getName());
        set();
        //reentrantLock.unlock();
        System.out.println("5 leave run thread name-->" + Thread.currentThread().getName());
    }

    public synchronized void set() {
        //reentrantLock.lock();
        System.out.println("4 set thread name-->" + Thread.currentThread().getName());
        //reentrantLock.unlock();
    }

    @Override
    public void run() {
        System.out.println("1 run thread name-->" + Thread.currentThread().getName());
        get();
    }

    public static void main(String[] args) {
        MyLockTest test = new MyLockTest();
        for (int i = 0; i < 5; i++) {
            new Thread(test, "thread-" + i).start();
        }
    }
}
```

```java
//可重入锁
package com.czbank.queue;

import java.util.concurrent.locks.ReentrantLock;

public class MyLockTest_enter implements Runnable {

    private ReentrantLock reentrantLock = new ReentrantLock();

    public void get() {
        System.out.println("2 enter thread name-->" + Thread.currentThread().getName());
        reentrantLock.lock();
        System.out.println("3 get thread name-->" + Thread.currentThread().getName());
        set();
        reentrantLock.unlock();
        System.out.println("5 leave run thread name-->" + Thread.currentThread().getName());
    }

    public void set() {
        reentrantLock.lock();
        System.out.println("4 set thread name-->" + Thread.currentThread().getName());
        reentrantLock.unlock();
    }

    @Override
    public void run() {
        System.out.println("1 run thread name-->" + Thread.currentThread().getName());
        get();
    }

    public static void main(String[] args) {
        MyLockTest_enter test = new MyLockTest_enter();
        for (int i = 0; i < 3; i++) {
            new Thread(test, "thread-" + i).start();
        }
    }

}
```

