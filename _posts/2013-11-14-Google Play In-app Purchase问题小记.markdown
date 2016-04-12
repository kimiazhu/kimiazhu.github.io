---
layout:     post
title:      "Google Play In-app Purchase问题小记"
date:       2013-11-14 00:36
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Android
    - IAP
    - In-app products
    - Games
    - Google
    - Google Play
---

1. 至今中国区无法正常访问付费应用，也无法正常途径收款。

2. 看看Google的In-app支持代码，其实很有意义，做得比国内绝大多数的计费sdk更好，更安全，也在允许范围内给了足够大的定制空间（底线也维持住了，限制了开发者恶意吸费的口子）

3. 关于测试，注册在国内的账户肯定是不能看到付费应用的，我有一个注册在香港的账户也不行。但是另外一个注册在美国的账户可以，也可以购买东西。不需要挂VPN，至于绑定信用卡的时候，你的美国地址得自己想办法，建行招行的双币信用卡都没有问题。

4. Samples代码有一个问题，可能造成连续两次点击付费出现异常：

	*<font color="red">11-12 01:18:48.370: E/AndroidRuntime(9770): java.lang.IllegalStateException: Can't start async operation (launchPurchaseFlow) because another async operation(launchPurchaseFlow) is in progress.</font>*

	这个问题是在IabHelper.java类中抛出的，原因是上次付费未被标记为已结束，就开始下次计费，这里我们可以将flagEndAsync()方法标记为public，然后在自己的代码开始计费之前，先调用一次flagEndAsync()

5. 无法支付，在支付页面会出现如下提示：

	<code>
	This is a test order, you will not be charged.</br>
	*<font color="red">This payment method has been declined.</font>*
	</code>
 
	原因未知。该测试账户非Developer，已经添加到LICENSE TESTING中，并且可以正常购买市场上的其他App和In-app Products。而且此测试app也是用的正式签名的Released包。直接跑Sample项目来测试购买也是一样显示上面的错误。

	<font color="grey">
	2013.11.15，注：<br/>
	发现问题了，香港帐号不行，要美国帐号。很奇怪的是，香港重新正式注册的帐号也不行，都怀疑是不是谷歌政策变化了，香港不再直接支持付费应用了？？
	</font>

## 参考

1. 在[这里](https://code.google.com/p/marketbilling/)是一个针对In-app Purchase的 **官方** issue tracker项目，只是还没有合并到SDK的Sample中。[Google官方对该页面的引用在这里](https://developer.android.com/google/play/billing/billing_admin.html#billing-support)。

2. [串接Google Play In-app-billing 易犯的錯誤](http://lp43.blogspot.tw/2012/04/google-play-in-app-billing.html) 这是一篇在墙外的文章。我转到了[这里](http://kchu.me/2013/11/14/%E4%B8%B2%E6%8E%A5-Google-Play-In-app-billing%E6%98%93%E7%8A%AF%E7%9A%84%E9%8C%AF%E8%AA%A4/)。