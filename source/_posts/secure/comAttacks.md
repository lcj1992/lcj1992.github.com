---
layout: post
title: 常见web攻击手段
date: 2015-12-08
categories: secure
---






　    


#### xss攻击(跨站脚本攻击) 

防范：

对用户输入的数据进行html转义

#### csrf攻击 
防范：

1.将cookie设置为httponly

2.增加token

3.通过referer识别

#### sql注入 
攻击原理：

    Class.forName("com.mysql.jdbc.Driver");
    con = DriverManager.getConnection( "jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8", "root", "root");
    int id = 1;
    String name = String.format("lcj' or 1=1");
    String sql = String.format("select * from account where id=%d and name='%s", id, name);
    Statement statement = con.createStatement();
    rs = statement.executeQuery(sql);

防范：

1.禁止使用Statement，使用PreparedStatement，

2.使用mybatis等ORM框架

3.避免密码明文存放

4.处理好相应的异常



