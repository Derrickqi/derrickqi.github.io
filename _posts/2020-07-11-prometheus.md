---
layout:     post   				    # 使用的布局（不需要改）
title:      从零搭建Prometheus监控系统 				# 标题 
subtitle:   Prometheus+Grafana监控Docker容器及主机 #副标题
date:       2020-07-11 				# 时间
author:     Derrick 				# 作者
header-img: img/post-bg-travel-1.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - 监控
    - Docker
---

## Prometheus+Grafana监控Docker容器及主机

**什么是Prometheus?**

`Prometheus`是由SoundCloud开发的开源监控报警系统和时序列数据库(TSDB)。`Prometheus`使用Go语言开发，是Google BorgMon监控系统的开源版本。
2016年由Google发起Linux基金会旗下的原生云基金会(Cloud Native Computing Foundation), 将`Prometheus`纳入其下第二大开源项目。
Prometheus目前在开源社区相当活跃。
`Prometheus`和`Heapster`(Heapster是K8S的一个子项目，用于获取集群的性能数据。)相比功能更完善、更全面。Prometheus性能也足够支撑上万台规模的集群。


**Prometheus的特点**

* 多维度数据模型。
* 灵活的查询语言。
* 不依赖分布式存储，单个服务器节点是自主的。
* 通过基于HTTP的pull方式采集时序数据。
* 可以通过中间网关进行时序列数据推送。
* 通过服务发现或者静态配置来发现目标服务对象。
* 支持多种多样的图表和界面展示，比如Grafana等。



**安装准备**
### 一. 监控主机



1. 准备两台linux操作系统，本文均使用CentOS7版本。

|  HostName| IP  |
| :----: | :----: | 
| master  | 192.168.124.4 |
| node1  | 192.168.124.5 |



2. Docker部署Prometheus（master机器）

```shell
docker run -d \
--name=prometheus \
-p 9090:9090 \
-v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
prom/prometheus
```

编辑prometheus.yml配置文件
```shell
.
.
.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    static_configs:
    - targets: ['localhost:9090']
 #添加master及node节点添加
  - job_name: 'master'
    static_configs:
    - targets: ['192.168.124.4:9100']
  
  - job_name: 'node'
    static_configs:
    - targets: ['192.168.124.5:9100']

.
.
.
```





访问master的9090端口

![img](/img/2020-07-11-prometheus/prometheus1.png)

3. 部署node_exporter（master和node）
下载地址https://prometheus.io/download


```shell
[root@prometheus-4 ~]# tar -xf node_exporter-1.0.1.linux-amd64.tar.gz
[root@prometheus-4 ~]# cp node_exporter-1.0.1.linux-amd64/node_exporter /usr/local/bin/node_exporter
## 编辑启动脚本
[root@prometheus-4 ~]# vim /usr/lib/systemd/system/node_exporter.service

[Unit]
Description=node_export
After=network.target
 
[Service]
Type=simple
ExecStart=/usr/local/bin/node_exporter
Restart=on-failure
[Install]
WantedBy=multi-user.target

[root@prometheus-4 ~]# systemctl start node_exporter

## 监题9100端口
[root@prometheus-server ~]# netstat -anlptu|grep 9100
tcp6       0      0 :::9100                 :::*                    LISTEN      5216/node_exporter  
```

访问master及node端的9100端口
![img](/img/2020-07-11-prometheus/node_exporter1.png)

下图为node通过9100端口暴露出的监控指标
![img](/img/2020-07-11-prometheus/node_exporter2.png)

再次访问master的9090端口
![img](/img/2020-07-11-prometheus/prometheus2.png)




4. Docker部署grafana监控服务

```shell
docker run -d \
-p 3000:3000 \
--name=grafana \
-v /opt/grafana-storage:/var/lib/grafana \
grafana/grafana



新建空文件夹grafana-storage，用来存储数据

mkdir /opt/grafana-storage
设置权限

chmod 777 -R /opt/grafana-storage
```

访问master的3000端口
**默认账号密码都为admin**
![img](/img/2020-07-11-prometheus/Grafana1.png)

**点击设置，将URL改为http://192.168.124.4:9090并保存**
![img](/img/2020-07-11-prometheus/Grafana2.png)

**点击Import导入默认模板(代码为9276），并且保存**
![img](/img/2020-07-11-prometheus/Grafana3.png)


![img](/img/2020-07-11-prometheus/Grafana4.png)



![img](/img/2020-07-11-prometheus/Grafana5.png)




### 二. 监控主机容器
`cAdvisor`（Container Advisor）用于采集正在运行的容器资源使用和性能信息。
`cAdvisor`可以对节点机器上的资源及容器进行实时监控和性能数据采集，包括`CPU使用情况`、`内存使用情况`、`网络吞吐量`及`文件系统使用情况`.


1. Docker部署Cadvisor


在node节点执行

```shell
docker run -d \
--volume=/:/rootfs:ro \
--volume=/var/run:/var/run:ro \
--volume=/sys:/sys:ro \
--volume=/var/lib/docker/:/var/lib/docker:ro \
--volume=/dev/disk/:/dev/disk:ro \
--publish=8080:8080 \
--detach=true \
--name=cadvisor \
google/cadvisor:latest
```
**访问node节点的8080端口**
![img](/img/2020-07-11-prometheus/cadvisor1.png)
![img](/img/2020-07-11-prometheus/cadvisor2.png)

**同样cadvisor也会有同样的监控指标**
访问http://192.168.124.5:8080/metrics

![img](/img/2020-07-11-prometheus/cadvisor3.png)




2. **修改prometheus.yml,在末尾添加**

.
.
.
  - job_name: 'docker'
    static_configs:
    - targets: ['192.168.124.5:8080']
.
.
.


3. **再次访问master的9090端口，可以看到容器监控指标已经显示出来了**
![img](/img/2020-07-11-prometheus/cadvisor4.png)


**然后在master主机登录Grafana，导入Docker监模板，id：193**
![img](/img/2020-07-11-prometheus/Grafana6.png)

![img](/img/2020-07-11-prometheus/Grafana7.png)

**这样我们就可以看到Grafana已经同步被监控端的容器信息**
![img](/img/2020-07-11-prometheus/Grafana8.png)

![img](/img/2020-07-11-prometheus/Grafana9.png)
