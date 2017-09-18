# 树莓派安装MySql

在我安装MySQL时遇到了无法登陆的问题,困扰了一天,终于解决了.

问题在于安装MySQL时没有提示输入root密码,在安装成功后在一般用户下使用`mysql -u root -p`命令始终提示权限不允许,无法登录,切换root用户后不需要输入密码即可登录.

求助Stackoverflow后发现是由于最近通过apt方式安装的MySQL的认证默认使用的是UNIX auth_socket plugin