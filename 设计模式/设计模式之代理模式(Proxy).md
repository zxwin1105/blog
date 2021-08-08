### 前言

代理模式(Proxy Pattern)是在程序设计中使用非常广泛。在学习框架时，可以发现有大量的实际应用，比如Mybatis中使用动态代理生成可执行的Mapper类、Spring中AOP技术的实现。在现实生活中也有非常多的例子，比如房屋中介代替顾客去租房子等等。所以学习并掌握代理模式是必要的。

### 一、代理模式

#### 1.1 代理模式介绍

在代理模式中一般包含的对象包括：代理类，被代理类，共同接口(代理类和被代理类共同实现的接口，确保它们由共同的功能)，客户端四个部分。

在代理模式中个对象之间的关系：

- 代理类和被代理类需要**实现共同的接口**
- **代理类持有被代理类的引用**，代理类在功能方法中调用被代理类的方法，可以对该方法进行增强/处理。
- 客户端调用代理类的方法(实际上代理类会去调用被代理类的方法)

![proxy](E:\Blog\image\design_proxyStruct.png)

#### 1.2 代理模式的分类

- 静态代理：由程序员在编码过程中就编写代理类，在运行时直接调用代理类的字节码文件。
- 动态代理：程序员不编写代理类，而是在程序运行阶段根据用户自定义规则来生成代理类对象

### 二、静态代理

#### 2.1 案例引入

生活中有许多代理的案例，这里采用工厂与代理商/销售商的例子。

例：有某品牌电脑生产工厂，有许多销售商代理并销售该品牌电脑。

上例中可以轻松的找出电脑生产工厂实际就是被代理对象(工厂的生产的电脑可以被代理销售)，销售商是代理对象，销售商代理的是电脑生产工厂的销售功能，所以需要定义一个它们共同实现的销售功能接口。这样就将上例中的实际生产关系抽象成代码。

#### 2.2 代码编写

- 创建共同接口，工厂和销售商共同功能是销售

```java
public interface Sell {
 	//销售功能
    void sell(double price);
}
```

- 创建工厂类，工厂类是被代理类

```java
public class PCFactory implements Sell{
    //工厂的销售功能
    @Override
    public void sell(double price) {
        System.out.println("PCFactory销售PC一台："+price+"元");
    }
    //除了销售功能工厂还有其他，如工厂的制造功能。
    public void make(){};
}
```

- 创建销售商类，销售类是代理类，代理并销售工厂生产的电脑

```java
public class SellProxy implements Sell{
    //代理对象需要持有被代理对象的引用，因为代理对象需要调用被代理对象的功能方法。
    private PCFactory pcFactory;
    //定义一个利润属性，表示代理商销售一台电脑可以获得的利润
    private double profit;
    public void setPcFactory(PCFactory pcFactory) {
        this.pcFactory = pcFactory;
    }

    public void setProfit(Double profit) {
        this.profit = profit;
    }
    @Override
    public void sell(double price) {
        System.out.println("SellProxy销售一台PC挣了"+profit+"元");
        pcFactory.sell(price-profit); //price-profit表示销售商卖出一台电脑，工厂拿到手的钱。
    }
}
```

- 客户端

```java
public class Client {
    public static void main(String[] args) {
        //实例化被代理对象
        PCFactory pcFactory = new PCFactory();
        //实例化代理对象
        SellProxy sellProxy = new SellProxy();
        sellProxy.setProfit(100.0);
        //将被代理对象的引用注入给代理对象
        sellProxy.setPcFactory(pcFactory);
        sellProxy.sell(5600);
    }
}
```

- 结果

> SellProxy销售一台PC挣了100.0元
> PCFactory销售PC一台：5500.0元

#### 2.3 静态代理总结

以上案例通过生活中的一个例子来帮助我们理解静态代理。

我们可以发现一些静态代理的特点：

- 静态代理会在编码阶段完成所有工作，包括代理类的编码工作

静态代理是面向实现编程(代理类和被代理类都需要实现Sell接口)而不是面向接口编程，是把程序写死了，不利于程序的扩展。所以就有了动态代理

### 三、动态代理

在程序运行阶段根据用户自定义规则来生成代理类对象，动态代理比静态代理的应用更广泛，在许多框架设计中都可以看见动态代理的身影。

#### 3.1 JDK动态代理方法

实现动态代理的方法有多种，这里我们学习JDK提供的方法

###### JDK动态代理的核心方法

- Proxy类的newProxyInstance()方法

```java
/**
 * 该方法用来生成代理类对象
 * @param loader 被代理类对象加载器
 * @param interfaces 被代理类对象实现的接口和继承的类的Class数组
 * @param h 用户自定义增强被代理类方法的接口
 */
public static Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h)throws IllegalArgumentException{
 	//code      
}
```

- InvocationHandler接口的invoke()方法

```java
/**
 * 用户自定义规则接口需要实现此接口，invoke()方法用于增强被代理对象方法
 * @param proxy 代理对象
 * @param method method.invoke()可以调用被代理对象的方法
 * @param args 方法的参数
 */
public Object invoke(Object proxy, Method method, Object[] args)throws Throwable;
```

#### 3.2 案例引入

