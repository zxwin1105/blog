### 一、方法间的依赖关系

我们在spring概述中曾经说过，程序中的耦合一般有类之间的耦合，和方法之间的耦合。我们通过Spring提供的Ioc容器降低了类之间的依赖关系。今天我们来了解一下方法之间的依赖，我们通过一个模拟业务代码案例来分析程序中的问题。程序如下

持久层接口

```java
public interface AccountMapper {
    //查询所有方法
    List<Account> selectAll();
    //通过id产找账户
    Account selectById(Integer id);
    //更新账户
    int update(Account account);
}
```

业务层实现代码

```java
package com.zxwin.service.impl;

@Service("accountService")
public class AccountServiceImplOld implements AccountService {
    private AccountMapper accountMapper;
    //封住了对事务管理的工具类，在这里不做重点(只需要知道提供了事务的操作)，不给出实现类
    private TransactionManage transactionManage;
    @Autowired
    public void setTransactionManage(TransactionManage transactionManage) {
        this.transactionManage = transactionManage;
    }
    @Autowired
    public void setAccountMapper(AccountMapper accountMapper) {
        this.accountMapper = accountMapper;
    }
    
    @Override
    public List<Account> findAllAccount() {
        List<Account> accounts = null;
        //开启事务
        transactionManage.beginTransaction();
        try {
            accounts = accountMapper.selectAll();
            //提交事务
            transactionManage.commit();
        } catch (SQLException e) {
            e.printStackTrace();
            //出现异常时回滚
            transactionManage.rollback();
        }finally {
            //释放资源的操作
            transactionManage.release();
        }
        return accounts;
    }
    //转账操作
    @Override
    public void transfer(Integer sourceId, Integer targetId, double money) {
        transactionManage.beginTransaction();
        try {
            Account source = accountMapper.selectById(sourceId);
            Account target = accountMapper.selectById(targetId);
            source.setMoney(source.getMoney()-money);
            target.setMoney(target.getMoney()+money);
            accountMapper.update(source);
            accountMapper.update(target);
            transactionManage.commit();
        } catch (SQLException e) {
            e.printStackTrace();
            transactionManage.rollback();
        }finally {
            transactionManage.release();
        }
    }
}
```

通过上例我们可以发现业务代码中每个方法都要通过TransactionManage类中的发给发实现对事务的控制，造成了代码大量冗余，降低了我们的开发效率，更糟的情况是如果TransactionManage类中的方法被修改(如参数)我们就需要对整个业务层中的代码进行修改。这就是存在方法间的依赖。

如何降低这种依赖呢，Spring给出了我们解决的方案AOP (面向切面编程)。在了解AOP之前，我们可以先通过我们已经掌握的技术实现对上例代码的改造。

上例中业务中存在的问题是有大量的事务控制代码重复且事务控制代码固定不变。我们可以通过动态代理技术实现对业务类的增强，这样业务类只需要完成正常的业务操作，事务的操作则通过代理类增强。我们看一下使用动态代理后的程序。

对动态代理实现的封装类

```java
@Component
public class ServiceAop {
    //代理类需要持有被代理类的引用
    private AccountService accountService;
    private TransactionManage transactionManage;
    @Autowired
    public void setAccountService(AccountService accountService) {
        this.accountService = accountService;
    }
    @Autowired
    public void setTransactionManage(TransactionManage transactionManage) {
        this.transactionManage = transactionManage;
    }

    //获取增强后的业务代理类
    @Bean("accountServiceProxy")
    public AccountService getAccountService(){
        //这里采用第三放工具类，实现通过子类的动态代理方法
        return (AccountService) Enhancer.create(accountService.getClass(),
                new MethodInterceptor() {
                    @Override
                    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
                        Object result = null;
                        //增强功能：开启事务
                        transactionManage.beginTransaction();
                        try {
                            //执行被代理的方法
                            result = method.invoke(accountService,args);
                            //增强功能：提交事务
                            transactionManage.commit();
                        } catch (Exception e) {
                            e.printStackTrace();
                            //增强功能：回滚事务
                            transactionManage.rollback();
                        }finally {
                            //增强功能：释放资源
                            transactionManage.release();
                        }
                        return result;
                    }
                });
    }
}
```

业务层代码只需要实现业务功能

