# Linux静默安装oracle19c

## 一、前期准备

> * 资源要求
> * 固定依赖 
> * 系统限制

### 检查安装环境

````
grep MemTotal /proc/meminfo
grep SwapTotal /proc/meminfo
df -h /tmp
df -h /dev/shm
free -g
uname -m
````

### 安装依赖

````
yum install -y compat-libcap1 compat-libstdc++-33 gcc-c++ ksh libaio-devel libstdc++-devel elfutils-libelf-devel fontconfig-devel libXrender-devel
````

### 检查依赖

````check depend
rpm -q --qf '%{NAME}-%{VERSION}-%{RELEASE} (%{ARCH})\n' \
bc \
binutils \
compat-libcap1 \
compat-libstdc++-33 \
elfutils-libelf \
elfutils-libelf-devel \
fontconfig-devel \
glibc \
gcc \
gcc-c++  \
glibc \
glibc-devel \
ksh \
libstdc++ \
libstdc++-devel \
libaio \
libaio-devel \
libXrender \
libXrender-devel \
libxcb \
libX11 \
libXau \
libXi \
libXtst \
libgcc \
libstdc++-devel \
make \
sysstat \
unzip \
readline \
smartmontools | grep 'not installed' |column -t
````

### 配置内核参数

> * 手动使配置文件生效

````
vim /etc/sysctl.conf
/sbin/sysctl -p
````

```sys.conf
fs.aio-max-nr = 1048576
fs.file-max = 6815744
kernel.shmall = 16451328
kernel.shmmax = 33692319744
kernel.shmmni = 4096
kernel.sem = 250 32000 100 128
net.ipv4.ip_local_port_range = 9000 65500
net.core.rmem_default = 262144
net.core.rmem_max = 4194304
net.core.wmem_default = 262144
net.core.wmem_max = 1048576
```

## 二、用户配置

### 创建用户组

> * Oracle用户组，指定位置创建用户

````set userhome
groupadd oinstall
groupadd dba
groupadd asmdba
groupadd backupdba
groupadd dgdba
groupadd kmdba
groupadd racdba
groupadd oper
useradd -g oinstall -G dba,asmdba,backupdba,dgdba,kmdba,racdba,oper -d /u01 oracle
````

> * Oracle用户组，默认位置创建用户

````default userhome
groupadd oinstall
groupadd dba
groupadd asmdba
groupadd backupdba
groupadd dgdba
groupadd kmdba
groupadd racdba
groupadd oper
useradd -g oinstall -G dba,asmdba,backupdba,dgdba,kmdba,racdba,oper -m oracle
````

### 安装rlwrap

> * 此处为root用户

```Setup rlwrap
yum -y install readline*
tar-zxvf rlwrap-0.46.1.tar.gz -C /home/oracle
cd /home/oracle/rlwrap-0.46.1
./configure
make install
```

### 用户环境变量

> * 此处为Oracle用户
> * 编辑局部环境变量--仅对当前用户生效
> * 修改后刷新变量

````
su - oracle
vim .bash_profile
source .bash_profile
````

````Oracle Profile
export NLS_LANG=AMERICAN_AMERICA.AL32UTF8
export ORACLE_BASE=/u01/app/oracle
export ORACLE_HOME=/u01/app/oracle/product/19.5.0
export PATH=$PATH:$ORACLE_HOME/bin:/usr/local/bin
export ORACLE_HOSTNAME=172.20.42.11
export ORACLE_SID=orcl
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:$ORACLE_HOME/rdbms/lib:$ORACLE_HOME/network/lib:/lib:/usr/lib
export CLASSPATH=$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib:$ORACLE_HOME/network/jlib

## 配置变量后，未安装rlwrap无法正常使用 --sqlplus not command
## 安装rlwrap，解决sqlplus上下左右键乱码
alias sqlplus='/usr/local/rlwrap/bin/rlwrap sqlplus'
alias sqlplus='rlwrap sqlplus'
alias rman='rlwrap rman'
````

### 创建安装目录

> * 此处为root用户
> * 修改所属Oracle用户组

````
mkdir /u01
chmod 777 /u01
mkdir -p /u01/app/oracle/product/19.5.0
mkdir -p /u01/app/oraInventory
mkdir -p /u01/app/oracle/flash_recovery_area
cd /u01/app/
chown -R oracle:oinstall oracle/
chown -R oracle:oinstall oraInventory/
````

### 解压安装包

> * 此处为Oracle用户

````
su - oracle
cd /u01/app/oracle/product/19.5.0
unzip LINUX.X64_193000_db_home.zip
````

### 安装配置模板

> * 了解内容
> * 安装、监听、建库的配置模板

````
软件:/u01/app/oracle/product/19.5.0/install/response/db_install.rsp
监听:/u01/app/oracle/product/19.5.0/assistants/netca/netca.rsp
建库:/u01/app/oracle/product/19.5.0/assistants/dbca/dbca.rsp
````

## 三、Oracle部署

### 安装软件

> * 此处oracle用户
> * 模板安装

