### 一、 配置声明式事务所需的依赖

```xml
<dependency>
	<groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.2.13.RELEASE</version>
</dependency>
<!--spring的事务支持-->>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-tx</artifactId>
    <version>5.2.13.RELEASE</version>
<!--解析切入点表达式-->>
</dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.4</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.2.13.RELEASE</version>
</dependency>
```

### 二、配置

1. 配置事务管理器

2. 配置事务的通知

   - 添加事务的约束(tx名称空间和约束)和aop约束
   - 配置事务的通知属性

   ```
   isolation:指定事务的隔离级别，默认值是DEFAULT表示是数据库默认的隔离级别
   propagation:指定事务的传播级别,默认值是REQUIRED,表示一定会有事务(增删改的选择)。查询可以选择SUPPORTS
   read-only:指定事务是否为只读，只有查询方法才设置为true，默认值是false表示读写
   timeout:指定事务的超时时间，默认值-1表示永不超时。单位s
   rollback-for:指定一个异常，当产生该异常时事务回滚，产生其他异常时事务不回滚。没有默认值表示任何异常都回滚
   no-rollback_for:指定一个异常，当产生该异常时事务不回滚，产生其他异常时事务回滚。没有默认值表示任何异常后回滚
   ```

3. 配置AOP中通用的切入点表达式

4. 管理事务的通知和切入点

```xml
<!--配置事务管理器-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager"/>
<!--配置事务的通知-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
	<tx:attributes>
		<!--*匹配所有-->
        <tx:method name="*" propagation="REQUIRED" read-only="false"/>
        <tx:method name="select*" propagation="SUPPORTS" read-only="true"/>
    </tx:attributes>
</tx:advice>
<!--配置AOP中通用的切入点表达式-->
<aop:config>
	<aop:pointcut id="pt1" expression="execution(* com.zxwin.service.impl.*.*(..))"/>
    <!--管理事务的通知和切入点-->
	<aop:advisor advice-ref="txAdvice" pointcut-ref="pt1"/>
</aop:config>
```

### 三、使用注解配置

```xml
<!--注解配置声明式事务-->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"/>
</bean>
<!--配置事务管理器-->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource"/>
</bean>
<!--开启事务注解支持-->
<tx:annotation-driven/>
```

在业务类

```java
@Service("accountService")
@Transactional //开启事务
public class AccountServiceImpl implements AccountService {
    private AccountMapper accountMapper;
    @Autowired
    public void setAccountMapper(AccountMapper accountMapper) {
        this.accountMapper = accountMapper;
}
```

### 四、使用纯注解配置

主配置类

```java
@Configuration
@ComponentScan("com.zxwin")
@PropertySource("classpath:dataSource.properties")
@Import({JdbcConfiguration.class,TransactionConfiguration.class})
@EnableTransactionManagement
public class SpringConfiguration {
}
```

jdbc配置类

```java
public class JdbcConfiguration {
    @Value("${jdbc.driverClass}")
    private String driverClass;
    @Value("${jdbc.url}")
    private String url;
    @Value("${jdbc.user}")
    private String username;
    @Value("${jdbc.password}")
    private String password;

    @Bean(name="jdbcTemplate")
    public JdbcTemplate createJdbcTemplate(DataSource dataSource){
        return new JdbcTemplate(dataSource);
    }

    @Bean(name="dataSource")
    public DataSource createDataSource(){
        ComboPooledDataSource dataSource = new ComboPooledDataSource();
        try {
            dataSource.setDriverClass(driverClass);
            dataSource.setJdbcUrl(url);
            dataSource.setUser(username);
            dataSource.setPassword(password);
        } catch (PropertyVetoException e) {
            e.printStackTrace();
        }
        return dataSource;
    }
}
```

事务配置类

```java
public class TransactionConfiguration {
    @Bean("transactionManager")
    public PlatformTransactionManager createTransactionManager(DataSource dataSource){
        return new DataSourceTransactionManager(dataSource);
    }
}
```

