### 基本概念了解

- 进程：应用程序在内存中运行的空间。
- 线程：进程中的一个执行单元，负责进程中的程序执行。

### 创建线程

1. 继承Thread
   - 定义一个类继承Thread
   - 重写run方法
   - 创建子类对象(创建线程对象)
   - 调用start()的方法(开启线程，调用run方法内容)

> 线程对象调用run方法和调用start方法的区别?
>
> > 调用run()方法不开启线程，仅是对象调用方法。
>
> > 调用start()开启线程，并让jvm调用run()

2. 实现Runnable接口(避免单继承的局限性)
   - 定义类实现Runnable接口
   - 覆盖接口中的run()方法，将线程任务代码定义到run()中
   - 创建Thread类的对象
   - 将Runnable接口的子类对象作为参数传递给Thread类的构造函数
   - 调用Thread类的start()方法开启线程

> 实现Runnable的优势
>
> > 避免了单继承的局限性
> >
> > 实现Runnable接口更符合面向对象，线程分为两部分，一部分是线程对象(Thread类)，另一部分是线程任务(Runnable的子类)。而继承Thread类，线程对象和线程任务耦合在了一起。

### 线程的状态

![ThreadState](.\image\ThreadState.PNG)



### 案例

```java
/**
	售票案例
售票的动作需要同时执行，所以需要使用多线程技术
*/

 class Sale implements Runnable{
	
	private int ticketNum = 100; // 设置默认票数100张
	/**
		多线程的任务
		实质就是售票的逻辑代码
	*/ 
	public void run(){
		while(true){
			if(ticketNum > 0){
                try{Thread.sleep(10);}catch(InterruptedException e){}
				System.out.println(Thread.currentThread().getName() + "买到一张票,剩余"+ --ticketNum);
			}		
		}
	}
 }
 
 public class SaleTicket{
	
	public static void main(String args[]){
		Sale s1 = new Sale();
		Thread t1 = new Thread(s1);
		t1.setName("黄牛one");
		Thread t2 = new Thread(s1);
		t2.setName("黄牛two");
		t1.start();
		t2.start();
	}
 }
```

- 这个案例出现了线程安全问题，出现错误问题的原因
  - 多个线程在操作共享的数据
  - 线程任务操作共享数据的代码有多条(运算有多个)

- 解决思路
  - 只要让一个线程在执行线程任务时将多条操作共享数据的代码执行完过程中，不让其他线程参与运算。
  - java中实现通过 synchronized(对象){ code} 同步代码块

```java
class SaleTicket{
    private int ticketNum = 100;
    private Object obj = new Object();
    public void run(){
            while(true){
                //obj就像是一把锁，有线程进入到代码块执行时，先获取Object对象，在该线程没有释放锁时，其他线程不能参与到该代码块的运算。
                /*注意：如果将obj 用new Object()代替时不会解决线程安全问题
                  ★同步的前提: 必须保证多个线程在同步中使用的是同一个锁。所以如果使用new Object()来充当锁时，每个线程进来会拿到不同的锁。
                */          
                synchronized(obj)	{
                        if(ticketNum > 0){
                        try{Thread.sleep(10);}catch(InterruptedException e){}
                        System.out.println(Thread.currentThread().getName() + "买到一张票,剩余"+ --ticketNum);
                    }	
                }
            }
        }
}
```

- 同步的优缺点
  - 优点：解决了多线程安全问题
  - 缺点：降低了程序的性能

- 同步的体现

  - 同步代码块 

    ```java
    private Object obj = new Object();
    synchronized(obj){
        //code
    }
    ```

  - 同步方法

    ```java
    //同步方法使 用的锁对象是this
    private synchronized void method(){
        //code
    }
    //特例 静态同步方法，使用的锁不是this，而是字节码文件对象(类名.class)。
    ```

    >同步方法和同步代码块的特点
    >
    >>同步代码块的锁可以使用人任意对象，当线程任务中需要多个同步是时，必须通过同步代码块来实现。
    >>
    >>同步方法的锁是固定的this。

