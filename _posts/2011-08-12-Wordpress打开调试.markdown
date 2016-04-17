---
layout:     post
title:      "Wordpress打开调试"
date:       2011-08-12 15:53
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - PHP
    - Wordpress
---

极少涉及PHP，留个笔记。

如何打开Worpress调试功能，让系统记录错误日志？找了很久，最终参考了[这篇文章](http://codex.wordpress.org/Editing_wp-config.php)。

1. wp-config.php文件中，打开一个调试选项

`PHP Code:`
	
	define('WP_DEBUG', false);
修改为：

`PHP Code:`
	define('WP_DEBUG', true);
可以打开调试模式

2. wp-settings.php 中打开日志并指定日志文件：

`PHP Code:`

    @ini_set('log_errors','On');
    @ini_set('display_errors','Off');
    @ini_set('error_log','/var/www/html/test.com/logs/error.log');
    
    
需要注意可能出现的权限问题，可以创建好目录再执行chmod赋予写权限，让系统能写日志。
 
应该就OK了，之后出问题页面上会直接打印堆栈信息，并且对应目录下的error.log也会记录。