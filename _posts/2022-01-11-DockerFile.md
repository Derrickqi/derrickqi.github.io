---
layout:     post   				    # 使用的布局（不需要改）
title:      正确的姿势编写DockerFile 				# 标题 
subtitle:   DockerFile案例 #副标题
date:       2022-01-11 				# 时间
author:     Derrick 				# 作者
header-img: img/post-bg-docker.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - Docker
    - 虚拟化
---

# DockerFile案例



### 1.1 将启动Docker容器，同时开启Docker容器对外的22端口的监听，实现通过CRT或者Xshell登录。



```shell
#设置基本的镜像，后续命令都以这个镜像为基础

FROM centos

#作者信息

MAINTAINER  SeVen7nu.github.io

#安装依赖工具&删除默认YUM源，使用YUM源为国内阿里云 YUM源；

RUN rpm --rebuilddb;yum install make wget tar gzip passwd openssh-server gcc -y

RUN rm -rf /etc/yum.repos.d/*;wget -P /etc/yum.repos.d/ https://mirrors.aliyun.com/repo/Centos-8.repo

#配置SSHD&修改root密码为123456

RUN yes|ssh-keygen -q -t rsa -b 2048 -f /etc/ssh/ssh_host_rsa_key -N ''

RUN yes|ssh-keygen -q -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N ''

RUN yes|ssh-keygen -q -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N ''

RUN echo '123456' | passwd --stdin root

#启动SSHD服务进程，对外暴露22端口；

EXPOSE  22

CMD /usr/sbin/sshd -D
```



### 1.2 **开启SSH 6379端口，让Redis端口对外访问，Dockerfile内容如下：**



```shell
FROM centos:latest

#作者信息

MAINTAINER  SeVen7nu.github.io

#安装依赖工具&删除默认YUM源，使用YUM源为国内阿里云 YUM源；

RUN rpm --rebuilddb;yum install make wget tar gzip passwd openssh-server gcc -y

RUN rm -rf /etc/yum.repos.d/*;wget -P /etc/yum.repos.d/ https://mirrors.aliyun.com/repo/Centos-8.repo

#配置SSHD&修改root密码为123456

RUN ssh-keygen -q -t rsa -b 2048 -f /etc/ssh/ssh_host_rsa_key -N ''

RUN ssh-keygen -q -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N ''

RUN ssh-keygen -q -t ed25519 -f /etc/ssh/ssh_host_ED25519_key -N ''

RUN echo '123456' | passwd --stdin root

#Redis官网下载Redis最新版本软件；

RUN wget -P /tmp/ http://download.redis.io/releases/redis-5.0.2.tar.gz

#解压Redis软件包，并且基于源码安装,创建配置文件；

RUN cd /tmp/;tar xzf redis-5.0.2.tar.gz;cd redis-5.0.2;make;make PREFIX=/usr/local/redis install;mkdir -p /usr/local/redis/etc/;cp redis.conf /usr/local/redis/etc/

#创建用于存储应用数据目录/data/redis&修改redis配置文件dir路径；

RUN mkdir -p /data/redis/

RUN sed -i 's#^dir.*#dir /data/redis#g' /usr/local/redis/etc/redis.conf

#将应用数据存储目录/data/进行映射，可以实现数据持久化保存；

VOLUME ["/data/redis"]

#修改Redis.conf监听地址为bind：0.0.0.0；

RUN sed -i '/^bind/s/127.0.0.1/0.0.0.0/g' /usr/local/redis/etc/redis.conf

#启动Redis数据库服务进程，对外暴露22和6379端口；

EXPOSE  22

EXPOSE  6379

CMD /usr/sbin/sshd;/usr/local/redis/bin/redis-server /usr/local/redis/etc/redis.conf
```



### 1.3 **基于Dockerfile开启Nginx 80端口，并远程连接服务器，dockerfile内容如下：**




```shell
FROM centos:latest

#作者信息

MAINTAINER  SeVen7nu.github.io

#安装依赖工具&删除默认YUM源，使用YUM源为国内阿里云 YUM源；

RUN rpm --rebuilddb;yum install make wget tar gzip passwd openssh-server gcc pcre-devel open

ssl-devel net-tools -y

RUN rm -rf /etc/yum.repos.d/*;wget -P /etc/yum.repos.d/ https://mirrors.aliyun.com/repo/Centos-8.repo

#配置SSHD&修改root密码为123456

RUN yes|ssh-keygen -q -t rsa -b 2048 -f /etc/ssh/ssh_host_rsa_key -N ''

RUN yes|ssh-keygen -q -t ecdsa -f /etc/ssh/ssh_host_ecdsa_key -N ''

RUN yes|ssh-keygen -q -t ed25519 -f /etc/ssh/ssh_host_ed25519_key -N ''

RUN echo '123456' | passwd --stdin root

#Nginx官网下载Nginx最新版本软件；

RUN wget -P /tmp/ http://nginx.org/download/nginx-1.14.2.tar.gz

#解压Nginx软件包，隐藏WEB服务器版本号；

RUN cd /tmp/;tar xzf nginx-1.14.2.tar.gz;cd nginx-1.14.2;sed -i -e 's/1.14.2//g' -e 's/nginx\//WS/g' -e 's/"NGINX"/"WS"/g' src/core/nginx.h

#基于源码安装,创建配置文件；

RUN cd /tmp/nginx-1.14.2;./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module;make;make install

#启动Nginx服务进程，对外暴露22和80端口；

EXPOSE  22

EXPOSE  80

CMD /usr/local/nginx/sbin/nginx;/usr/sbin/sshd -D
```



### 1.4 **Dockerfile来生成mysql镜像并启动运行**



```shell
FROM centos:v1

RUN groupadd -r mysql && useradd -r -g mysql mysql

RUN rpm --rebuilddb;yum install -y gcc zlib-devel gd-devel

ENV MYSQL_MAJOR 5.6

ENV MYSQL_VERSION 5.6.20

RUN

&& curl -SL "http://dev.mysql.com/get/Downloads/MySQL-$MYSQL_MAJOR/mysql-$MYSQL_VERSION-linux-glibc2.5-x86_64.tar.gz" -o mysql.tar.gz \

&& curl -SL "http://mysql.he.net/Downloads/MySQL-$MYSQL_MAJOR/mysql-$MYSQL_VERSION-linux-glibc2.5-x86_64.tar.gz.asc" -o mysql.tar.gz.asc \

&& mkdir /usr/local/mysql \

&& tar -xzf mysql.tar.gz -C /usr/local/mysql \

&& rm mysql.tar.gz* \

ENV PATH $PATH:/usr/local/mysql/bin:/usr/local/mysql/scripts

WORKDIR /usr/local/mysql

VOLUME /var/lib/mysql

EXPOSE 3306

CMD ["mysqld", "--datadir=/var/lib/mysql", "--user=mysql"]
```




### Docker遇到的问题及解决方法



**报错内容： System has not been booted with systemd as init system (PID 1). Can't operate.Failed to connect to bus: Host is down**




**解决方法：docker run -itd   --privileged --name myCentos centos /usr/sbin/init**
原因就是： 默认情况下，在第一步执行的是 /bin/bash，而因为docker中的bug，无法使用systemctl;所以我们使用了 /usr/sbin/init 同时 --privileged 这样就能够使用systemctl了，但覆盖了默认的 /bin/bash;因此我们如果想进入容器 就不能再使用 docker attach myCentos 