---
layout:     post
title:      "修复Django Admin中的链接错误"
date:       2012-03-10 14:05
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Django
    - Python
---

## 问题：

1. Django Admin后台导航栏出现的文档、修改密码、注销等三个链接不正确，链接中出现了文件系统的绝对路径（/var/www/html这类的绝对路径）。

2. 每次访问后台，会引用到jsi18n的路径，可以在生成的后台页面HTML源代码中找到，这个链接也是不正确的

## 环境：

1. Apache 2服务器下部署的Django

2. Django1.3

3. Python 2.4/2.7下均有此问题

至于是否有其他配置项影响到它现在还未知，以下提供一种解决燃眉之急的方法。

## 修复

跟了下源代码和文档，这几个变量的使用其实历经了多次改变，目前的方式是使用反射获取App的路径信息，反射的出来的信息没有去掉本身工程在文件系统中的绝对路径的前缀，导致页面使用该值生成链接的时候出错了。但是出问题的地方太过底层，暂时没敢做太大动作去改动，怕影响到其他地方，因此选择在页面上直接改掉这个链接，因为也是不带域名的绝对路径，所以不会对开发、测试、生产环境的切换产生影响。

#### 修复导航栏错误

修改以下文件修正后台导航栏的链接错误：

/usr/lib/python2.4/site-packages/django/contrib/admin/templates/admin/base.html

1. 修改：
	
		<a href="{{ docsroot }}">...
	改成
	
		<a href="/admin/doc/">...

2. 修改：

		<a href="{{ password_change_url }}">...
	改成
	
		<a href="/admin/password_change/">...
		
3. 修改：

		<a href="{{ logout_url }}">...
	改成

		<a href="/admin/logout/">...


#### 修复jsi18n错误
以下三个文件需要修正jsi18n的链接错误：

1. /usr/lib/python2.4/site-packages/django/contrib/admin/templates/admin/auth/user/change_password.html

2. /usr/lib/python2.4/site-packages/django/contrib/admin/templates/admin/change_list.html

3. /usr/lib/python2.4/site-packages/django/contrib/admin/templates/admin/change_form.html

这三个文件中的jsi18n的引用全部改成/admin/jsi18n/