### JDBC概念：

- Java DataBase Connectivity ，java语言操作库

  **JDBC本质：是sun公司定义了一套操作所有关系型数据库的规则，即接口。各数据库厂商去实现这套接口提供数据库驱动jar包。我们可以使用这套接口变成，真正的代码是驱动jar包中的实现类。**

### JDBC快速入门

```java
//1. 导入驱动jar包
	//1-创建libs文件夹，导入jar包
	//2-右键点击 add as library
//2. 注册驱动
Class.forName("com.mysql.jdbc.Driver");
//3. 获取数据库链接对象
Connection conn=DriverManager.getConnection("jdbc:mysql://localhost:3306/Test","root","root");
//4. 定义sql语句
String sql = "update student set score = 99.5 where id = '002'";
//5. 获取执行sql的对象 Statement
Statement stmt = conn.createStatement();
//6. 执行sql
int count = stmt.executeUpdate(sql);
//7. 处理结果
System.out.println(count);
//8. 释放资源
stmt.close();
conn.close();
```

