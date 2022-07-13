# MySQL

## 安装

```shell
$> groupadd mysql
$> useradd -r -g mysql -s /bin/false mysql
$> cd /usr/local
$> tar zxvf /path/to/mysql-VERSION-OS.tar.gz
$> ln -s full-path-to-mysql-VERSION-OS mysql
$> cd mysql
$> mkdir mysql-files
$> chown mysql:mysql mysql-files  //给mysql用户赋予权限
$> chmod 750 mysql-files
$> bin/mysqld --initialize --user=mysql
$> bin/mysql_ssl_rsa_setup  //ssl安全连接
$> bin/mysqld_safe --user=mysql &  //启动mysql
# Next command is optional
$> cp support-files/mysql.server /etc/init.d/mysql.server
```
设置环境变量
export PATH=$PATH:/usr/local/mysql/bin

mysql -u root -p
或mysql -u root -p''

ALTER USER 'root'@'localhost' IDENTIFIED BY 'root-password';


./mysqladmin -u root -p shutdown

mysql.server {start|stop|restart}

ERROR 2002 (HY000): Can’t connect to local MySQL server through socket ‘/tmp/mysql.sock’ (2)
/etc/my.cnf没有配置[client]
[client]
port=3306
socket=/var/lib/mysql/mysql.sock
