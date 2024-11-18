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

* 构建部署体系

````
mkdir -p data/log
mkdir -p data/temp
mkdir -p data/binlog
````

* 创建my.cnf

```` 
###### [client]配置模块 ######
[client]
default-character-set=utf8mb4
socket=/mysql/.data/tmp/mysql.sock

###### [mysql]配置模块 ######
[mysql]
# 设置MySQL客户端默认字符集
default-character-set=utf8mb4
socket=/mysql/.data/tmp/mysql.sock

###### [mysqld]配置模块 ######
[mysqld]
skip_host_cache
skip-name-resolve=1
port=21001
user=mysql
innodb_strict_mode=0
# 设置sql模式 sql_mode模式引起的分组查询出现*this is incompatible with sql_mode=only_full_group_by，这里最好剔除ONLY_FULL_GROUP_BY
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
# MySQL服务安装的基本目录
basedir=/mysql/.mysql
# MySQL服务数据存储
datadir=/mysql/.mysql/data
socket=/mysql/.data/tmp/mysql.sock

# 服务ID
server-id = 221

# 是否只读，1代表只读，0代表读写，从库建议设置成只读 SUPER权限可无视该规则（比如root账号）
read-only=0

# MySQL8 的密码认证插件 如果不设置低版本navicat无法连接
authentication_policy=mysql_native_password

# 允许最大连接数
max_connections=10000

# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8mb4

open_files_limit = 2000
table_open_cache = 2048M
#max_connections = 200
# 创建新表时将使用的默认存储引擎
default-storage-engine=InnoDB
innodb_buffer_pool_size = 24G
innodb_flush_log_at_trx_commit=1
innodb_io_capacity=5000
innodb_log_buffer_size=48M
general_log=0
# 0: 表名将按指定方式存储，并且比较区分大小写;
# 1: 表名以小写形式存储在磁盘上，比较不区分大小写；
lower_case_table_names=1

max_allowed_packet=16M


# 设置时区
default-time_zone='+8:00'

# binlog 配置 只要配置了log_bin地址 就会开启
log_bin = /mysql/.data/binlog/mysql_bin
# 日志存储天数 默认0 永久保存
# 如果数据库会定期归档，建议设置一个存储时间不需要一直存储binlog日志，理论上只需要存储归档之后的日志
#expire_logs_days = 30
# binlog最大值
max_binlog_size = 2048M
# 规定binlog的格式，binlog有三种格式statement、row、mixad，默认使用statement，建议使用row格式
binlog_format = ROW
# 在提交n次事务后，进行binlog的落盘，0为不进行强行的刷新操作，而是由文件系统控制刷新日志文件，如果是在线交易和账有关的数据建议设置成1，如果是其他数据可以保持为0即可
sync_binlog = 1

# MySQL服务器安全启动的配置部分
[mysqld_safe]
# mysqld_safe进程的日志文件位置
log-error=/mysql/.data/log/mysqld_safe.err
# mysqld_safe的pid文件存放位置
pid-file=/mysql/.data/tmp/mysqld.pid
# mysqld_safe使用的socket文件位置
socket=/mysql/.data/tmp/mysql.sock

# MySQLadmin工具的配置部分
[mysqladmin]
# mysqladmin工具使用的socket文件位置
socket=/mysql/.data/tmp/mysql.sock
````

* 初始化MySQL

````
--defaults-file：指定配置文件（要放在--initialize 前面）
--user： 指定用户
--basedir：指定安装目录
--datadir：指定初始化数据目录
--intialize-insecure：初始化无密码（否则生成随机密码）

mysqld --defaults-file=data/conf/my.cnf --basedir=mysql --datadir=/data --user=mysql --initialize-insecure

````

* 启动

````
--安全启动
mysqld_safe --defaults-file=data/conf/my.cnf &

--server启动
mysql.server start
````

* 登录

````
mysql -u root --skip-password

--修改密码
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';

--刷新权限
FLUSH PRIVILEGES;

--查看用户
use mysql;
select user,host,plugin,authentication_string from user;

开启远程
--创建用户
CREATE user 'root'@'%';
--设置首次密码
ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root';
--授权用户所有权限，刷新权限
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%';
FLUSH PRIVILEGES;
````







## 性能调优







## 主从复制

> 主节点创建主从复制用户

````
create user 'slave'@'%' identified with mysql_native_password by 'slave';
grant replication slave on *.* to 'slave'@'%';
````

>主节点机器查看binlog

````
show master status\G

--记录logfile和position
````

> 从节点开启复制

````
stop slave;
change master to  master_host='master_ip',master_user='slave_user',master_password='slave_passwd',master_port=21001,master_log_file='master_logfile',master_log_pos=master_position;
start slave;
````

> 4查看是否成功开启主从

````
show slave status\G

--以下两项为Yes即可
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
````

## 注意事项

````
修改密码需要注意版本
--5.6/5.1
update mysql.user set password=password('i2') where user='i2';
--5.7
ALTER USER 'i2'@'%' IDENTIFIED BY 'i2';
set password for 'i2'@'%'=password('i2');

解决MySQLmax连接数不生效
find && vim / -name mysqld.service
LimitNOFILE=65535
LimitNPROC=65535
service mysqld restart
````

