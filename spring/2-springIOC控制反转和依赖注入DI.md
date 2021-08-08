### 目录

控制反转Ioc

基于XML的Ioc环境搭建

spring对Bean的管理细节

spring中的依赖注入DI

### 一、控制反转Ioc

控制反转（Inversion of Control，缩写为**IoC**），面向对象编程中的一种设计原则/思想，可以用来减低计算机代码之间的耦合度。其中最常见的方式叫做依赖注入（Dependency Injection，简称**DI**），还有一种方式叫“依赖查找”（Dependency Lookup）。通过控制反转，对象在被创建的时候，由一个调控系统内所有对象的外界实体将其所依赖的对象的引用传递给它。也可以说，依赖被注入到对象中。

如上是百度百科对Ioc的介绍,很全面。spring的核心就是Ioc容器,通过Ioc容器可以实现对bean对象的管理。Ioc控制反转其实是一种思想/编程方式,这里所说的反转是指把对Bean的管理由程序员交给spring,我们不再需要new对象,而是需要配置bean对应的配置文件,在使用其对象时直接从spring容器中获取。

### 二、基于XML的Ioc环境搭建

导入依赖

```xml
<dependency>
	<groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.13.RELEASE</version>
</dependency>
```

创建spring配置文件applicationContext.xml并添加约束

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    https://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--将Bean对象交给spring管理-->
    <bean id="userMapper" class="com.zxwin.mapper.impl.UserMapperImpl"/>
    <bean id="userService" class="com.zxwin.service.imple.UserServiceImpl"/>
</beans>
```

测试

```java
public class Client {
    public static void main(String[] args) {
       //获取spring的核心容器对象
        ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");
        //通过核心容器获取Bean对象
        UserMapper userMapper = ac.getBean("userMapper", UserMapper.class);
        UserService userService = ac.getBean("userService", UserService.class);
        System.out.println(userMapper);
        System.out.println(userService);
    }
}
```

> com.zxwin.mapper.impl.UserMapperImpl@464bee09
> com.zxwin.service.imple.UserServiceImpl@f6c48ac

#### 核心容器接口ApplicationContext有三个实现类

- ClassPathXmlApplicationContext: 用来获取类路径下的配置文件
- FileSystemXmlApplicationContext: 用来获取磁盘中任意位置的配置文件
- AnnotationConfigApplicationContext: 用来读取注解创建容器的

#### 核心容器的两个接口的区别

- ApplicationContext: 它在创建容器时采用的策略时立即加载模式,一读完配置文件马上会创建对象.适用于单例对象
- BeanFactory: 创建容器时采用的策略是延迟加载模式,什么时候使用对象,什么时候创建对象,适用于多例对象

### 三、spring对Bean的管理细节

#### 三种创建Bean对象的方式

- 使用默认构造函数来创建对象

  在spring配置文件中使用<bean>标签,配置除了id和class之外没有其他属性和标签时使用默认构造函数创建对象

  ```xml
  <bean id="userMapper" class="com.zxwin.mapper.impl.UserMapperImpl"/>
  ```

- 使用普通工厂中的方法创建对象(使用某方法中的方法创建对象)

  ```xml
  <!--spring会先创建InstanceFactory的对象,通过getService()方法来创建UserService对象-->
  <bean id="instanceFactory" class="com.zxwin.Factory.InstanceFactory"/>
  <bean id="userService" factory-bean="instanceFactory" factory-method="getService"/>
  ```

- 使用工厂中的静态方法创建对象(使用某方法中的静态方法创建对象)

  ```xml
  <bean id="userService" class="com.zxwin.Factory.InstanceFactory" factory-method="getService"/>
  ```

#### Bean的作用范围

bean的作用范围通过在<bean>标签的scope属性上配置

scope属性的取值

- singleton: 单例模式(默认)
- prototype: 多例模式
- request: 作用于web应用的请求范围
- session: 作用于web应用的会话范围
- global-session: 作用于集群环境的会话范围

#### Bean的生命周期

bean的生命周期与其是单例模式还是多例模式有关

- 单例模式  生命周期和容器一致
  - 出生: 在容器创建时对象出生
  - 死亡: 容器销毁时对象消亡
- 多例模式 只要对象一直使用对象就活着
  - 出生: 在使用对象时有容器创建
  - 死亡: 该对象没有被引用时有JVM回收

### 四、spring中的依赖注入DI

Ioc的作用是降低程序间的依赖关系,我们将程序的依赖关系交给spring管理。如果当前类中需要依赖其他类都由spring为我们提供,我们只需要在配置文件中说明,这种依赖关系的维护叫依赖注入(Dependenty Injection)。**Ioc的实现需要依赖DI**

对于DI可以通俗的理解为是spring创建对象时给成员赋值的操作

#### 依赖注入的三种方法

###### 使用构造函数注入

在<bean> 标签内部使用<constructor-tag>标签

<constructor-tag>标签的属性值包括

- type:指定要注入的数据类型,spring根据数据类型匹配对应的形参 
- index: 指定要注入参数的索引,索引从0开始
- name: 指定要注入参数的参数名
- value: 用于提供基本类型和String类型的值
- ref: 指定其他Bean对象

```xml
<bean id="diTest" class="com.zxwin.DiTest">
	<constructor-arg name="name" value="zxwin"></constructor-arg>
    <constructor-arg name="age" value="22"></constructor-arg>
    <constructor-arg name="birthday" ref="birthday"></constructor-arg>
