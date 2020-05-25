---
layout:     post   				    # 使用的布局（不需要改）
title:      初识Django 				# 标题 
subtitle:   Django的初始化配置       #副标题
date:       2020-05-24 				# 时间
author:     Derrick 				# 作者
header-img: img/post-bg-Django.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - Python
    - Django
---

## Django的初始化配置


**创建项目文件夹**

```python
    django-admin startproject ProjectName
```

**创建App**

```python
    python manage.py startapp AppName
```

**修改setting.py**
在 INSTALLED_APPS的列表中，添加创建的App名字
```python
    mysite/setting.py
    
    INSTALLED_APPS = [
        'AppName',
        ...
```

**修改TEMPLATES中的DIRS，以将前端文件统一放在根目录下的templates中。**
```python
    mysite/setting.py

    TEMPLATES = [
        {
            'BACKEND': 'django.template.backends.django.DjangoTemplates',
            'DIRS': [os.path.join(BASE_DIR, "templates")],
            ...

```

**在最后添加静态文件配置**
```python
    mysite/setting.py

    STATIC_URL = '/static/'
    STATICFILES_DIRS = [
        os.path.join(BASE_DIR, 'static')
    ]
```

**修改项目urls.py**
`官方文档的内容，如果在urls.py中使用到正则匹配路径（^$）的时候，就需要使用re_path,而不能使用path，不然页面会显示404错误,如果未用到正则，那么使用path即可。`
```python
    mysite/urls.py

    from django.contrib import admin
    from django.urls import path, include, re_path
    #匹配路由
    urlpatterns = [
        path('admin/', admin.site.urls),
        re_path('',include('devops.urls')),
    ]

```

**App下新建urls.py并初始化**
```python
    devops/urls.py

    from django.urls import path 
    from . import views
    
    app_name = 'AppName'
    
    urlpatterns = [
        path('index/', views.index, name="index"),
    ]
```