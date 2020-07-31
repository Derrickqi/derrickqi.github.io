---
layout:     post   				    # 使用的布局（不需要改）
title:      Kubernetes部署Java项目 				# 标题 
subtitle:    #副标题
date:       2020-07-31 				# 时间
author:     Derrick 				# 作者
header-img: img/post-bg-view-1.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - Kubernetes
    - Docker
---

## Kubernetes部署Java项目



### 安装Java环境以及Maven仓库



准备项目代码————>编译构建————>将产出得wae打到镜像中————>推送镜像仓库



1.准备编译环境
```shell
yum -y install java-1.8.0-openjdk maven -y
```



2.修改Maven镜像源
```shell
vim /etc/maven/settings.xml
  <mirrors>
    <mirror>     
      <id>central</id>     
      <mirrorOf>central</mirrorOf>     
      <name>aliyun maven</name>
      <url>https://maven.aliyun.com/repository/public</url>     
    </mirror>
  </mirrors>
```



3. 编译构建(在Java目录下执行)
```shell
mvn clean package -D maven.skip.test=true 


参数：
- maven.skip.test=true : 跳过maven单元检测
```



### 镜像制作



1.编写dockerfile
```shell
vim dockerfile



FROM tomcat:8
RUN rm -rf /usr/local/tomcat/webapps/*
ADD tartget/*.war /usr/local/tomcat/webapps/ROOT.war
```



2.构建镜像
```shell
docker build -t java-demo:v1 .
```



3.将镜像推送到自己得私有仓库
```
#登陆dockerHub
docker login


#将构建的镜像打tag
docker tag java-demon:v1 seven7num/java-demon:v1
```



## 部署应用到k8s




1.部署deployment
```shell
#创建应用
kubectl create deployment web --image=seven7num/java-demon:v1  

#查看pod详情
kubectl describe pod web

查看pod状态
kubectl get pod
```



2.发布应用
```shell
#发布应用并暴露端口
kubectl expose deployment web --port=80 --target-port=8080 --type=NodePort 

#查看应用
kubectl get service

#访问页面
curl http://nodeip
```