</bean>
<bean id="birthday" class="java.util.Date"></bean>
```

```java
public class DiTest {
    private String name;
    private Integer age;
    private Date birthday;
    public DiTest(String name, Integer age, Date birthday) {
        this.name = name;
        this.age = age;
        this.birthday = birthday;
    }
}
```

> 注意:一般要注入的数据是不轻易改变的数据,如:service层中调用dao层可以注入。上面的例子仅用于演示

使用构造函数注入的优点:在获取对象时,注入数据是必须的操作,否则对象无法创建成功

弊端: 改变了bean对象的实例化方法,即使用不到这些数据,也需要提供

###### 使用set方法注入

在<bean> 标签内部使用<property>标签

<property>标签的属性

- name: 用于指定注入时所调用的set方法名称
- value: 用于提供基本类型和String类型的值
- ref: 指定其他Bean对象

```xml
<bean id="diTest" class="com.zxwin.DiTest">
    <property name="name" value="zxwin"/>
	<property name="age" value="22"/>
	<property name="birthday" ref="birthday"/>
</bean>
<bean id="birthday" class="java.util.Date"></bean>
```

```java
public class DiTest {
    private String name;
    private Integer age;
    private Date birthday;
    //注意这里必须由set方法,因为<property>标签的name属性调用的是set方法
    public void setName(String name) {
        this.name = name;
    }
    public void setAge(Integer age) {
        this.age = age;
    }
    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }
}
```

使用set方法注入的优势:创建对象时没有限制,可以直接使用默认构造函数创建对象

弊端:如果某个成员必须有值,我们没有调用set方法注入导致无法执行

###### 使用注解注入

在注解章节说明

#### 依赖注入支持的数据类型

- 基本类型和String
- 其他由spring管理的bean类型
- 复杂类型/集合类型

```java
public class DiTest {
    private String[] array;
    private List<String> list;
    private Map<String,String> map;
    private Set<String> set;
    private Properties props;

    public void setArray(String[] array) {
        this.array = array;
    }

    public void setList(List<String> list) {
        this.list = list;
    }

    public void setMap(Map<String, String> map) {
        this.map = map;
    }

    public void setSet(Set<String> set) {
        this.set = set;
    }

    public void setProps(Properties props) {
        this.props = props;
    }
}
```

```xml
<bean id="diTest" class="com.zxwin.DiTest">
        <!--注入数组-->
        <property name="array">
            <array>
                <value>aaa</value>
                <value>bbb</value>
                <value>ccc</value>
            </array>
        </property>
        <!--注入list -->
        <property name="list">
            <list>
                <value>aa</value>
                <value>bb</value>
                <value>cc</value>
            </list>
        </property>
        <!--注入map -->
        <property name="map">
            <map>
                <entry key="map1" value="aaa"/>
                <entry key="map2" value="bbb"/>
                <entry key="map3" value="ccc"/>
            </map>
        </property>
        <!--注入set -->
        <property name="set">
            <set>
                <value>a</value>
                <value>b</value>
                <value>c</value>
            </set>
        </property>
        <!--注入property -->
        <property name="props">
            <props>
                <prop key="prop1">a</prop>
                <prop key="prop2">b</prop>
                <prop key="prop3">c</prop>
            </props>
        </property>
</bean>
```

> 注意点:
>
> 用于给list结构集合注入的标签<list> <set> <array>
>
> 用于给map结构集合注入的标签<map> <props>
>
> 同归结构间标签可以互换使用
