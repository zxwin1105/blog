### 目录

[一、spring介绍](#spring介绍)

[二、程序间的耦合](#程序间的耦合)

### <a id="spring介绍">一、spring介绍</a>

#### 什么是spring?

Spring框架是一个开源的J2EE应用程序框架,是针对bean的生命周期进行管理的轻量级容器，Spring解决了开发者在J2EE开发中遇到的许多常见的问题，提供了功能强大IOC、AOP及Web MVC等功能。

![spring体系结构图](E:\Blog\image\spring的体系结构.png)

#### 名词介绍

Bean：计算机英语用表示可重用的组件

javaBean：java语言编写的可重用组件，如Serivce,Dao

IOC：控制反转(Inversion of control)

AOP：面向切面编程(Aspext Oriented Programming)

#### spring的主要优势

- 通过spring提供的核心容器，可以将对象间的依赖关系交给spring控制。	
- AOP编程的支持
- 声明式事务的支持：通过声明的方式灵活的进行事务管理

### 二、程序间的耦合

在学习spring之前，需要先了解一下程序间的耦合。等我们了解了程序间的耦合，对理解sping的核心容器有很大的帮助。

耦合也就是程序间的依赖关系，程序间的依赖关系包括：类之间的依赖，方法之间的依赖关系。

我们先来看类之间的依赖关系。

```java
//Service层
public class UserServiceImpl implements UserService {
    private UserMapper userMapper = new UserMapperImpl();
    @Override
    public void login(String username, String password) {
        System.out.println("Service : login");
        userMapper.selectUser(username,password);
    }
}
//Mapper层
public class UserMapperImpl implements UserMapper {
    @Override
    public void selectUser(String username, String password) {
        System.out.println("Mapper : selectUser");
    }
}
```

我们可以看到上面的代码中UserServiceImpl依赖于UserMapperImpl，在UserServiceImpl中需要用new关键字来实例化UserMapperImpl。这样做有很多的局限性，如：UserMapperImpl出错或不存在时程序会直接出错。

一般解决类之间依赖关系需要做到**编译期不依赖，运行期依赖**，为了达到这一目的我们应该避免使用new关键字。可是不用new关键字怎么实例化对象呢，我们知道除了使用new关键字实例化对象，还可以使用反射的技术来实例化对象。

###### 两者间有什么不同呢？可以解决依赖关系吗？

我们回想在使用反射技术来实例化对象时，我们只需要提供一个全类名，程序会在运行期间动态的实例化一个对象，如果该全类名指向的类不存在时，会发生运行时异常。显然使用反射技术实例化对象可以达到**编译期不依赖，运行期依赖**的效果。所以new和反射的不同在于，new关键字在编译时期就依赖一个具体的类，而反射在编译时期只依赖于一个全限定类名的字符串，在运行期间才会依赖具体的类。

###### 使用反射创建对象不会存在问题吗？

```java
Class.forName("com.mysql.jdbc.Driver");
```

上述是使用mysql时注册驱动时代码，使用的就是反射技术。我们想如果想使用另外一种数据库就需要来修改代码，这显然是不合适的。我们可以通过配置文件的方式来配置需要用到的全类名。

综上我们可以总结出解除依赖关系的思路

- 使用反射技术创建对象，避免使用new关键字
- 提供配置文件，获取要创建对象的全限定类名

我们可以将上面的代码修改为

```java
public class UserMapperImpl implements UserMapper {
    @Override
    public void selectUser(String username, String password) {
        System.out.println("Mapper : selectUser");
    }
}

public class UserServiceImpl implements UserService {
    private UserMapper userMapper =(UserMapper) BeanFactory.getBean("userMapper");
    @Override
    public void login(String username, String password) {
        System.out.println("Service : login");
        userMapper.selectUser(username,password);
    }
}
```

这里的`BeanFactory.getBean("userMapper")`,BeanFactory是一个用于生产Bean对象的工厂类。

```java
public class BeanFactory {
    //获取bean.properties中的内容
    private static Properties props;
    //用来存储实例化后的对象,在初始化时将所有的配置中所有的bean实例化并存储可以实现单例效果
    private static Map<String,Object> beanMap;
    static {
        //获取properties文件流
        InputStream rs = BeanFactory.class.getClassLoader().getResourceAsStream("bean.properties");
        props = new Properties();
        try {
            props.load(rs);
            //实例化Bean 并将其添加到beanMap容器中
            beanMap = new HashMap<>();
            //获取配置文件中所有的key
            Enumeration<Object> keys = props.keys();
            while(keys.hasMoreElements()){
                String key = keys.nextElement().toString();
                //获取key对应的全限定类名
                String beanPath = props.getProperty(key);
                //实例化并添加到容器中
                beanMap.put(key,Class.forName(beanPath).newInstance());
            }
        } catch (Exception e) {
            throw new ExceptionInInitializerError("beanFactory初始化错误");
        }
    }

    //用于获取Bean对象的方法
    public static Object getBean(String key) {
        return beanMap.get(key);
    }
}
```

该工厂类工作流程：先读取bean的配置文件，获取配置中所有的key，使用key获取对应的全限定类名，使用反射技术实例化该对象并将其添加到map容器中。

```properties
userMapper=com.zxwin.mapper.impl.UserMapperImpl
userService=com.zxwin.service.imple.UserServiceImpl
```

我们通过两张图来看一下上面两种实现方式的区别

- 使用new关键字创建对象

![newBean](E:\Blog\image\newBean.png)

- 使用factory来获取对象

![factoryBean](E:\Blog\image\factoryBean.png)

实际上spring的IOC容器就是通过配置文件+反射来实现对Bean对象的管理，我们可以把上面的例子看作是mini版容器