````
./runInstaller -silent -responseFile /u01/app/oracle/product/19.5.0/install/response/db_install.rsp 
````

> * 动态参数配置安装

````install DB software
./runInstaller -ignorePrereq -waitforcompletion -silent \
oracle.install.option='INSTALL_DB_SWONLY' \
UNIX_GROUP_NAME=oinstall \
INVENTORY_LOCATION=/u01/app/oraInventory \
ORACLE_HOME=/u01/app/oracle/product/19.5.0 \
ORACLE_BASE=/u01/app/oracle \
oracle.install.db.InstallEdition=EE \
oracle.install.db.OSDBA_GROUP=dba \
oracle.install.db.OSOPER_GROUP=oper \
oracle.install.db.OSBACKUPDBA_GROUP=backupdba \
oracle.install.db.OSDGDBA_GROUP=dgdba \
oracle.install.db.OSKMDBA_GROUP=kmdba \
oracle.install.db.OSRACDBA_GROUP=racdba \
oracle.install.db.rootconfig.executeRootScript=false
````

### 配置监听

> * 模板生成监听文件

````lintener.model
netca /silent /responseFile /u01/app/oracle/product/19.5.0/assistants/netca/netca.rsp
````

> * 修改LISTENER监听

````
cd /u01/app/oracle/product/19.5.0/network/admin

vim listener.ora
````

````listener.ora
## 修改对应IP/HostName，端口等，配置SID_LIST

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 172.20.42.11)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )
SID_LIST_LISTENER =
(SID_LIST =
  (SID_DESC =
  (GLOBAL_DBNAME = ORCL)
  (SID_NAME = orcl)
  )
  (SID_DESC =
  (GLOBAL_DBNAME = PDB01)
  (SID_NAME = orcl)
  )
)
````

> * 修改TNS监听

````
cd /u01/app/oracle/product/19.5.0/network/admin

vim tnsnames.ora
````

````
## 修改对应IP/HostName，端口等，配置PDB实例监听

LISTENER_ORCL =
  (ADDRESS = (PROTOCOL = TCP)(HOST = 172.20.42.11)(PORT = 1521))


ORCL =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 172.20.42.11)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl)
    )
  )

PDB01 =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 172.20.42.11)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = pdb01)
    )
  )
LISTENER_PDB01 =
  (ADDRESS = (PROTOCOL = TCP)(HOST = 172.20.42.11)(PORT = 1521))
````

> * 启动监听

````
lsnrctl stop
lsnrctl start
````

### 安装数据库

> * 安装CDB，附加PDB
> * 静默安装

````
dbca -silent -createDatabase -templateName General_Purpose.dbc -responseFile NO_VALUE \
-gdbname orcl  -sid orcl \
-createAsContainerDatabase TRUE \
-numberOfPDBs 1 \
-pdbName pdb01 \
-pdbAdminPassword oracle \
-sysPassword oracle -systemPassword oracle \
-datafileDestination '/u01/app/oracle/oradata' \
-recoveryAreaDestination '/u01/app/oracle/flash_recovery_area' \
-redoLogFileSize 1024 \
-storageType FS \
-characterset AL32UTF8  -nationalCharacterSet AL16UTF16 \
-sampleSchema true \
-totalMemory  24576 \
-databaseType OLTP  \
-emConfiguration NONE
````

### 卸载库

> 模板
>
> dbca -silent -deleteDatabase -sourceDB SID -sysDBAUserName SYS -sysDBAPassword passwd

````
dbca -silent -deleteDatabase -sourceDB orcl -sysDBAUserName sys -sysDBAPassword oracle
````

## 四、Oracle配置

### 修改用户默认表空间

> 注意路径需与自己安装路径对应

> * 创建用户表空间

````
create tablespace i2 logging datafile '/u01/app/oracle/oradata/ORCL/pdb01/i2.dbf' size 1024m autoextend on next 128m maxsize unlimited;
````

> * 创建临时表空间

````
create temporary tablespace i2temp tempfile '/u01/app/oracle/oradata/ORCL/pdb01/i2temp.dbf' size 128m autoextend on next 64m maxsize unlimited;
````

> * 查询创建的数据文件

````
select name from v$datafile;
````

> * 创建用户指定表空间与临时表空间

````
create user i2 identified by i2 default tablespace i2 temporary tablespace i2temp;
````

### 相关日志

> * 归档日志

````
## 开启
shutdown immediate
startup mount
alter database archivelog;
alter database open;
alter pluggable database pdb01 open read write;
````

> * 开启最小附加日志

````
alter database add supplemental log data;
````

> * 改写强制写日志

````
alter database force logging;
````

### 表数据量

> * 查询某个表的大小

````
select Segment_Name "表名",sum(bytes)/1024/1024 "表大小(M)" From User_Extents Group By Segment_Name  having Segment_Name='要查表名'; 
	analyze table dds_lob compute statistics; 
	select num_rows * avg_row_len 
	from user_tables 
where table_name = 'DDS_LOB';
````