开发完成后插入日志功能
例：现某系统开发完成，应后期要求在系统中插入日志功能。应该怎么做呢？精神大的同学已经开始修改代码了，逐行打印日志，显然这是一个要命的操作。
那么到底应该怎么实现呢，在经过几秒的头脑风暴后，灵光乍现。使用代理模式不就轻松解决问题了吗。代理模式可以在不改变原有功能代码的基础上对原有功能实现增强。

我们来完成一个具有计算功能的接口，并实现该接口。

#### 3.3 代码编写

- 计算功能接口

```java
public interface Arithmetic {
    int add(int a,int b); //加方法
    int sub(int a,int b); //减方法
    int mut(int a,int b); //乘方法
    int div(int a,int b); //除方法
}
```

- 计算功能接口的实现

```java
public class ArithmeticImpl implements Arithmetic{
    @Override
    public int add(int a, int b) {
        return a+b;
    }
    @Override
    public int sub(int a, int b) {
        return a-b;
    }
    @Override
    public int mut(int a, int b) {
        return a*b;
    }
    @Override
    public int div(int a, int b) {
        return a/b;
    }
}
```

到目前为止计算功能已经开发完成可以投入使用

- Client

```java
public class Client {
    public static void main(String[] args) {
        //目前我们完成了计算接口，并实现了计算接口，可以正常使用计算功能
        ArithmeticImpl arithmetic = new ArithmeticImpl();
        System.out.println(arithmetic.add(5, 6));
        System.out.println(arithmetic.mut(5, 6));
    }
}
```

> 11
> 30

在此完整的功能上我们要给各计算方法添加日志，我们来看具体操作

- 实现动态代理

这里是将代理对象的生成过程全部封装起来，我们在使用时只需要实例化该对象，将被代理对象注入，就可以通过getArithmeticProxy()方法获取代理后具有日志功能的对象。

```java
public class ArithmeticProxy{
    //持有要代理的对象
    private Arithmetic arithmetic ;
    public void setArithmetic(Arithmetic arithmetic) {
        this.arithmetic = arithmetic;
    }
    //提供一个获取代理后对象的方法
    public Arithmetic getArithmeticProxy(){
        //实现代理操作,增强原方法
        InvocationHandler h = new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("执行方法："+method.getName()); //打印日志
                Object result = method.invoke(arithmetic, args);  //调用被代理类的方法
                System.out.println("方法结果："+result);			  //打印日志
                return result;									  //返回方法的返回值
            }
        };
        Arithmetic proxy = (Arithmetic) Proxy.newProxyInstance(arithmetic.getClass().getClassLoader(),
                arithmetic.getClass().getInterfaces(),
                h);
        return proxy;
    }
}
```

- Client

来看看代理后的效果

```java
public class Client {
    public static void main(String[] args) {
        //目前我们完成了计算接口，并实现了计算接口，可以正常使用计算功能
        ArithmeticImpl arithmetic = new ArithmeticImpl();
        System.out.println(arithmetic.add(5, 6));
        System.out.println(arithmetic.mut(5, 6));
        //现在我们有新的需求，要求在执行计算方法时输出对应的日志
        ArithmeticProxy arithmeticProxy = new ArithmeticProxy();
        arithmeticProxy.setArithmetic(arithmetic);
        Arithmetic proxy = arithmeticProxy.getArithmeticProxy();
        proxy.add(9,6);
        proxy.div(8,3);
    }
}
```

> 11
> 30
> 执行方法：add
> 方法结果：15
> 执行方法：div
> 方法结果：2

可以看到日志争取的打印了出来；我们在不修改原有代码的基础上实现了对原有功能的增强，这就是动态代理的一个简单案例，希望能帮助大家更好的了解动态代理。

#### 3.4 动态代理再思考

看完上面的例子大家可能对动态代理还是有疑惑的地方，我们再来分析几个问题。

- 为什么调用Proxy.newProxyInstance()方法时可以强转为Arithmetic类型
- 为什么调proxy.add()会自动调用method.invoke()方法

大家可以参考[一步一步搞懂MyBatis中设计模式源码——代理模式](https://blog.csdn.net/shang_0122/article/details/106105717?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161743019516780274141527%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=161743019516780274141527&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~baidu_landing_v2~default-6-106105717.first_rank_v2_pc_rank_v29&utm_term=mybatis+%E4%BB%A3%E7%90%86%E6%A8%A1%E5%BC%8F)

### 参考

[一步一步搞懂MyBatis中设计模式源码——代理模式](https://blog.csdn.net/shang_0122/article/details/106105717?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161743019516780274141527%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=161743019516780274141527&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~baidu_landing_v2~default-6-106105717.first_rank_v2_pc_rank_v29&utm_term=mybatis+%E4%BB%A3%E7%90%86%E6%A8%A1%E5%BC%8F)

[利用代理模式实现日志功能](https://blog.csdn.net/qq_23937341/article/details/89165523)

[代理模式的使用总结](https://blog.csdn.net/xiaofeng10330111/article/details/105633821?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161735245116780266280121%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=161735245116780266280121&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-105633821.first_rank_v2_pc_rank_v29&utm_term=%E4%BB%A3%E7%90%86%E6%A8%A1%E5%BC%8F)