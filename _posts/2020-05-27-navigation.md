---
layout:     post   				    # 使用的布局（不需要改）
title:      bootstrap导航栏 				# 标题 
subtitle:   快速布置导航栏 #副标题
date:       2020-05-27 				# 时间
author:     Derrick 				# 作者
header-img: img/post-bg-navigation.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - Html
    - BootStrap
---

## bootstrap导航栏.nav和.navbar



**`.justify-content-center`类设置导航居中显示，`.justify-content-end`类设置导航右对齐。
选项卡`.nav-tabs`,胶囊导航`.nav-pills`;当屏幕缩小时，会折叠（.nav导航不会折叠）；使用 `class .nav、.nav-pills` 的同时使用 class .nav-stacked，让胶囊垂直堆叠。使用 `.nav、.nav-tabs` 或 `.nav、.nav-pills` 的同时使用 `class .nav-justified`，让标签式或胶囊式导航菜单与父元素等宽。**



```html
	<html>
	<head>
		<meta charset="utf-8"> 
		<title>Bootstrap导航菜单</title>
		<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
		<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
		<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
	</head>


	<body>
	<p>标签式的导航菜单</p>
	<ul class="nav nav-tabs">
		<li class="active"><a href="#">Home</a></li>
		<li><a href="#">SVN</a></li>
		<li><a href="#">iOS</a></li>
		<li><a href="#">VB.Net</a></li>
		<li><a href="#">Java</a></li>
		<li><a href="#">PHP</a></li>
	</ul>

	</body>
	</html>
```
 


**图示**
![img](/img/2020-05-27-navigation-2020/nav.png)




**添加 `role="navigation"`，有助于增加可访问性,向 <div> 元素添加一个标题 `class .navbar-header`，内部包含了带有 `class navbar-brand` 的 <a> 元素。这会让文本看起来更大一号。添加`.navbar-inverse`s 使导航栏颜色变为黑色**

```html
	<!DOCTYPE html>
	<html>
	<head>
		<meta charset="utf-8">
		<title>Bootstrap 实例 - 默认的导航栏</title>
		<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css ">
		<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
		<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
	</head>
	<body>

	<nav class="navbar navbar-default" role="navigation">
		<div class="container-fluid">
		<div class="navbar-header">
			<a class="navbar-brand" href="#">菜鸟教程</a>
		</div>
		<div>
			<ul class="nav navbar-nav">
				<li class="active"><a href="#">iOS</a></li>
				<li><a href="#">SVN</a></li>
				<li class="dropdown">
					<a href="#" class="dropdown-toggle" data-toggle="dropdown">
						Java
						<b class="caret"></b>
					</a>
					<ul class="dropdown-menu">
						<li><a href="#">jmeter</a></li>
						<li><a href="#">EJB</a></li>
						<li><a href="#">Jasper Report</a></li>
						<li class="divider"></li>
						<li><a href="#">分离的链接</a></li>
						<li class="divider"></li>
						<li><a href="#">另一个分离的链接</a></li>
					</ul>
				</li>
			</ul>
		</div>
		</div>
	</nav>

	</body>
	</html>

```



**图示**
![img](/img/2020-05-27-navigation-2020/navbar.png)