---
layout:     post
title:      LVS 和 Nginx 的区别
subtitle:   Web服务器负载均衡的搭建
date:       2020-03-21
author:     Derrick
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - Linux
    - 负载均衡
---

# **LVS** 和 **Nginx** 的区别

-   # **LVS**

    **LVS**是`Linux Virtual Server`的简写，意即Linux虚拟服务器，是一个虚拟的服务器集群系统。
    **LVS**负载能力强，因为其工作方式逻辑非常简单，仅进行请求分发，而且工作在网络的第4层即`传输层`，所以**LVS**可以对几乎所有应用进行负载均衡，包括Web、数据库等。
    **LVS**有三种工作模式
>1. `VS/NAT`
    -   通过网络地址转换实现的虚拟服务器
    -   大并发访问时，调度器的性能成为瓶颈
>2. `VS/DR`
    -   直接使用路由技术实现虚拟服务器
    -   节点服务器需要配置VIP，注意MAV地址广播
>3. `VS/TUN`
    -   通过隧道方式实现虚拟服务器



-   # **Nginx**

    **Nginx**是一个高性能的HTTP和反向代理web服务器，同时也提供了`IMAP/POP3/SMTP`服务，而且也是一个安装非常的简单、配置文件非常简洁（还能够支持perl语法）、Bug非常少的服务。**Nginx**启动特别容易，并且几乎可以做到7X24不间断运行，即使运行数个月也不需要重新启动。你还能够不间断服务的情况下进行软件版本的升级。**Nginx**工作在网路第7层即`应用层`，所以可以对HTTP应用实施分流策略，比如域名、结构等。



***




### `LVS-NAT`部署

   |proxy ip|192.168.2.1|
   |:-:|:-:|
   |web1 ip|192.168.2.100|
   |web2 ip |192.168.2.200|



1. 准备俩台Web服务器(配置相同，ip分别为100/200)
 ```
    [root@web ~]# yum -y install httpd 
    [root@web ~]# echo "192.168.2.100/200" > /var/www/html/index.html 
    [root@web ~]# systemctl start httpd 
```
2. 确认调度器的路由转发功能
```    
    [root@proxy ~]# echo 1 > /proc/sys/net/ipv4/ip_forward 
    [root@proxy ~]# echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf 
```
3. 创建集群服务器 
```    	
    [root@proxy ~]# yum -y install ipvsadm 
    [root@proxy ~]# ipvsadm -A -t 192.168.2.1:80 -s wrr
```
4. 添加真实服务器 
```    
    [root@proxy ~]# ipvsadm -a -t 192.168.2.1:80 -r 192.168.2.100 -w 1 -m 
    [root@proxy ~]# ipvsadm -a -t 192.168.2.1:80 -r 192.168.2.200 -w 1 -m 
```
5. 查看规则列表，并保存规则 
```
   [root@proxy ~]# ipvsadm -Ln 
   [root@proxy ~]# ipvsadm-save -n > /etc/sysconfig/ipvsadm 
```
6. 客户端测试
客户端使用curl命令反复连接http://192.168.2.1，查看访问的页面是否会轮询到不同的后端真实服务器。

### `Nginx`反向代理部署

   |proxy ip|192.168.2.1|
   |:-:|:-:|
   |web1 ip|192.168.2.100|
   |web2 ip |192.168.2.200|
   
1. 准备俩台Web服务器(配置相同，ip分别为100/200)
```    
    [root@web ~]# yum -y install httpd 
    [root@web ~]# echo "192.168.2.100/200" > /var/www/html/index.html 
    [root@web ~]# systemctl start httpd
```
2. 在proxy代理服务器上安装(跳过)并编译Nginx(`with-stream`为反向代理模块)
```
    [root@proxy nginx-1.16.1]# ./config --prefix=/usr/local/nginx --user=nginx --with-stream
```
3.  配置nginx均衡 
```
    [root@proxy ~]# vim /usr/local/nginx/conf/nginx.conf 

    .
    .
    upstream web {  
    	server 192.168.2.100;          
    	server 192.168.2.200;      
    } 
    
    server {   
    	listen       80;  
    	server_name localhost;   
    	#charset koi8-r;  
    	#access_log /var/log/nginx/host.access.log main;   
    
    location / {  
    	proxy_pass http://web;      
	root  /usr/share/nginx/html;      
    	index index.html index.htm;  
    }
    .
    .
 ```
4. 启动服务 
```
[root@proxy ~]# nginx
```
5. 客户端测试
客户端使用curl命令反复连接http://192.168.2.1，查看访问的页面是否会轮询到不同的后端真实服务器。




<script type="text/javascript">document.write(unescape("%3Cspan id='cnzz_stat_icon_1281111180'%3E%3C/span%3E%3Cscript src='https://v1.cnzz.com/z_stat.php%3Fid%3D1281111180%26online%3D1%26show%3Dline' type='text/javascript'%3E%3C/script%3E"));</script>