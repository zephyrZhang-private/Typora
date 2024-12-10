# Nginx配置转发TCP协议 -- SMTP

## 前期准备

> 下载安装包

````
wget http://nginx.org/download/nginx-1.18.0.tar.gz 
````

> 安装依赖

````
yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel
yum install -y pcre-devel
````

> 添加用户

````
useradd -M -s /sbin/nologin nginx  
````

## 编译安装

> tar -zxvf nginx-1.18.0.tar.gz

 ````
## 编译时一定要有stream模块
./configure --prefix=/usr/local/nginx --user=nginx --group=nginx --with-mail  --with-stream  --with-http_stub_status_module --with-http_ssl_module

make && make install 
 ````

## 修改配置

> cd /usr/local/nginx/conf 
>
> cat nginx.conf.default | grep -v '#\|^$' > nginx.conf 
>
> vim /usr/local/nginx/conf/nginx.conf 

````
stream {

	server {

		listen 465; #监听端口号

		proxy_pass smtp.xxxx.com:465; #转发目的地

	}

}
````

## 配置服务

> vim /usr/lib/systemd/system/nginx.service

````
[Unit]
Description=Nginx Server
After=network.target

[Service]
## User=nginx
## Group=nginx
## UMask=0037

LimitNOFILE=1000000
LimitNPROC=1000000
LimitCORE=1000000


Type=forking
ExecStart=/usr/local/nginx/sbin/nginx
ExecStop=/usr/local/nginx/sbin/nginx -s stop
ExecReload=/usr/local/nginx/sbin/nginx -s stop  && /usr/local/nginx/sbin/nginx

[Install]
WantedBy=multi-user.target
````

## 重新加载服务

````
systemctl daemon-reload
systemctl status nginx
systemctl start nginx
systemctl stop nginx
````

## 注意

> 系统启动Nginx报错： [emerg] bind() to 0.0.0.0:XXXX failed (13: Permission denied)错误的处理方式
>
> 端口小于1024

````
1024以下端口启动时需要root权限，sudo nginx或者修改nginx.conf将启动用户nginx改为root
````

> 端口大于1024

````
## 查看http允许访问的端口

semanage port -l | grep http_port_t

## 将要启动的端口（8881）加入到如上端口列表中

semanage port -a -t http_port_t  -p tcp 8885 
````



