# 海量数据 Vast Base G100 部署

## 一. 环境配置

### 防火墙

* 关闭防火墙

````
--检查状态
firewall-cmd --state
systemctl status firewalld.service

--关闭&&关闭自启
systemctl stop firewalld.service
systemctl disable firewalld.service
````

* 开放端口

````
--开放端口
firewall-cmd --zone=public --permanent --add-port=5432/tcp

--重载防火墙配置
firewall-cmd --reload

--开放端口列表
firewall-cmd --list-port
````

### selinux配置

* 临时关闭

````
--查看是否开启SELINUX。如果是未开启则是 diabled，已开启则是enforcing，宽容模式为 permissive
getenforce

--若为开启状态则临时关闭SELINUX
setenforce 0
````

* 永久关闭

````
--通过修改配置文件永久关闭SELINUX
vi /etc/selinux/config
SELINUX=disabled

--重启生效
reboot
````

### 时区设置

````
--查看时区
timedatectl

--修改时区
tzselect
````

### 时间同步

* 手动更改

````
--手动修改格式
date -s "2025-01-01 08:00:00"

--将系统时间写入硬件时间
hwclock -w | clock -w
````

* 自动修改

````
--上传时间同步文件
mv sync.service /usr/lib/systemd/system/

--重载服务
systemctl daemon-reload

--开启开机自动同步
systemctl start sync.service
systemctl enable sync.service
````

### 内核参数

> vim /etc/sysctl.conf 

````
fs.aio-max-nr=1048576
fs.file-max= 76724600
kernel.sem = 4096 2097152000 4096 512000
kernel.shmall = 26843545         # pages,  80% MEM or higher
kernel.shmmax = 68719476736     # bytes,  80% MEM or higher
kernel.shmmni = 819200
net.core.netdev_max_backlog = 10000
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 4194304
net.core.somaxconn = 4096
net.ipv4.tcp_fin_timeout = 5
vm.dirty_background_bytes = 409600000 
vm.dirty_expire_centisecs = 3000
vm.dirty_ratio = 80
vm.dirty_writeback_centisecs = 50
vm.overcommit_memory = 0
vm.swappiness = 0
net.ipv4.ip_local_port_range = 40000 65535
fs.nr_open = 20480000
````

> sysctl -p 

### 资源限制

>  vi /etc/security/limits.conf

````
vastbase soft nproc unlimited
vastbase hard nproc unlimited
vastbase soft stack unlimited
vastbase hard stack unlimited
vastbase soft core unlimited
vastbase hard core unlimited
vastbase soft memlock unlimited
vastbase hard memlock unlimited
vastbase soft nofile 10240000
vastbase hard nofile 10240000
````

### 远程登录

> vi /etc/ssh/sshd_config 

````
PermitRootLogin yes
````

> service sshd restart 

### IPC配置

> vi /etc/systemd/logind.conf 

````
RemoveIPC=no
````

> vi /usr/lib/systemd/system/systemd-logind.service 

````
RemoveIPC=no
````

> systemctl daemon-reload
>
> systemctl restart systemd-logind 

### 补充依赖

````
yum install -y libxslt libxslt-devel
````

> 依赖列表

````
readline : 6.2
python : 2.7.5
libicu : 50.2
cracklib : 2.9.0
libxslt : 1.1.28
tcl : 8.5.13
perl : 5.16.3
openldap : 2.4.44
pam : 1.1.8
systemd-libs : 219
bzip2 : 1.0.6
gettext : 0.19.8.1
libaio : 0.3.109
ncurses-libs : 5.9
````

## 二. 单机部署

### 安装准备

> 创建安装用户并指定目录
>
> 创建安装包存放目录
>
> 将db_install.rsp与installer.tar.gz上传至basic

````
mkdir /vastbase
mkdir -p /vastbase/basic
useradd -d /vastbase vastbase
cp /root/.bash_profile /root/.bashrc /vastbase
chown -R vastbase:vastbase /vastbase
````

### 静默安装

> root用户登录
>
> 解压安装程序
>
> 赋予权限

````
cd /vastbase/basic
tar -xvf Vastbase-G100-installer-2.2_Build15-17408-kylin_v10sp2-x86_64-no_mot-20231221.tar.gz
chown -R vastbase:vastbase /vastbase
chmod 775 /vastbase/basic/*
````

> 编辑静默安装参数文件
>
> cat db_install.rsp

````
vastbase_password=Aa123456
encryption_key=Aa123456
vastbase_home=/vastbase/local/vbinstall
vastbase_data=/vastbase/data/vbinstall
port=5432
max_connections=100
shared_buffers=200
db_compatibility=A
isinitdb=true
````

> vastbase用户执行

````
su - vastbase
cd basic/vastbase-installer/
./vastbase_installer --silent -responseFile /vastbase/basic/db_install.rsp
````

### 启动服务

> 服务启停

````
vb_ctl start
vb_ctl status
vb_ctl stop
vb_ctl restart
````

### 远程用户

> 登录服务

````
-d：指定数据库
-U：指定用户
-p：指定端口
-h：指定IP
-W：指定密码
-r：防止上下键乱码

vsql -d vastbase -p 5432 -r
````

> 创建用户并赋权

````
create user zephyr password 'zephyr@6377';
grant all privileges to zephyr;
````

> 删除vastbase中自动创建的用户模式

````
drop schema zephyr;
````

### 配置服务

> 配置监听
>
> vi /vastbase/data/vastbase/postgresql.conf

```
port=21001
```

> 配置远程连接
>
> vi /vastbase/data/vastbase/pg_hba.conf

```
# IPv4 local connections:
host    all             all             0.0.0.0/0               md5
```

> 重启生效

````
vb_ctl restart
````

### 登录验证

````
vsql -d zephyr -U zephyr -W zephyr@6377 -h 10.1.111.95 -p 21001 -r
vsql -d zephyr -U zephyr -W zephyr@6377 -h 10.1.111.97 -p 21001 -r
````

### 完成创建

> 创建库

````
create database zephyr owner zephyr;
grant all privileges on database zephyr to zephyr;
````

> 创建模式

````
create schema zephyr;
grant all on schema zephyr to zephyr;
````

