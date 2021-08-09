Spring框架对JDBC的简单封装，提供了一个JDBCTemplate

- 使用步骤

  1. 导入jar包

  2. 创建JdbcTemplate对象，依赖于数据源DataSource

     `JdbcTemplate template = new JdbcTemplate(ds);`

  3. 调用JdbcTemplate的方法来完成CRUD的操作

     - update():执行DMl语句，
     - queryForMap():查询结果将结果封装为map集合；查询的结果集长度为1
     - queryForList():查询结果将结果封装为List集合；将每条结果封装成一个map集合 再将所有的map集合封装到list集合中
     - query():查询结果将结果封装为JavaBean对象
     - queryForObject():查询结果，将结果封装为对象；一般用于聚合函数的查询