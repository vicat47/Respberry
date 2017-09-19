# 树莓派安装MySql

在我安装MySQL时遇到了无法登陆的问题,困扰了一天,终于解决了.

问题在于安装MySQL时没有提示输入root密码,因为它安装的根!本!不!是!MySQL,而是一个叫mariadb的数据库,不知道是啥玩意,在安装成功后在一般用户下使用`mysql -u root -p`命令始终提示权限不允许,无法登录,切换root用户后不需要输入密码即可登录.

求助Stackoverflow后发现是由于树莓派通过apt方式安装的MySQL是为树莓派设计的,认证默认使用的是UNIX_socket plugin,通过socket连接  
这意味着:db_user使用它,将会被**系统用户**认证,这样就会导致当你使用root账户登录时甚至不需要密码.  
可以用`root`用户执行以下操作查看  
```sql
$ sudo mysql -u root # 必须使用root用户操作
mysql> USE mysql;
mysql> SELECT User, Host, plugin FROM mysql.user;

+------------------+-----------------------+
| User             | plugin                |
+------------------+-----------------------+
| root             | auth_socket           |
| mysql.sys        | mysql_native_password |
| debian-sys-maint | mysql_native_password |
+------------------+-----------------------+
```  
可以明显的看出,root用户使用的是socket验证的方式登录的,如果想要使用用户名密码登陆,就需要将插件修改为`mysql_native_password`.  
有两种方法可以解决这个问题
1. 直接修改root用户使用`mysql_native_password`插件
2. 创建一个新用户
```sql
$ sudo mysql -u root # 必须使用root用户操作,因为没有其他的用户

mysql> USE mysql;
mysql> CREATE USER '你的用户名'@'localhost' IDENTIFIED BY '你的密码';
mysql> GRANT ALL PRIVILEGES ON * . * TO '你的用户名'@'localhost';
mysql> UPDATE user SET plugin='mysql_native_password' WHERE User='你的用户名';
mysql> FLUSH PRIVILEGES;
mysql> exit;

$ service mysql restart
```
这样就OK了,enjoy it.

## 远程连接MySQL
mysql的远程连接默认是关闭的,只需要修改`/etc/mysql/mariadb.conf.d`,注释掉`bind-address`一行,修改需要root权限,然后重启...连接时居然又报错...绝望  
`ERROR 1130: Host '192.168.1.3' is not allowed to connect to this MySQL server`
原因是用户的host项为`localhost`...滚去改表...  
有两种办法
1. 将user表里的'localhost'改为'%'
```sql
$ mysql -u root -p
mysql>use mysql;
mysql>update user set host = '%' where user = 'root';
mysql>select host, user from user;
```
2. 授权某ip连接数据库
```sql
GRANT ALL PRIVILEGES ON *.* TO '你要授权的用户名'@'你要授权的ip,如果为所有ip则为%' IDENTIFIED BY '创建该ip的登陆密码,可以和用户密码不同' WITH GRANT OPTION;
```  
finally 更新权限!important  
`mysql>flush privileges;`