```java
@Service("accountService")
public class AccountServiceImpl implements AccountService {
    private AccountMapper accountMapper;
    private TransactionManage transactionManage;
    @Autowired
    public void setTransactionManage(TransactionManage transactionManage) {
        this.transactionManage = transactionManage;
    }
    @Autowired
    public void setAccountMapper(AccountMapper accountMapper) {
        this.accountMapper = accountMapper;
    }

    @Override
    public List<Account> findAllAccount() {
        List<Account> accounts = null;
        accountMapper.selectAll();
        return accounts;
    }
    @Override
    public void transfer(Integer sourceId, Integer targetId, double money) {
        Account source = null;
        Account target = null;
        source = accountMapper.selectById(sourceId);
        target = accountMapper.selectById(targetId);
        source.setMoney(source.getMoney()-money);
        target.setMoney(target.getMoney()+money);
        accountMapper.update(source);
        accountMapper.update(target);
        transactionManage.commit();
    }
}
```

这样我们就通过动态代理将业务功能和事务功能分离，即使事务功能代码发生变化我们只需要在增强功能一处进行修改。

这就是解决方法依赖的办法，上例对我们理解spring中的AOP技术有很大帮助。

### 二、AOP概念

AOP(Aspect Oriented Programming)面向切面编程，是通过预编译和运行期动态代理实现程序统一维护的一种技术，利用AOP可以对逻辑业务的各个部分进行分离，从而使逻辑业务各部分间耦合度降低，提高程序的可重用性，提高开发效率。

AOP的作用在程序运行期间不修改源码实现对已有功能的增强；

AOP的优势有减少重复代码；提高开发效率；维护方便

AOP的实现依赖动态代理，所有我们要先对动态代理有一些了解，才能更好的学习AOP

### 三、AOP中的术语

JoinPoint(连接点)：连接点是指那些被拦截到的点，在spring中这些点是指方法(spring只支持方法类型的连接点)。也就是被代理对象中的所有方法

Pointcut(切入点)：切入点是指我们要对哪些JoinPoint进行拦截的定义。也就是需要被增强的方法

Advice(通知/增强)：通知是指被拦截到JoinPoint之后要做的事情就是通知。通知的类型：前置通知，后置通知，异常通知，最终通知，环绕通知

Introduction(引介)：使一种特殊的通知在不修改代码的前提下，Introduction可以在运行期为类动态的添加一些方法或字段 

Target(目标对象)：代理的目标对象

Proxy(代理对象)：一个类被AOP织入增强后，产生的结果代理类

Weaving(织入)：是指把增强应用到目标对象来创建新代理对象的过程

Aspect(切面)：是切入点和通知(引介)的结合

### 四、通知的分类

通知其实就是对被代理对象方法增强的操作

上面术语中提到了通知，主要有5中通知类型，每种通知类型是什么意思呢？我们先来看前四种。

我们知道AOP是通过动态代理技术在程序运行期间在不修改程序的情况下实现对已有功能的增强。我们来看一下例子

```java
//这段代码是动态代理中实现增强功能的
InvocationHandler h = new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                Object result = null;
                try{
                    System.out.println("前置通知");
                    //执行切入点方法
                    result = method.invoke(arithmetic, args);
                    System.out.println("后置通知");
                }catch (Exception e){
                    e.printStackTrace();
                    System.out.println("异常通知");
                }finally {
                    System.out.println("最终通知");
                }
                return result;
            }
        };
```

上例中明确的给出了四种通知的位置

前置通知就是在切入点方法前执行的操作；后置通知是在切入点方法正常执行之后执行的操作；异常通知是在切入点方法发生异常后执行的操作；最终通知是无论切入点方法是否正确执行都会执行。

环绕通知是spring为我们提供的一种可以在代码中手动控制通知何时执行的方式。后边会具体了解。

### 五、学习springAOP要明确的事

#### 开发阶段

由程序员编写核心业务代码

将公用代码抽取出来，制作成通知(开发阶段最后做)

在配置文件中，声明切入点与通知之间的关系即切面

#### 运行阶段

spring框架监控切入点方法的执行，一旦监控到切入点方法被执行，使用代理机制动态创建目标对象的代理对象，根据通知类别在代理对象对应位置将通知对应的功能织入，完成完整代码逻辑

### 六、AOP案例

我们模拟平时开发中打印日志的需求，先要求对Serivce层功的方法通过AOP实现日志打印功能

需要导入的依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>org.example</groupId>
    <artifactId>SpringTest</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.13.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.1.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13</version>
            <scope>test</scope>
        </dependency>
        <!--用来解析切入点表达式-->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.4</version>
        </dependency>
    </dependencies>
