# MySQL部署

## 环境检查：

> 清空自带mysql和mariadb，避免冲突
>
> 删除/etc下my.cnf文件
>
> 检查用户组，id mysql

## MySQL Version 5.x

* 解压赋权

> 解压MySQL安装包
>
> MySQL目录赋MySQL所属权

* 编译初始化

```` 
su - mysql
cd ~/mysql/bin
./mysqld --user=mysql --datadir=~/mysql/data --basedir=~/mysql --initialize-insecure
````

* 配置文件

```` my.cnf
[mysqld]
datadir=~/mysql/data
port=3306
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
symbolic-links=0
max_connections=400
innodb_file_per_table=1
lower_case_table_names=1
log-bin=mysql-bin
server_id=1
binlog-format=row
sync_binlog=1
log_slave_updates=1
default_authentication_plugin=mysql_native_password
````

* Server文件

```` 
vim ~/mysql/support-files/mysql.server
/usr/local/mysql >> ~/mysql(实际安装位置，共五处修改)
````

* 启动

```` 
cd ~/mysql/support-files
./mysql.server start
````

* 登录

````
mysql -u root '-p[临时密码]'

--修改root密码
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'new_passwd';

--开放远程连接
use mysql;
update user set user.Host='%' where user.User='root';
flush privileges;

--验证远程连接
mysql -uroot -p'root' -h127.0.0.1 -P3306
````

* 检查binog

````
show variables like "log_%";
````

* 开机自启

````
cp ~/mysql/support-files/mysql.server /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld
chkconfig --add mysqld
chkconfig --list
````

## MySQL Version 8.x





## 注意事项

````
修改密码需要注意版本
--5.6/5.1
update mysql.user set password=password('i2') where user='i2';

--5.7
ALTER USER 'i2'@'%' IDENTIFIED BY 'i2';
set password for 'i2'@'%'=password('i2');
````

