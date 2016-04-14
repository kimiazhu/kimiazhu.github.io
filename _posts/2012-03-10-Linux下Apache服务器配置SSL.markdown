---
layout:     post
title:      "Linux下Apache服务器配置SSL"
date:       2012-03-10 13:25
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Linux
    - Apache
    - SSL
    - HTTPS
---
# 一、生成证书文件

### 1. 生成服务器用的私钥文件test_server.key 

进入/etc/httpd/conf目录，执行命令行[^1]
  
	$ openssl genrsa -out test_server.key 1024
  
### 2. 生成未签署的test_server.csr 

进入/etc/httpd/conf目录，执行命令行
  
	$ openssl req -new -key server.key -out test_server.csr -config /etc/pki/tls/openssl.cnf 
  
提示输入一系列的参数 [^2]
  
	  ...... 
	  Country Name (2 letter code) [AU]: CN
	  State or Province Name (full name) [Some-State]: Guangdong
	  Locality Name (eg, city) []: Shenzhen
	  Organization Name (eg, company) [Internet Widgits Pty Ltd]: Test Inc.
	  Organizational Unit Name (eg, section) []: Test Inc.
	  Common Name (eg, YOUR name) []: test-server.com
	  Email Address []: me@test-server.com
	  ..... 
  

### 3. 签署服务器证书文件test_server.crt 

  进入/etc/httpd/conf目录，执行命令行：
  
	$ openssl x509 -req -days 365 -in test_server.csr -signkey test_server.key -out  test_server.crt 
  
### 4. 生成IE可用的证书

生成可导入IE的证书(p12格式)，以便在IE下导入该证书后IE能信任该HTTPS连接[^3]：

	$ cat test_server.crt test_server.key > test_server.pem
	$ openssl pkcs12 -export -in test_server.pem -out test_server.p12

以上签署证书仅仅做测试用，真正运行的时候，应该将CSR发送到一个CA返回真正的用书.网上有些文档描述生成证书文件的过程比较繁琐，就是因为他们自己建立了一个CA中心证书，然后再签署test_server.csr. 
  
用以下命令可以查看证书的内容。证书实际上包含了Public Key.
  
	$ openssl x509 -noout -text -in test_server.crt


---


# 二、配置ssl虚拟主机


1. 确保apache相关模块已经安装

2. /etc/httpd/conf/http.conf文件不用改动，可以维持原有的http形式的服务可以访问

3. /etc/httpd/conf.d/ssl.conf文件,注释掉原来关于虚拟主机的配置，新增以下配置:

```config

<VirtualHost test_server:443>	SSLEngine On   SSLCertificateFile /etc/httpd/conf.d/cert/test_server.crt	SSLCertificateKeyFile /etc/httpd/conf.d/cert/test_server.key	SetEnvIf User-Agent ".*MSIE.*" \	     nokeepalive ssl-unclean-shutdown \	     downgrade-1.0 force-response-1.0	DocumentRoot "/var/www/html/project_folder"	ServerName test_server.com	<Location "/">	    SetHandler python-program	    PythonHandler django.core.handlers.modpython	    SetEnv DJANGO_SETTINGS_MODULE project_folder.settings	    PythonOption django.root /var/www/html/project_folder/	    PythonPath "['/var/www/html/project_folder/', '/usr/lib/python2.7/site-packages/django/'] + sys.path"	    PythonDebug off	</Location>   <Location "/static/">	    SetHandler None	</Location>
</VirtualHost> 
```[^1]: 有文档指出使用**openssl genrsa -des3 -out test_server.key 1024** 生成私钥文件，这样生成的私钥文件是需要口令的。 Apache启动失败，错误提示是：**Init: SSLPassPhraseDialog builtin is not supported on Win32 (key file …..)**原因是window下的apache不支持加密的私钥文件。
[^2]: Common Name必须和httpd.conf中server name必须一致，否则apache不能启动。启动apache时错误提示为：**RSA server certificate CommonName (CN) `Koda' does NOT match server name!?**


[^3]: 其他相关命令请[参照这里](http://www.iteye.com/topic/584927)