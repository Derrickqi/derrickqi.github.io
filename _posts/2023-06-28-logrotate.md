---
layout:     post   				    # 使用的布局（不需要改）
title:      利用logrotate切割nginx日志 		# 标题 
subtitle:   Hello World, Hello Blog #副标题
date:       2023-06-28 				# 时间
author:     Derrick 				# 作者
header-img: img/post-bg-logrotate.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - linux
---

### 使用系统自带的logrotate切割日志



```shell
# cat /etc/logrotate.d/nginx

/usr/local/nginx/logs/*log {
    size 1G   //文件大小超过1G则切割日志
    create 0664 nginx root    //创建并赋予权限
    rotate 3      //最大保留3个
    dateext     //割后的日志文件以当前日期为格式结尾
    dateformat -%Y%m%d_%s     //时间戳精确到秒
    compress    //通过gzip压缩转储以后的日志文件
    missingok    //如果日志丢失，不报错继续滚动下一个日志
    notifempty    //当日志文件为空时，不进行轮转
    sharedscripts 
    postrotate
        /bin/kill -USR1 `cat /usr/local/nginx/logs/nginx.pid 2>/dev/null` 2>/dev/null || true
    endscript
}

```


<br/><br/> 
### logrotate结合crotab对nginx日志进下切割存储



```shell
//crontab -l 

59 * * * * logrotate /etc/logrotate.d/nginx
//每59分钟执行检查一次nginx日志
```


## logrotate配置常用参数
```
compress                                   通过gzip 压缩转储以后的日志
nocompress                                不做gzip压缩处理
copytruncate                              用于还在打开中的日志文件，把当前日志备份并截断；是先拷贝再清空的方式，拷贝和清空之间有一个时间差，可能会丢失部分日志数据。
nocopytruncate                           备份日志文件不过不截断
create mode owner group             轮转时指定创建新文件的属性，如create 0777 nobody nobody
nocreate                                    不建立新的日志文件
delaycompress                           和compress 一起使用时，转储的日志文件到下一次转储时才压缩
nodelaycompress                        覆盖 delaycompress 选项，转储同时压缩。
missingok                                 如果日志丢失，不报错继续滚动下一个日志
errors address                           专储时的错误信息发送到指定的Email 地址
ifempty                                    即使日志文件为空文件也做轮转，这个是logrotate的缺省选项。
notifempty                               当日志文件为空时，不进行轮转
mail address                             把转储的日志文件发送到指定的E-mail 地址
nomail                                     转储时不发送日志文件
olddir directory                         转储后的日志文件放入指定的目录，必须和当前日志文件在同一个文件系统
noolddir                                   转储后的日志文件和当前日志文件放在同一个目录下
sharedscripts                           运行postrotate脚本，作用是在所有日志都轮转后统一执行一次脚本。如果没有配置这个，那么每个日志轮转后都会执行一次脚本
prerotate                                 在logrotate转储之前需要执行的指令，例如修改文件的属性等动作；必须独立成行
postrotate                               在logrotate转储之后需要执行的指令，例如重新启动 (kill -HUP) 某个服务！必须独立成行
daily                                       指定转储周期为每天
weekly                                    指定转储周期为每周
monthly                                  指定转储周期为每月
rotate count                            指定日志文件删除之前转储的次数，0 指没有备份，5 指保留5 个备份
dateext                                  使用当期日期作为命名格式
dateformat .%s                       配合dateext使用，紧跟在下一行出现，定义文件切割后的文件名，必须配合dateext使用，只支持 %Y %m %d %s 这四个参数
size(或minsize) log-size            当日志文件到达指定的大小时才转储，log-size能指定bytes(缺省)及KB (sizek)或MB(sizem).
当日志文件 >= log-size 的时候就转储。 以下为合法格式：（其他格式的单位大小写没有试过）
size = 5 或 size 5 （>= 5 个字节就转储）
size = 100k 或 size 100k
size = 100M 或 size 100M

```


<br/><br/>
### 注意
`注意:` ***logrotate的最小执行时间为每天执行一次，即使在配置文件中添加size参数 也是按照每天一次切割，需要结合脚本和crontab定时任务缩短切割周期，根据size去切割日志***

