---
layout: post
title: mysql相关配置
categories: db
tags: workbeanch
---


### mac上mysql 安装

    lcj:tool fool$ brew install mysql
    ==> Downloading https://homebrew.bintray.com/bottles/mysql-5.7.10.el_capitan.bottle.3.tar.gz
    Already downloaded: /Library/Caches/Homebrew/mysql-5.7.10.el_capitan.bottle.3.tar.gz
    ==> Pouring mysql-5.7.10.el_capitan.bottle.3.tar.gz
     ==> Caveats
    We've installed your MySQL database without a root password. To secure it run:
        mysql_secure_installation

    To connect run:
        mysql -uroot

    To have launchd start mysql at login:
      ln -sfv /usr/local/opt/mysql/*.plist ~/Library/LaunchAgents
    Then to load mysql now:
      launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist
    Or, if you don't want/need launchctl, you can just run:
      mysql.server start
    ==> Summary
    🍺  /usr/local/Cellar/mysql/5.7.10: 12,677 files, 433.2M

 从现有数据库中导出ER图和建表语句
1. ER图： 打开workbench  连接上数据库，然后 Database---Reverse  engineer (或者直接快捷键 Ctrl + R)

2.建表语句：在看ER图时，然后File--Export--Forward Engineering SQL_CREATE Script.(或者直接快捷键Ctrl+Shift+G)

3.ER图另一个入口

![workbeanch](/images/tool/workbeanch.png)

点击Models右边第三个按钮 > 就会有这两种方法。

4.建表语句另一个入口

Management---Data Export

命令：mysqldump -u用戶名 -p密码 -d 数据库名 表名 > 脚本名;

### mysql 允许远程连接

ubuntu从源中安装的mysql默认是不开始远程连接的。（mysql 5.6）

网上有的说直接改表，mysql中user表，测试不行。下，亲测有效

vi  /etc/mysql/my.cnf（注释掉bind-address）

Instead of skip-networking the default is now to listen only on
localhost which is more compatible and is not less secure.
bind-address = 127.0.0.1
安全删除

set SQL_SAFE_UPDATES = 0;

###  参考

[1]<https://aaaaaashu.gitbooks.io/mac-dev-setup/content/MySql/index.html>

[2]<http://stackoverflow.com/questions/22317204/exporting-only-table-structure-using-mysqlworkbench>
