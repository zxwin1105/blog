# 一、基础概念
## 1.1 进程和线程概念
### 1. 进程
进程是一段静态代码程序的一次动态执行，
### 2. 线程
线程是进程中的一个执行单元，负责进程中的程序执行，一个进程中可能有多个线程。
# 二、Java中线程的基本操作
## 2.1 创建多线程
在java中，创建线程的方式有两种。一种是继承Thread类；一种是实现Runable接口
### 1. 继承Thread类

1. 定义一个类继承Thread
1. 重写run()方法，run()方法用来定义线程任务
1. 创建子类对象，也就是创建线程对象
1. 线程对象调用start()方法，开启线程，执行操作
```java
public class TestThread{
    public static void main(String[] args) {
        Thread_ t1 = new Thread_("zxwin");
        Thread_ t2 = new Thread_("zxw");
        t2.start(); // 线程对象，调用任务方法
        t1.run(); // 调用普通方法

    }
}
// 多线程类
class Thread_ extends Thread{

    private String name;
    Thread_(String name){
        this.name = name;
    }

    @Override
    public void run() {
        for (int i = 1; i <= 20; i++) {
            System.out.println("name:"+name+"------>"+i);
        }
    }
}
```
## 2. 实现Runnable接口

1. 创建类实现Runnable接口
1. 重写run()方法
1. 创建该类的实例，并作为参数传递给线程对象
1. 调用线程对象的start()方法
```java
public class TestRunnable {
    public static void main(String[] args) {
        // 实例化线程对象
        Thread t1 = new Thread(new Runnable_());
        Thread t2 = new Thread(new Runnable_());
        // 使用线程对象调用start()方法
        t1.start();
        t2.start();
    }
}
class Runnable_ implements Runnable{
    @Override
    public void run() {
        for (int i = 1; i <= 20; i++) {
            System.out.println(Thread.currentThread().getName() + "---->" + i);
        }
    }
}
```
## 2.2 Java中多线程注意问题
### 1. 使用线程对象调用run()和start()方法的区别？
调用run()方法不会开去线程，仅是一个线程对象调用一个普通方法
调用start()方法会开启线程，并让jvm调用run()方法，在开启的线程中执行
### 2. 使用Runnable 的优势

1. 使用实现Runnable接口的方式比Thread更加符合面向对象编程方式，Runnable方式将线程对象和线程任务分成两部分，Thread方式将线程对象和线程任务耦合在一起。（Runnable接口用来定义线程任务）
1. 使用Runnable的方式可以避免单继承的局限性
### 3 线程状态
![threadStatus](E:\Blog\Java\image\ThreaStatus.png)

## 3.1 线程安全问题示例
以下代码是模拟售票的功能，在开启多个线程同时开始售票时，该程序会出现一些线程问题，如下
```java
public class ThreadProblem {
    public static void main(String[] args) {
        Ticket ticket = new Ticket();
        Thread t1 = new Thread(ticket);
        Thread t2 = new Thread(ticket);
        Thread t3 = new Thread(ticket);
        Thread t4 = new Thread(ticket);
        t1.start();
        t2.start();
        t3.start();
        t4.start();

    }
}
// 模拟售票功能
class Ticket implements Runnable{
    private Integer ticket = 100;
    @Override
    public void run() {
        while (true){
            try {
                Thread.sleep(10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            if (ticket > 0){
                System.out.println(Thread.currentThread().getName() + "--->" + ticket--);
            }
        }
    }
}
```
程序的执行结果：
> Thread-1--->10
> Thread-3--->9
> Thread-1--->7
> Thread-0--->7
> Thread-2--->8
> Thread-3--->6
> Thread-2--->5

> Thread-1--->3

> Thread-0--->4

> Thread-3--->2

> Thread-0--->1

> Thread-2--->0

> Thread-1--->-1

从程序的执行结果来看，有些票同时被卖出去了多张，甚至还出现了负数。这显然不合理#
## 3.2 线程安全问题分析
jvm会为每个运行期间的线程开辟一块独立的栈空间，这些线程在运行期间会访问共同的数据ticket。由于cpu工作的原理是按照自己的计算方式对每个线程进行执行，可能出现问题，当ticket=1时，这是Thread-0得到cpu执行权，当Thread-0刚执行完if语句后还没有对ticket--，线程Thread-1就得到了cpu的执行权，此时ticket=1，当Thread-1执行完线程任务后ticket=0，此时Thread-0得到cpu执行权后执行ticket--，就会出现负数的票，也就是发生了线程安全问题。
### 1. 通过以上对线程安全的分析，可以得出发生线程安全的两个原因：

- 存在多个线程共享同一个资源
- 线程任务操共享资源的代码有多条
### 2. 解决线程安全问题的思路
通过发生线程安全的两个原因，我们可以控制让一个线程在执行操作共享数据的任务代码时，必须要执行完其代码，才运行其他线程进入。
在Java种提供的同步代码块来解决线程安全问题
## 3.3 多线程同步
### 1. Java中的几种同步方式

1. 同步代码块
```java
// 此时的同步锁可以是任意对象
synchronized (/*同步锁*/) { // 同步代码}
```

2. 同步方式
```java
// 此时的同步锁是this
public synchrnoized void test() {
	// 同步代码
}
```

3. 静态同步方法
```java
// 此时同步锁是类的字节码对象 类.class
public synchronized static void test(){
	// 同步代码
}
```
**使用同步代码块解决多线程安全问题的前提是必须在同步中使用同一把锁**
### 2. 同步常见问题