- 同步的另一个弊端 死锁

  - case 1： 多线程任务中出现多个同步时，如果同步中嵌套了其他的同步，容易引起死锁。(要避免这种写法)

    ```java
    //Thread-0
    synchronized(obj1){
        //Thread-0拿到了obj1锁,Thread-1拿不到obj2锁，没有执行完也不会释放锁，就引起死锁。
        synchronized(obj2){
            
        }
    }
    
    //Thread-1
    synchronized(obj2){
        //Thread-1拿到了obj2锁
        synchronized(obj1){
              
        }
    }
    ```

  - case 2 ：所有的线程都处于wait()状态，在生产者消费者模式中提及。

### 等待唤醒机制

- wait()：会让线程处于等待状态，其实就是将线程临时存储到了线程池中。

- notify()：会唤醒线程池中任意一个等待的线程。

- notifyAll()：会唤醒线程池中所有的等待的线程。

  **这些方法必须在同步中使用，因为必须要标识wait(),notify()等方法所属的锁，同一个锁上notify()，只能唤醒该锁上被wait()的线程**

   **这些方法定义在Object类中，因为这些方法会标识所属的锁，而锁是任意对象，任意对象可以调用的方法只能定义在Object中**

---

#### java中的另一种同步机制

**在jdk 1.5中提供了新的同步机制，在包java.util.concurrent.locks中提供了Lock接口来代替synchronized代码块，还提供了Condition接口代替了旧的监视器方法(wait,notify,notifyAll)**

- 这种同步机制将锁和监视器方法解耦，更符合面向对象的设计。

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

class LockTest implements Runnable{
    private int num = 1
    //创建锁 通过子类
    private Lock lock = new ReentrantLock();
    //创建监视器
    private Condition con = lock.newCondition();
    
    //演示新同步机制的使用
    public void run(){
        //由于synchronized代码块是隐试的执行获得锁和释放锁的过程，所以不用考虑程序出现异常时跳转到其它程序时还是否能执行到释放锁的代码，而Lock是手动的获得锁，释放锁所以要考虑这种情况。如下解决
        /* 监视器方法用法和Objec监视器方法一样。 Lock锁的对象是Lock
        	lock.await();
        	lock.signal();唤醒一个等待线程。 
        	lock.signalAll();唤醒所有等待线程。
        */
        try{
            lock.lock();//获得锁
            //逻辑代码
        }finally{
            lock.unlock(); //释放锁
        }
        
    }
    
}
```

### 多线程中的细节

- sleep() 和 wait() 的异同点。
  - 相同点
    - sleep() 和 wait()都可以使线程处于冻结状态。
  - 不同点
    - sleep()必须指定时间；wait()可以指定也可以不指定
    - sleep()时间到，线程处于阻塞或者运行状态；wait()如果没有时间，必须通过notify()唤醒
    - sleep()不一定要定义在同步中；wait()必须定义在同步中
    - ★在同步中,线程执行到sleep()，不会释放锁；线程执行到wait()，会释放锁
- 线程如何停止？
  - ~~stop()~~方法：已过时
  - run()方法：让run()执行完
    - run()方法中通常定义循环，只要控制住循环就可以停止线程(通常定义标记控制循环)。
    - 如果线程在任务中处于冻结状态，就不能去判断标记。应通过interrupt()方法来中断该等待；interrupt的功能是将线程的冻结状态清除，让线程强制处于运行状态。因为是强制恢复运行状态所以会抛异常，所以可以在catch中捕获异常。
  - 守护线程：

  ### 线程的优先级

​	**线程的优先级用数字1-10来标识，最明显的优先级是1，5，10**

 ### join() & yield()

- join()：主线程执行到这里释放执行权，等到调用者线程执行完毕恢复执行资格。
- yield()：线程临时暂停，将执行权释放。

​     