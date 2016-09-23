---
layout:     post
title:      "Django资料和开源App集合"
date:       2012-03-15 15:53
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Django
    - Python
---

*本文源于网络[《我和Django》](http://lds.osser.me/data/20110902224717/index.html)。我对其中部分明显有误的地方进行了更正，并对部分内容作了补充，补充内容大部分以斜体的形式直接插入文中，或者以注解的形式存在于文章末尾。*

---

我使用python的很大一部分原因就是django。虽然在以前也用过python，不过始终没有什么特别的感觉。然后接触到了django。可以说django非常对我的胃口，特别是他的admin给人的感觉特别的棒。

django是个独断且固执的框架，框架里用的组件都是自己写的，而且往往会“知错不改”。所以用django就要试着think in django，接受django所谓的设计哲学，如果接受不了那就换pylons或ROR什么的试试吧。

django并不完美，但这并不妨碍她成为一个优秀的web开发框架。

---

## 资源
* [django官网](http://www.djangoproject.com/)
* [django文档](http://docs.djangoproject.com/en/dev/)
* [Django Step by Step](http://www.woodpecker.org.cn/share/projects/django/django-stepbystep/newtest/doc/)曾是最佳的django入门教程，只是这个教程是针对0.95的，现在不少地方已经有所变动。注：这里有个基于Django1.1的[2010版](http://bbs.quickbest.com.cn/thread-61139-1-1.html)，看样子似乎是转帖，不知道原帖地址。
* [django可复用app设计](http://ericholscher.com/projects/reusable-app-docs/)
* [django最佳实践](http://github.com/lincolnloop/django-best-practices/tree/master) django可复用app设计 的一个更好的分支。个人为人这个文档是每个django开发人员必读的。
* [django最佳实践](http://yangyubo.com/django-best-practices/) 中文翻译
* [django book](http://www.djangobook.com/) 一本免费的django电子书
* [django book 中文翻译](http://djangobook.py3k.cn/)
* [djangosnippets](http://www.djangosnippets.org/) 一个关于django的代码片段网站，在里面可以找到一些应用的django代码片段。站点本身是用django写的，且开发源码。
* [djangosites](http://www.djangosites.org/) 这个网站里介绍了很多实用django搭建的站点。而且可以从这里找到很多带源代码的django站点。不过我个人觉得里面没有多少值得参考的站点代码。
* [DjangoCon](http://djangocon.us/) 里面可以找到一些不错的技术主题
* [Django Packages](http://www.djangopackages.com/) 帮助寻找Django可服用组件的网站。将可复用组件进行归类，并以表格的形式进行对比。

---

## 使用django搭建的站点
* [addons.mozilla.org](https://addons.mozilla.org/) FireFox的插件下载网站，从google的统计数据上看PV大概在douban一半的样子。技术细节方面可以看他们的[幻灯片](http://www.slideshare.net/jbalogh/djangocon)。该网站完全开源，代码可以在[这里](http://github.com/jbalogh/zamboni)找到。

* [disqus](http://disqus.com/) 这个网站在国内见得不多，可能很多人对它不太清楚。这个网站为其他网站增加评论功能。据其介绍，该网站每月有1.2亿的独立访问者。技术细节参考他们的[幻灯片](http://www.slideshare.net/zeeg/djangocon-2010-scaling-disqus).
 
* [bitbucket](https://bitbucket.org/) 基于HG的代码托管网站。
 
* [django官网](http://www.djangoproject.com/) django官网本身使用django搭建，而其提供了代码。django官网的大多功能由整合的trac实现，所以网站的django代码没几行。
 
* [海报网](http://www.haibao.cn/) 据说这是目前流量最大的django站点。据我的了解，这个网站的流量也确实大的有些超乎想象，该网站应当有接近CSDN的流量。不过这个网站将大量的页面进行了静态化，如果以这个网站的标准来评价django的性能应当不够客观。
 
* [好看簿](http://haokanbu.com/) 国内的另一个django站点，目前也有不错的流量。作为该网站的用户，我感觉网站的响应速度并不怎么快。看来好看簿在服务器优化方面还需要更多的努力。
 
* [Instagram](http://instagr.am/) 在短时间内迅速崛起的一个iPhone应用，用户增长的非常快。从技术人员的角度看，instagram的迷人之处是在不足10人的情况下，服务了万用户。在这篇文章[What Powers Instagram: Hundreds of Instances, Dozens of Technologies](http://instagram-engineering.tumblr.com/post/13649370142/what-powers-instagram-hundreds-of-instances-dozens-of) [^1]里，有介绍他们所用到的一些技术。
 
* *[扇贝网](http://www.shanbay.com/) 是一个帮助背诵单词的站点。*

---

## django的开源项目
* [pinax](http://pinaxproject.com/) 这是我看到的最有价值的django开源项目。pinax可以看做是django的一个脚手架。她提供了快速开始一个新django项目的方法，同时对大量第三方app的使用方法进行了演示。django的app质量参差不齐，如果你想挑选app，那你可以看看pinax里都集成了哪些app。pinax里集成了的app通常都不至于太烂。此外pinax自身也带了一些有用的app，比如blog等。
 
 	如果你想以最快的速度了解pinax，可以去 [cloud27](http://cloud27.com/) 看看。这是一个用pinax搭建的SNS网站。
 	
* [Satchmo](http://www.satchmoproject.com/) 网店系统。看她的介绍，似乎已经有不少人在用这东西了。
 
* [LFS(Lightning Fast Shop)](http://www.getlfs.com/) 网店系统，就Demo来看似乎是倾向于房屋交易平台。陆陆续续的也有部分商业网站开始使用该系统了，比如[这个](http://www.terrassenueberdachung-bearcounty.de/)。
 
* [Reviewboard](http://code.google.com/p/reviewboard/) 非常有前途的一个code review工具。最开始是[VMware](http://www.vmware.com/)在用，来后给开源了。

---

## django的可重用APP
[Django Packages](http://www.djangopackages.com/) 这个网站将可复用组件进行归类，并以表格的形式进行对比。如果你想找Django可重用APP，去这个网站是最方便的。我这里只对我认为最优秀的Django APP进行整理。

### 项目组织
django没有统一的项目组织规范，所以django项目的目录组织方式都各不相同。为解决该问题，也出现了一些相关项目。

* [dj-scaffold](https://github.com/vicalloy/dj-scaffold) 我的django脚手架项目。提供命令dj-scaffold.py，用于生成一个基础的django项目模板。
 
* [django-startproject](https://github.com/lincolnloop/django-startproject) 也是用于生成项目模板的项目。我的不少代码都是参考这个项目的。
 
* [playdoh](https://github.com/mozilla/playdoh) 顶着mozilla的名头，应当还是值得一看的吧。不过他的目录组织方式不太符合我的习惯的。

### CMS
* [Django CMS](https://www.django-cms.org/) Django CMS与其说是一个APP，倒不如说这是一个框架。Django CMS是目前开源Django CMS中功能最为完善的一个。
 
 	Django CMS提供了插件接口，可以方便的以插件的方式进行扩展。此外，目前现成可用的插件也已经有一大堆了。

### Forum
Django的论坛APP不少，但到就目前而言，还没有什么杀手级的APP。

* [DjangoBB](https://bitbucket.org/slav0nic/djangobb) 功能比较完整。不过我认为搞的有些复杂了，易用性一般。如果你贪图它相对强大的功能，又不怕麻烦的话，可以试试。
 
* [LBForum](http://githubx.com/vicalloy/LBForum) 我开发的论坛应用。优点是界面漂亮（提供了[FluxBB](http://fluxbb.org/)和[V2EX](http://v2ex.com)两种界面风格），部署简单，功能方面就不怎么强大了。如果你想要一个简单易用的Django论坛系统，推荐这个。

### Blog
用Django写Blog数量众多（可能是数量最多的Django应用了），我虽然也写了一个，但我是不会去用这些Django博客。Blog很重要的一点是那些漂亮的模板。如果使用这些小众的东西，实在是难以找到让人满意的模板。

* [zinnia](https://github.com/Fantomas42/django-blog-zinnia) 功能比较完善的一个Django博客，界面比较清爽。简单的看了一下她的代码，感觉写的很规范。比较看好这个博客系统。如果你想用django搭建自己的博客，推荐试试。
 
### 调试

* [django-debug-toolbar](http://github.com/robhudson/django-debug-toolbar/tree/master) 为django站点增加调试功能，支持查看django生成的sql语句，及sql的执行时间等，功能强大。不过由于该组件使用了jquery，似乎会使用部分使用了jquery的站点无法正常工作。 
 
* [django-sentry](https://github.com/dcramer/django-sentry) [disqus](http://disqus.com/)的开源项目。将django的所有异常保存到数据库，并提供异常的察看界面。
 
* [django-devserver](https://github.com/dcramer/django-devserver) （*注：现在这个页面已经移动到了[这里](https://github.com/dcramer/sentry)*） django开发服务器扩展。将SQL语句/执行时间等调试信息直接显示在控制台上，而且是以彩色的方式显示。[^2]

### 数据库升级[^3]
在项目开发过程中表结构的变动总是难免，django目前还不支持表结构的自动更新，不过相关的第三方app倒不少。

* [South](http://south.aeracode.org/) South已经比较成熟了，就目前而言South是该类APP的不二选择。

### 注册、认证

* [Django-Socialauth](https://github.com/agiliq/Django-Socialauth) 支持使用Facebook, Yahoo, Gmail, Twitter and Openid的帐号进行登陆认证。
 
* [django-socialregistration](https://github.com/flashingpumpkin/django-socialregistration) 支持OpenID, OAuth and Facebook的认证。似乎和Django-Socialauth差不多。没有对比过，希望用过的朋友给些心得。[^4]
 
* [django-registration](https://github.com/ubernostrum/django-registration) 注册功能，支持帐户的邮件激活。该项目似乎已经停止维护了。可作为参考项目，不太建议在新项目中使用了。
 
* [django-auth-ldap](http://bitbucket.org/psagers/django-auth-ldap/) Django的LDAP认证支持，使用LDAP的集成变得简单。

### Tagging
为站点增加Tag功能

* [django-taggit](http://github.com/alex/django-taggit) 取代[django-tagging](http://code.google.com/p/django-tagging/)，成为Django的首选Tagging APP。不过我对这个APP的性能始终有所顾虑。

### Avatar（用户头像）
* [django-avatar](https://github.com/ericflo/django-avatar) 当前首选。感觉复杂了些，而且我觉得支持多个头像啥的功能不是很实用，还增加了复杂度。

* [django-simple-avatar](https://github.com/vicalloy/django-simple-avatar) 我自己写的avatar APP，其中的不少代码来源于django-avatar。用起来比django-avatar要简单些。

### 翻页

* [django-pagination](http://code.google.com/p/django-pagination/) 一组翻页相关的utils，包括用于实现翻页的tag等。使用起来非常简单。是目前使用最多的分页APP。

* [django-paging](https://github.com/dcramer/django-paging) 另一个翻页的APP，优点是支持jinja2作为模板。如果模板用了jinja2，可以考虑下。

### 搜索

* [Haystack](http://haystacksearch.org/) 全文搜索组件，提供对[Solr](http://lucene.apache.org/solr)、[Whoosh](http://whoosh.ca/)、[Xapian](http://xapian.org/)的支持。就它的quick start来看是挺易用的。该项目托管在github，似乎还挺有人气。

### RESTful

* [django-piston](https://bitbucket.org/jespern/django-piston) 用来写RESTful API的东西，据说很方便。未使用过，不多做评论。

### 消息队列（异步执行）

* [django-celery](https://github.com/ask/django-celery) web应用中难免会有些很费时的操作需要作成异步处理（比如在后台发送邮件，更新索引等），django-celery就是为解决该问题出现的。

### 其他

* [django-extensions](https://github.com/django-extensions/django-extensions) django一些扩展的集合。东西比较杂，具体使用还是去看看她的文档吧。

### *Dajngo主机服务*

* *[Google App Engine](https://appengine.google.com/) 本质上来说App Engine并不**直接**支持Django框架， 但是你还是能找到一些稍微绕绕弯子的方法，稍微修改一下Django或者找一个定制过的Django即可，我之前的[博客](http://me.kimiazhu.info/)即使使用过定制后的Django搭建，不过这个过程中可能还需要你煞费苦心为这个定制版找找Bug。具体方式可以参考[Running Django on Google App Engine](http://code.google.com/appengine/articles/django.html)，以及[google-app-engine-django](http://code.google.com/p/google-app-engine-django/)等。*

* *[ep.io](http://www.ep.io) 这个服务提供者之一也是South框架的作者，除了Django之外，它还支持Flask、WSGI应用。[ep.io](http://www.ep.io)目前（2012年3月19日）仍处于邀请测试阶段。*

* *[新浪App Engine](http://sae.sina.com.cn/) 对Python的支持目前（2012年3月19日）还在测试中，至于Django还不得而知。不过此服务在国内，访问速度应该不错。但是由于政策关系，新浪App Engine并不提供针对个人的自定义域名服务。*



[^1]: 注：[这里](http://www.dbanotes.net/arch/instagram.html)有该文章的一篇中文笔记。

[^2]: 注：django-sentry同时也被认为是一个日志和监控用的App44，Instagram团队和扇贝网也把它用在生产环境。它分成客户端和服务器，客户端部署在各个应用服务器上，用来收集错误信息，再发送到服务器（一个独立的Http服务器）进行分析和跟踪展示。

[^3]: 注：数据库升级方案实际上不只有South，其他的可选方案还包括[django-evolution](http://code.google.com/p/django-evolution/)和[dmigrations](http://code.google.com/p/dmigrations/)，它们各有优劣，不过目前最流行，也是即将可能被Django官方采纳的方案是South。[South官方文档](http://south.aeracode.org/docs/)中对这几种方案做了个详细的[比较](http://south.aeracode.org/wiki/Alternatives)。

[^4]: 注：从所支持的服务数量和开发活跃度来看，还是推荐使用django-socialregistration