</project>
```

我们先来看一下业务层代码和自定义的通知类(类用的方法用于对业务层中的方法实现增强)

```java
package com.zxwin.service.impl;
//模拟业务层代码
public class UserServiceImpl implements UserService {
    @Override
    public int updateUser() {
        System.out.println("执行了update方法");
        return 0;
    }
    @Override
    public void save() {
        System.out.println("执行了save方法");
    }
    @Override
    public void delete(int i) {
        System.out.println("执行了delete方法");
    }
}
```

```java
package com.zxwin.advice;
//定义通知类
public class Logger {
    //通知的方法
    public void printLog(){
        System.out.println("Logger 开始打印日志了");
    }
}
```

我们在来看AOP具体的配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop
        https://www.springframework.org/schema/aop/spring-aop.xsd">
    <!--spring配置AOP,需要导入AOP的约束-->
    <bean id="userService" class="com.zxwin.service.imple.UserServiceImpl"/>
    <!--将通知类交给spring管理-->
    <bean id="logger" class="com.zxwin.advice.Logger"/>
    <!--配置AOP
        1-使用标签<aop:config>开始配置aop
        2-在标签<aop:config>下使用标签<aop:aspect>开始配置切面
            id属性用来唯一标识切面
            ref属性用来执行通知bean
        3-在标签<aop:aspect>下使用标签<aop:before>等不同通知类型的标签开始配置通知
            method属性用来标识当前类型的通知要执行通知类中的那个方法
            pointcut属性用来标识切入点(业务类中哪些方法需要被该通知增强)
                pointcut的值需要一个切入点表达式 格式execution(表达式)
                表达式格式：访问修饰符 方法返回值 包名.包名.包名.类名.方法名(参数)
    -->
    <aop:config>
        <aop:aspect id="loggerAdvice" ref="logger">
            <aop:before method="printLog" pointcut="execution( public void com.zxwin.service.imple.UserServiceImpl.save())"/>
        </aop:aspect>
    </aop:config>
</beans>

```

测试类

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:applicationContext.xml"})
public class TestAOP {
    @Autowired
    private UserService userService;
    @Test
    public void testLoggerAdvice(){
        userService.save();
    }
}
```

执行结果

>Logger 开始打印日志了
>执行了save方法

可以看到在save()方法执行前,先执行了打印日志的方法,说明我们的aop配置正确。

上面配置中关于切入点表达式我们还可以通过通配符的方式来书写，规则如下

- 访问修饰符可以省略，上面的表达式等价于`execution( void com.zxwin.service.imple.UserServiceImpl.save())`

- 方法的返回值可以使用通配符表示任意返回值类型，又可将表达式修改为`execution( * com.zxwin.service.imple.UserServiceImpl.save())`
- 包名可以使用通配符表示任意包，注意有几级包就要写几个*.，表达式修改为

```
 * *.*.*.*.UserServiceImpl.save()
```

​		包名还可使用..表示任意包及子包，又可改写为

```
 * *..UserServiceImpl.save()
```

- 类名和方法名也都可以使用通配符表示

```
 * *..*.*()
```

​		这种写法会匹配到UserServiceImpl类中的两个方法updateUser()和save()方法。

- 参数列表

  - 可以直接写类型名

    - 基本类型可以直接写类型名称
    - 引用类型需要写全类名

  - 也可以使用通配符*表示任意类型

    ```
    * *..UserServiceImpl.*(*) //会匹配到delete(int i)方法
    ```

  - 可以使用..表示有无参数均可

    ```
    * *..UserServiceImpl.*(*) //3个方法都会被匹配到
    ```

> 通过上面表达式写法的学习，我们可以写出表达式 `* *..*.*(..)`该表达式是全通配符，会匹配当前项目的所有方法，不建议使用。
>
> 一把我们的使用原则是匹配到业务层实现类下的所有方法`* com.zxwin.service.impl.*.*(..)`

### 七、配置其他类型通知

上述AOP案例中我们只配置实现了前置通知，我们知道SpringAOP中的通知类型有5中分别是前置通知，后置通知，异常通知，最终通知，环绕通知。我们接下来在上例的基础上完成其他通知

通知类修改

```java
public class Logger {
    //前置通知
    public void beforeLog(){
        System.out.println("前置通知beforeLog");
    }
    //后置通知
    public void afterReturningLog(){
        System.out.println("后置通知afterReturningLog");
    }
    //异常通知
    public void afterThrowingLog(){
        System.out.println("异常通知afterThrowingLog");
    }
    //最终通知
    public void afterLog(){
        System.out.println("最终通知afterLog");
    }
}
```

修改切面配置

```xml
<aop:config>
	<aop:aspect id="loggerAdvice" ref="logger">
            <!--配置切入点表达式,该标签放在<aop:aspect>标签中只能当前切面调用。
            该标签也可以配置在<aop:config>标签下，可以供所有切面使用-->
            <aop:pointcut id="servicePointcut" expression="execution( * com.zxwin.service.impl.*.*(..))"/>
            <!--前置通知-->
            <aop:before method="beforeLog" pointcut-ref="servicePointcut"/>
            <!--后置通知-->
            <aop:after-returning method="afterReturningLog" pointcut-ref="servicePointcut"/>
            <!--异常通知-->
            <aop:after-throwing method="afterThrowingLog" pointcut-ref="servicePointcut"/>
            <!--最终通知-->
            <aop:after method="afterLog" pointcut-ref="servicePointcut"/>
    </aop:aspect>