1. 同步的优点：同步可以解决多线程的安全问题
1. 同步的缺点：
   - 使用同步解决安全问题时会降低程序的效率
   - 当线程任务中出现了多个同步时（多个锁），如果同步中嵌套了同步，这是容易引发死锁
3. 解决同步效率的方法：通常可以通过双重判断的方式来解决同步效率问题
# 四、线程的死锁
对于解决多线程中的死锁问题，可以采用生产者消费者模式来解决。
示例代码
```java
public class ProducerConsumer1 {
    public static void main(String[] args) {
        Resource resource = new Resource();

        Producer producer = new Producer(resource);
        Consumer consumer = new Consumer(resource);
        Thread pro = new Thread(producer);
        Thread con = new Thread(consumer);
        pro.start();
        con.start();
    }
}

// 资源类（多个线程共享的资源）
class Resource{
    // 生成的面包
    private String name;
    private int count = 1;
    private boolean flag = false;
    // 用于生成资源的操作
    public synchronized void set(String name) throws InterruptedException {
        if (this.flag) {
            wait();
        }
        this.name = name + this.count++;
        System.out.println(Thread.currentThread().getName() + "-----生产了-----" + this.name);
        this.flag = true;
        notify();
    }

    // 用于消费资源的操作
    public synchronized void out() throws InterruptedException {
        if (!this.flag) {
            wait();
        }
        System.out.println(Thread.currentThread().getName()+"-----消费了-----"+this.name);
        this.flag = false;
//        notify();
    }
}

// 生产者
class Producer implements Runnable{
    Resource resource;
    Producer(Resource resource){
        this.resource = resource;
    }

    @Override
    public void run() {
        while (true){
            try {
                resource.set("面包");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}

// 消费者
class Consumer implements Runnable{
    Resource resource;
    Consumer(Resource resource){
        this.resource = resource;
    }

    @Override
    public void run() {
        while (true){
            try {
                resource.out();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
## 4.1 方法介绍

- wait()：让当前持有锁的线程进入等待线程池
- notify()：唤醒等待线程池中的任意一个线程
- notifyAll()：唤醒等待线程池中的所有线程
> 上列方法为什么定义在Object类中？
> 在同步中同步锁可以是任意对象，以上方法对于线程的操作（停止，唤醒），需要靠锁标识（也就是对象），任意对象可以调用的方法必然定义在Object类中

## 4.2 Lock
在jdk1.5后提供了实同步的新方式Lock接口，Lock是一个锁可以用来替换同步，Condition用来替换Object中的监视器方法
```java
public class ProducerConsumer2 {
    public static void main(String[] args) {
        Resource2 resource2 = new Resource2();

        Producer2 pro = new Producer2(resource2);
        Consumer2 con = new Consumer2(resource2);

        Thread p1 = new Thread(pro);
        Thread p2 = new Thread(pro);
        Thread c1 = new Thread(con);
        Thread c2 = new Thread(con);
        p1.start();
        c1.start();
        p2.start();
        c2.start();

    }
}
class Resource2 {
    private final LinkedList<String> res = new LinkedList<>();
    private int count = 1;

    private Lock lock = new ReentrantLock();
    //
    private Condition proCondition = lock.newCondition();
    private Condition conCondition = lock.newCondition();

    // 生产资源,最大容量10
    public void set(String resName){
        lock.lock();
        try {
            while(res.size() == 10) proCondition.await();
            String s = resName+this.count++;
            res.add(s);
            System.out.println(Thread.currentThread().getName()+"-----生产了-----"+s + "===="+res.size());
            Thread.sleep(100);
            conCondition.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    // 消费资源
    public void out() {
        lock.lock();
        try {
            while(res.size() ==0) conCondition.await();
            String o = res.remove();
            System.out.println(Thread.currentThread().getName()+"-----消费了-----"+o+ "===="+res.size());
            Thread.sleep(10);
            proCondition.signal();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
// 生产者
class Producer2 implements Runnable{
    Resource2 resource;
    Producer2(Resource2 resource){
        this.resource = resource;
    }
    @Override
    public void run() {
        while (true){
            resource.set("面包");
        }
    }
}
// 消费者
class Consumer2 implements Runnable{
    Resource2 resource;
    Consumer2(Resource2 resource){
        this.resource = resource;
    }
    @Override
    public void run() {
        while (true){
            resource.out();
        }
    }
}
```
# 五、多线程细节问题
## 5.1 sleep() 和 wait()方法的区别

1. sleep() 方法必须指定时间；wait()方法可以不指定时间
1. sleep()方法的位置可以在同步中，也可以不在；wait()方法的位置必须在同步中
1. sleep()方法在睡眠时间到后会处于就绪或运行状态；wait()如果没有指定时间，必须通过notify() notifyAll()唤醒
1. 都定义在同步中，执行到sleep()方法不会是否锁，执行到wait()方法会是否锁
## 5.2 线程如何停止
早期使用线程对象的stop()方法来停止线程，但这一方法由于存在安全问题已经过时。给出了新的处理方式，可以在线程任务中定义标记，用来标识线程是否继续运行（一般作为线程任务中while循环的判断条件），该方法可以控制正常的线程结束。
如果当线程处于阻塞状态时，及时修改了标记，线程也不能结束。这时需要调用线程对象的interrupt()方法来中断线程的阻塞状态，使线程处于正常运行状态，该方法会抛出InterruptedException异常。因此可以在InterruptedException中修改循环标记使run()方法结束。
