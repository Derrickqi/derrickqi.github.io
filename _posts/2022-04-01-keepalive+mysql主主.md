---
layout:     post   				    # 使用的布局（不需要改）
title:      Keepalived + mysql主主（生产环境） 	# 标题 
subtitle:   实现数据库高可用 	#副标题
date:       2022-04-01				# 时间
author:     Derrick 				# 作者
header-img: img/post-bg-mysqlKeepalibed.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - mysql
    - 数据库
---

### Keepalived高可用 + mysql主主结构



#### 1.1 master和backup两台机器都安装MySQL数据库（两台机器都操作）

|  HostName| IP  |
| :----: | :----: | 
| master  | 172.17.0.2 |
| backup  | 172.17.0.3 |




```
[root@centos7 ~]# wget https://downloads.mysql.com/archives/get/p/23/file/mysql-5.7.36-1.el7.x86_64.rpm-bundle.tar
[root@centos7 ~]# tar -xvf mysql-5.7.36-1.el7.x86_64.rpm-bundle.tar -C /tmp
[root@centos7 ~]# yum -y install /tmp/*
[root@centos7 ~]# systemctl start mysqld
[root@centos7 ~]# grep -io 'password' /var/log/mysqld.log         #初始化密码在/var/log/mysqld.log中
```
<br/><br/><br/><br/>
#### 1.2 master和backup两台机器配置主主同步

**master(172.17.0.2)机器操作**

```
[root@centos7 ~]# cat /etc/mysql/my.cnf

bind-address = 172.17.0.2
server-id = 1
log_bin = mysql-bin 	#开启binglog日志
```

```
mysql> CREATE USER 'replica'@'172.17.0.3' identified by '123qqq...A';

mysql> ALTER USER 'replica'@'172.17.0.3' IDENTIFIED WITH mysql_native_password BY '123qqq...A';

mysql> grant replication slave on *.* to replica@'172.17.0.3';

mysql> CHANGE MASTER TO MASTER_HOST='172.17.0.3',MASTER_USER='replica',MASTER_PASSWORD='123qqq...A',MASTER_LOG_FILE='mysql-bin.000005',MASTER_LOG_POS=154;



mysql> show master status\G;
*************************** 1. row ***************************
				File: mysql-bin.000004
				Position: 154
				Binlog_Do_DB: 
				Binlog_Ignore_DB: 
				Executed_Gtid_Set: 



mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.17.0.3
                  Master_User: replica
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000005
          Read_Master_Log_Pos: 867
               Relay_Log_File: 32491ad8d234-relay-bin.000002
                Relay_Log_Pos: 1033
        Relay_Master_Log_File: mysql-bin.000005
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
```

<br/><br/>
**backup(172.17.0.3)机器操作**
```
[root@centos7 ~]# cat /etc/mysql/my.cnf

bind-address = 172.17.0.3
server-id = 2
log_bin = mysql-bin        #开启binglog日志
```

```
mysql> CREATE USER 'replica'@'172.17.0.2' identified by '123qqq...A';


mysql> ALTER USER 'replica'@'172.17.0.2' IDENTIFIED WITH mysql_native_password BY '123qqq...A';


mysql> grant replication slave on *.* to replica@'172.17.0.2';

mysql> CHANGE MASTER TO MASTER_HOST='172.17.0.2',MASTER_USER='replica',MASTER_PASSWORD='123qqq...A',MASTER_LOG_FILE='mysql-bin.000004',MASTER_LOG_POS=154;

mysql> show master status\G;
*************************** 1. row ***************************
				File: mysql-bin.000005
				Position: 154
				Binlog_Do_DB: 
				Binlog_Ignore_DB: 
				Executed_Gtid_Set: 




mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 172.17.0.2
                  Master_User: replica
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000004
          Read_Master_Log_Pos: 1115
               Relay_Log_File: 573082ac5d71-relay-bin.000002
                Relay_Log_Pos: 1281
        Relay_Master_Log_File: mysql-bin.000004
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes

```


<br/><br/><br/><br/>
#### 2.1 两台机器安装keepalived

```
[root@centos7 ~]# yum -y install keepalived ipvsadm
```

**master(172.17.0.2)机器操作**
```
[root@centos7 ~]# cat /etc/keepalived/keepalived.conf

! Configuration File for keepalived

global_defs {
   router_id host1
}

vrrp_script chk_mysql {
    script "/etc/keepalived/check_mysql.sh" 
    interval 2       
    weight 2           
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0             
    virtual_router_id 54    
    priority 100                  
    advert_int 1            
    nopreempt
    authentication {        
        auth_type PASS      
        auth_pass 1111
    }
    virtual_ipaddress {
        172.17.0.100
    }   
    track_script {
        chk_mysql      
    }   
}
```

```
[root@centos7 ~]# cat /etc/keepalived/check_mysql.sh

#!/bin/bash
if [ "$(netstat -nutlp | grep 3306)" == "" ];then
    #echo 1
    systemctl start mysqld &
    sleep 5

    if [ "$(netstat -nutlp | grep 3306 )" == "" ];then
        systemctl stop keepalived
        #echo 2
    fi
fi
```
<br/><br/>
**backup(172.17.0.3)机器操作**
```
[root@centos7 ~]# cat /etc/keepalived/keepalived.conf
! Configuration File for keepalived

global_defs {
   router_id host2
}

vrrp_script chk_mysql {
    script "/etc/keepalived/check_mysql.sh" 
    interval 2       
    weight 2           
}

vrrp_instance VI_1 {
    state BACKUP
    interface eth0             
    virtual_router_id 54    
    priority 50                  
    advert_int 1            
    nopreempt
    authentication {        
        auth_type PASS      
        auth_pass 1111
    }
    virtual_ipaddress {
        172.17.0.100
    }   
    track_script {
        chk_mysql      
    }   
}

```

```
[root@centos7 ~]# cat /etc/keepalived/check_mysql.sh

#!/bin/bash
if [ "$(netstat -nutlp | grep 3306)" == "" ];then
    #echo 1
    systemctl start mysqld &
    sleep 5

    if [ "$(netstat -nutlp | grep 3306 )" == "" ];then
        systemctl stop keepalived
        #echo 2
    fi
fi
```


<br/><br/><br/>
#### 2.2 测试脚本使用正常运行

```
[root@centos7 ~]# systemctl stop mysqld
[root@centos7 ~]# systemctl stop keepalived
```
