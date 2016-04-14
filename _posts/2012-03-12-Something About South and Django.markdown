---
layout:     post
title:      "Something about South and Django"
date:       2012-03-12 18:37
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - django
    - python
---

### 1 

如果你已经安装并使用了South来管理某个Django App，这时候你执行 **python manage.py syncdb** 命令，Django默认不会为该App创建对应的表，除非你将他们加入到South中在用South进行生成，或者将之移除到South的接管范围之外。

### 2

如果你的站点的生产环境使用SSL，而测试环境没有使用SSL的话，应该将**SESSION_COOKIE_SECURE**变量设置在生产环境和测试环境分开，生产环境设置为**True**，测试环境设置为**False**。如果在非SSL环境设置**SESSION_COOKIE_SECURE = True**会使Django Admin无法登录，即使输入正确的用户名和密码，页面会直接重定向会admin首页，不会有任何信息输出。

	[Django官方的FAQ](https://docs.djangoproject.com/en/dev/faq/admin/#i-can-t-log-in-when-i-enter-a-valid-username-and-password-it-just-brings-up-the-login-page-again-with-no-error-messages)有提到这个现象（一模一样的），但是他们说的原因于我却不适用，官方的说法是 **SESSION_COOKIE_DOMAIN** 设置引起的（官方意指该值可能设置不正确或者某些浏览器无法识别不带"."的域名，例如localhost），显然这里与我的问题的唯一关联就是确实是和cookie相关。
	
	**SESSION_COOKIE_SECURE = True**变量的实际作用是确保会话cookie只通过https协议进行传输，因此要生效则必须是服务器支持SSL的情况下才行。
	
### 3

对于已经存在数据库，但是之前没有使用south的，现在想使用south，在开发环境中执行**python manage.py convert_to_south myapp**一次之后就不需要再执行了，提交到版本库后，应用在服务端（或者是本地开发环境）部署的时候就不需要再执行这个命令的了。换言之，这是开发阶段的一条命令，部署阶（即使是本地部署）段需要执行的是**python manage.py migrate myapp 0001 --fake**，否则做migrete的时候会报类似**django.db.utils.DatabaseError: table "xxxs_xxx" already exists**的错误，系统报表已存在，无法执行migrate动作。

convert和migrate --fake这两条命令只需要执行一次，之后每次改动models，执行以下两条命令即可：
    
```bash 
$ python manage.py schemamigration myapp –-auto
$ python manage.py migrate myapp
```
    
第一条命令创建一个新的migration，系统根据实际情况可能会要你做一些选择，例如你定义了一个not null的字段却又没有给default值，那么系统就会让你选择终止migrate或者在当前给定一个default值。第二条命令使之真正应用到数据库中生效。

有一篇比较完整的South执行步骤的整理可以参见[这篇文章](http://mitchfournier.com/2010/06/23/getting-started-with-south-django-database-migrations/)
	
对于之前带数据库但是没有使用south的应用需要转化到使用South，可以看看[官方文档](http://south.aeracode.org/docs/convertinganapp.html#converting-an-app)。
	
### 4

对于已经使用了South的工程，如果要多增加一个app，有两种方式，效果是一样的：

#### 4.1. 创建好myapp2并在models.py中定义好模型之后：
	
```bash
$ manage.py syncdb
$ manage.py convert_to_south myapp2
```

第一条命令生成myapp2对应的数据表结构。然后再第二条命令将之转化到south来管理。
	
#### 4.2. 创建好myapp2并在models.py中定义好模型之后：
	
```bash
$ manage.py schemamigration myapp2 --initial
$ python manage.py migrate myapp2
```

第一条命令直接将新的app的表结构使用South管理，然后第二条命令让South来初始化创建表结构。

### 5

模型中的OneToOneField()、ForeignKey()和ManyToManyField():

OneToOneField相当于设置了**unique=True**的ForeignKey，在对象模型上相当于继承关系。并且反向引用的时候你可以得到一个父类对象。
	
用User Profile为例来说：
	
```python
	
class UserProfile(BaseModel):	user = models.OneToOneField(User)
	nick_name = models.CharField(verbose_name='昵称', \ 
		max_length=64,blank=True, null=True)
```
			
OneToOneField使得每个User都能拥有一个get_user_profile()方法，并且每个UserProfile对象都有一个get_the_user()方法。这也是它和ForeignKey的区别之一。
	
ForeignKey（外键）其实相当于*Many-To-One-Field*，是一个多对一的关系，当然也可以通过设置其中的**unique=True**参数使它成为一个一对一关系。在扩展UserProfile时官方文档使用的是**user = models.OneToOneField(User)**，但实际上我们使用**user = models.ForeignKey(User, unique=True)**也是一样的效果。
	
而针对ManyToManyField关系，Django会为我们创建一张额外的关系表来维护，这段对开发者实际上是透明的。
	