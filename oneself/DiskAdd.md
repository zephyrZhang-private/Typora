# 本文讲解磁盘扩容

> 针对现有磁盘，进行扩容操作
>
> Linux系统

## 1、相关命令

> 查看盘符信息：df
>
> 查看磁盘挂载：lsblk
>
> 磁盘分区：fdisk
>
> 创建物理卷：pvcreate
>
> 创建分区：vgextend
>
> 查看物理卷信息：vgdisplay
>
> 添加磁盘：lvresize
>
> 动态扩容：resize2fs | xfs_growfs

## 2、系统盘扩容

### 2.1 磁盘占用

````扩容之前
df -h
````

### 2.2 查看分区

````
## 查看磁盘是否存在未分配空间

fdisk -l
````

### 2.3 分区 

````
fdisk /dev/sda
````

````
命令 (输入 m 获取帮助)：n  #新建分区
Select (default p)：p  #主分区
分区号 (3，4，默认3)：3|回车  #第三个分区
起始 扇区：回车  #默认值
Last 扇区，+扇区 or +size{K,M,G}：回车 #默认值
命令 (输入 m 获取帮助)：t
Hex 代码(输入 L 列出所有代码)：L
Hex 代码(输入 L 列出所有代码)：8e  #分区格式为LVM
命令 (输入 m 获取帮助)：w  #保存
````

### 2.4 重新查看分区

````
## 若查看不到，可reboot重启服务器，重新链接终端

fdisk -l
````

### 2.5 创建物理卷和分区

````
## 物理卷
pvcreate /dev/sda3

## 分区
vgextend centos /dev/sda3
````

### 2.6 查看物理卷信息 

````
## 获取可增加硬盘空间总量

vgdisplay
````

### 2.7 增加

````
## 按照实际大小酌情增加，比如现在需增加400GB

lvresize -L +400G /dev/mapper/centos-root
````

### 2.8 动态扩容分区 

````
resize2fs /dev/mapper/centos-root

## resize2fs报错的话，执行

xfs_growfs /dev/mapper/centos-root
````

### 2.9 查看扩容结果

````
df -h
````

## 3、其他磁盘扩容

同上系统盘类似

