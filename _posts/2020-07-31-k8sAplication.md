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



3.编译构建(在Java目录下执行)
```shell
mvn clean package -D maven.skip.test=true 




参数：
maven.skip.test=true : 跳过maven单元检测
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



3.修改java应用连接数据库信息

```shell
server:
  port: 8080
spring:
  datasource:
    url: jdbc:mysql://java-demon-db:3306/test?characterEncoding=utf-8  #定义数据库信息
    username: root
    password: 123456
    driver-class-name: com.mysql.jdbc.Driver
  freemarker:
    allow-request-override: false
    cache: true
    check-template-location: true
    charset: UTF-8
    content-type: text/html; charset=utf-8
    expose-request-attributes: false
    expose-session-attributes: false
    expose-spring-macro-helpers: false
    suffix: .ftl
    template-loader-path:
      - classpath:/templates/

```


4.将镜像推送到自己得私有仓库
```
#登陆dockerHub
docker login


#将构建的镜像打tag
docker tag java-demon:v1 seven7num/java-demon:v1


#将镜像推送至DockerHub仓库
docker push seven7num/java-demon:v1
```



## 部署应用到k8s
1.部署java应用的deployment
```shell
#创建应用
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: web
  name: java-demon
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
      project: java
  template:
    metadata:
      labels:
        app: web
        project: java
    spec:
      containers:
      - image: seven7num/java-demon:v1
        name: java-demon
        imagePullPolicy: IfNotPresent
        env:
         - name: DEMO_GREETING
           value: "Hello from the environment"
         - name: DEMO_FAREWELL
           value: "Such a sweet sorrow"
        resources:
          requests:
            memory: "1Gi"
            cpu: "0.5"
          limits:
            memory: "10Gi"
            cpu: "1"
        livenessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10

#查看pod详情
kubectl describe pod web

查看pod状态
kubectl get pod
```



2.发布应用
```shell
#发布应用并暴露端口
apiVersion: v1
kind: Service
metadata:
  name: java-demon
spec:
  selector:
    app: web
    project: java
  ports:
    - protocol: TCP
      port: 80
      nodePort: 30010
      targetPort: 8080
  type: NodePort 


#查看应用
kubectl get service


```



3.部署pv持久存储
```shell
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  hostPath:
    path: /data

```



4.利用secret部署mysql的deployment，service以及pvc
```shell
apiVersion: v1
kind: Secret
metadata:
  name: java-demon-db 
  namespace: default
type: Opaque
data:
  mysql-root-password: "MTIzNDU2"
  mysql-password: "MTIzNDU2"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: java-demon-db 
  namespace: default
spec:
  selector:
    matchLabels:
      project: www
      app: mysql
  template:
    metadata:
      labels:
        project: www
        app: mysql
    spec:
      containers:
      - name: db
        image: mysql:5.7.30
        resources:
          requests:
            cpu: 500m
            memory: 512Mi
          limits: 
            cpu: 500m
            memory: 512Mi
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: java-demon-db
              key: mysql-root-password
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: java-demon-db
              key: mysql-password
        - name: MYSQL_USER
          value: "qihao"
        - name: MYSQL_DATABASE
          value: "k8s"
        ports:
        - name: mysql
          containerPort: 3306
        livenessProbe:
          exec:
            command:
            - sh
            - -c
            - "mysqladmin ping -u root -p${MYSQL_ROOT_PASSWORD}"
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          exec:
            command:
            - sh
            - -c
            - "mysqladmin ping -u root -p${MYSQL_ROOT_PASSWORD}"
          initialDelaySeconds: 5
          periodSeconds: 10
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
        
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: mysql-pv-claim
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
apiVersion: v1
kind: Service
metadata:
  name: java-demon-db
  namespace: default
spec:
  type: ClusterIP
  ports:
  - name: mysql
    port: 3306
    targetPort: mysql
  selector:
    project: www
    app: mysql 

```

#访问页面
curl http://nodeip