</aop:config>
```

测试类

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(locations = {"classpath:applicationContext.xml"})
public class TestAOP {
    @Autowired
    private UserService userService;

    @Test
    public void testLoggerAdvice() {
        userService.save();
    }
}
```

> 前置通知beforeLog
> 执行了save方法
> 后置通知afterReturningLog
> 最终通知afterLog

### 八、环绕通知

spring中的环绕通知是spring为我们提供的一种可以在代码中手动控制通知何时执行的方式。

我们来看一下spring中关于环绕通知的配置

```xml
<aop:config>
        <aop:aspect id="loggerAdvice" ref="logger">
            <!--配置切入点表达式,该标签放在<aop:aspect>标签中只能当前切面调用。
            该标签也可以配置在<aop:config>标签下，可以供所有切面使用-->
            <aop:pointcut id="servicePointcut" expression="execution( * com.zxwin.service.impl.*.*(..))"/>
            <!--环绕通知-->
            <aop:around method="aroundLog" pointcut-ref="servicePointcut"/>
        </aop:aspect>
</aop:config>
```

上面我们提到环绕通知实际是spring为我们提供的在代码中手动控制通知执行的方法

```java
public class Logger {
    //前置通知
    public void beforeLog(){
        System.out.println("前置通知beforeLog");
    }
    //后置通知
    public void afterReturningLog(){
        System.out.println("后置通知afterReturningLog");
    }
    //异常通知
    public void afterThrowingLog(){
        System.out.println("异常通知afterThrowingLog");
    }
    //最终通知
    public void afterLog(){
        System.out.println("最终通知afterLog");
    }
    /*环绕通知
    	spring为我们提供了ProceedingJoinPoint接口，调用接口的proceed()方法，此方法就相当明确调用切入点方法。再围绕proceed()方法完成环绕通知
     */
    public void aroundLog(ProceedingJoinPoint pjp){
        try {
            beforeLog();
            pjp.proceed(pjp.getArgs());
            afterReturningLog();
        } catch (Throwable throwable) {
            afterThrowingLog();
            throwable.printStackTrace();
        }finally {
            afterLog();
        }
    }
}
```

### 九、AOP的注解配置

```xml
<!--需要再xml配置文件中开启注解的支持，如果使用的纯注解配置只需要再配置类上添加@EnableAspectJAutoProxy注解-->
<aop:aspectj-autoproxy/>
```

```java
@Aspect //标识该类是一个切面类
@Component("logger")
public class Logger {
    //切面表达式
    @Pointcut("execution(* com.zxwin.service.impl.*.*(..))")
    public void pointcut(){}
    
    //前置通知
    @Before("pointcut()")
    public void beforeLog(){
        System.out.println("前置通知beforeLog");
    }
    //后置通知
    @AfterReturning("pointcut()")
    public void afterReturningLog(){
        System.out.println("后置通知afterReturningLog");
    }
    //异常通知
    @AfterThrowing("pointcut()")
    public void afterThrowingLog(){
        System.out.println("异常通知afterThrowingLog");
    }
    //最终通知
    @After("pointcut()")
    public void afterLog(){
        System.out.println("最终通知afterLog");
    }
    //@Around("pointcut()") 环绕通知和其他四个通知完成的功能一样，这里注释掉
    public void aroundLog(ProceedingJoinPoint pjp){
        try {
            beforeLog();
            pjp.proceed(pjp.getArgs());
            afterReturningLog();
        } catch (Throwable throwable) {
            afterThrowingLog();
            throwable.printStackTrace();
        }finally {
            afterLog();
        }
    }
}
```

> 注意点：在使用注解配置时配置其他四种通知可能会出现调用顺序的问题。使用注解配置的环绕通知则可以正常使用。

