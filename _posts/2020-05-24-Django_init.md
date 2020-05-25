---
layout:     post   				    # 使用的布局（不需要改）
title:      初识Django 				# 标题 
subtitle:   Django的初始化配置       #副标题
date:       2020-05-24 				# 时间
author:     Derrick 				# 作者
header-img: img/post-bg-travel-4.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - Python
    - Django
---

## Django的初始化配置


**创建项目文件夹**

`
    django-admin startproject ProjectName
`

**创建App**

`
    python manage.py startapp AppName
`

**修改setting.py**
在 INSTALLED_APPS的列表中，添加创建的App名字

`
    INSTALLED_APPS = [
        'AppName',
        ...
`

**修改TEMPLATES中的DIRS，以将前端文件统一放在根目录下的templates中。**

`
    TEMPLATES = [
        {
            'BACKEND': 'django.template.backends.django.DjangoTemplates',
            'DIRS': [os.path.join(BASE_DIR, "templates")],
            ...
`

**在最后添加静态文件配置**

`
    STATIC_URL = '/static/'
    STATICFILES_DIRS = [
        os.path.join(BASE_DIR, 'static')
    ]
`

**修改项目urls.py**
`官方文档的内容，如果在urls.py中使用到正则匹配路径（^$）的时候，就需要使用re_path,而不能使用path，不然页面会显示404错误,如果未用到正则，那么使用path即可。`


`
    from django.contrib import admin
    from django.urls import path, include, re_path
    ;匹配路由
    urlpatterns = [
        path('admin/', admin.site.urls),
        re_path('',include('devops.urls')),
    ]
`

**App下新建urls.py并初始化**

`
    from django.urls import path 
    from . import views
    ；设置APP_name
    app_name = 'AppName'
    ;配置主路由
    urlpatterns = [
        path('index/', views.index, name="index"),
    ]
`

**修改App下views.py**

`
    from django.shortcuts import render
    # Create your views here.
    ；
    def index(request):
        return render(request, "index.html", locals())
`

**新建index.html**
`
    {% load staticfiles %}
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <link rel="stylesheet" type="text/css" href="{% static 'css/test.css' %}">
        <script src="{% static 'js/test.js' %}"></script>   
        <title>Document</title>
    </head>
    <body>
        Hello World
        <a href="{% url 'speech:index' %}" > test </a>
        <img src="{% static 'images/test.jpg' %}" />
    </body>
    </html>
`

**设计和表对应得类**

`
    from django.db import models
    ；
    ；
    # 设计和表对应得类
    # Create your models here.
    ；
    class UserInfo(models.Model):
        gender = (
            ('male', '男'),
            ('female', '女'),
        )
        ；
        """用户模型类"""
        # CharField说明一个字符串， max_length指定字符串最大长度
        username = models.CharField(max_length=20, unique=True, verbose_name='用户名')
        # 密码长度
        password = models.CharField(max_length=20, verbose_name='密码')
        email = models.EmailField(unique=True, verbose_name='邮箱')
        sex = models.CharField(max_length=32, choices=gender, default='男', verbose_name='性别')
        c_time = models.DateTimeField(auto_now_add=True)
        ；
        ；
    class HostInfo(models.Model):
        ipstatus_choices = (
            (0, '空闲中'),
            (1, '使用中'),
        )
        ；
        ipaddr = models.CharField(max_length=20, verbose_name="IP地址")
        ipstatus = models.SmallIntegerField(choices=ipstatus_choices, default='空闲中', verbose_name="IP状态")
        comment = models.TextField(max_length=256, blank=True, null=True, verbose_name="备注")
        ；
`

**初始化数据库并创建Superuser**

`
    python manage.py migrate
    python manage.py makemigrations
    python manage.py createsuperuser
`

 自定义模型管理类

`
    from django.contrib import admin
    from .models import UserInfo, HostInfo
        ；
        ；
    # 后台管理相关文件
    # Register your models here.
        ；
    # 自定义模型管理类
    class UserInfoAdmin(admin.ModelAdmin):
        """用户模型管理类"""
        list_display = ['id', 'username', 'email', 'c_time']
        ；
        ；
    class HostInfoAdmin(admin.ModelAdmin):
        """服务器模型管理类"""
        list_display = ['id', 'ipaddr', 'ipstatus', 'comment']
        ；
        ；
    # 注册模型类
    admin.site.register(UserInfo, UserInfoAdmin)
    admin.site.register(HostInfo, HostInfoAdmin)
`
        ；
**运行测试**

`
    python manage.py runserver
`



