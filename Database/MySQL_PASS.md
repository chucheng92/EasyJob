## MySQL修改root密码

### 方法一：用set password命令

首先，登陆mysql

mysql -u root -p

然后执行set password命令

set password for root@localhost = password('654321');

上面例子，将root密码更改为654321

### 方法二：使用mysqladmin

格式为：mysqladmin -u用户名 -p旧密码 password 新密码

mysqladmin -uroot -p123456 password "654321"

上面例子，将root密码由123456更改为654321

### 方法三：更改mysql的user表

首先，登陆mysql

```
mysql -uroot -p
然后操作mysql库的user表，进行update
mysql> use mysql;
mysql> update user set password=password('654321') where user='root' and host='localhost';
mysql> flush privileges;
```

### 方法四：忘记密码的情况下

首先停止mysql服务

1、service mysqld stop

以跳过授权的方式启动mysql

2、mysqld_safe --skip-grant-tables &

3、mysql -u root

操作mysql库的user表，进行update

```
mysql> use mysql;
mysql> update user set password=password('654321') where user='root' and host='localhost';
mysql> flush privileges;
mysql> quit
重启mysql服务
```

重启mysql服务
service mysqld restart

## 开启远程mysql连接

步骤：
1、登入mysql 

2、use mysql命令

3、GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;

4、flush privileges

5、查看select host,user from user