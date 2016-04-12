---
layout:     post
title:      "Cocos2d-x项目移植到WinRT/Win8小记"
date:       2013-11-17 23:33
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Android
    - adb
    - Windows Phone
    - Windows RT
    - WinRT
    - Visual Studio
---

现在的WinRT貌似仍然不争气低没拿下什么市场，这货做得不上不下，位置确实很尴尬，可能真的难有出路。但是如果是已有的游戏进行跨平台移植，那试试也无妨？ :)

基于WP的版本上进行打包编译应该还是比较简单的。WP的版本可以看[这篇小结](http://kchu.me/2013/10/24/Cocos2d-x%E9%A1%B9%E7%9B%AE%E7%A7%BB%E6%A4%8D%E5%88%B0WP8%E5%B0%8F%E8%AE%B0/)。这里是WinRT的一些问题。

1. 打包错误，提示找不到<font color="green">CCPlatformMacros.h</font>或者<font color="green">CCPlatformConfig.h</font>文件。

	<font color="red">
	Error	3	error C1083: Cannot open include file: 'platform/CCPlatformMacros.h': No such file or directory	d:\workspacecocos2dx\cocos2d-x-2.2\cocos2dx\platform\winrt\CCStdC.h	29	1	ThunderPlaneX
	
	Error	18	error C1083: Cannot open include file: 'platform/CCPlatformConfig.h': No such file or directory	D:\WorkspaceCocos2dx\cocos2d-x-2.2\cocos2dx\include\ccConfig.h	30	1	ThunderPlaneX
	</font>

	错误说的很明显了，检查一下项目的include路径，应该是少了cocos2dx所在目录，加入即可： <br/>**<font color="blue">$(ProjectDir)..\..\..\cocos2dx</font>**

	另外需要注意的是,一下两个目录也别忘了要加入include路径当中。：<br/>
	**<font color="blue">$(ProjectDir)..\..\..\CocosDenshion\include</font>**<br/>
	**<font color="blue">$(ProjectDir)..\..\..\CocosDenshion\include</font>**

2. 微软有一个陋习，那就是系统兼容性很差（或者也不叫陋习，那只是促进软硬件更新换代的惯用手法），cocos2.2版本用在win8.1上会出现相当多的deprecated标记，如下。如不影响编译，可以直接忽略：

	<font color="red">
	..\platform\winrt\DirectXBase.cpp(186): warning C4973: 'Windows::Graphics::Display::DisplayProperties::LogicalDpi::get' : marked as deprecated

	Message: 'DisplayProperties may be altered or unavailable for releases after Windows 8.1. Instead, use DisplayInformation.'
	</font>

3. 提示链接错误，找不到CocosDenshion和Extension工程的相关obj，这个问题只需要将这两个工程引入到解决方案中，再在你的游戏工程中添加对这两个工程的依赖即可。

4. 编译错误：找不到pch.h

	<font color="red">
	Error	6	error C1010: unexpected end of file while looking for precompiled header. Did you forget to add '#include "pch.h"' to your source?	D:\WorkspaceCocos2dx\cocos2d-x-2.2\projects\ThunderPlaneX\Classes\about\HTAbout.cpp	72	1
	</font>

	在每个出问题的cpp开头加入**<font color="green">#include "pch.h"</font>**。注意这个不能加#if条件编译，否则会报未预期的#endif标识符。这是因为对于vc++来说，预编译头文件的#include之前的代码将会被完全忽略。

	所以为了和WP版本完全兼容，可以在WP版本的解决方案目录下新建一个空白的pch.h文件。这个不兼容的地方，也希望看到微软以后能够解决掉。

	> 感谢楼下兄弟的回复，我自己对win+vs也是及不了解，刚刚重新找这个问题，确实我们可以在如下位置找到选项，设置不使用预编译头文件：**Project->Configuration Properties->C/C++->Precompiled Headers**中，如图：
	![](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-11-17/8.jpg)

5. 要记得，WinRT默认也只会认识wav格式的声音文件。

6. Debug版本中，有背景音乐但是没有音效？注释掉CocosDenshion里面Audio.cpp的控制音量的一行代码。记得一定是winrt里头的Audio.cpp，路径是：<br />
	**%COCOS2DX_ROOT%\CocosDenshion\<font color="red">winrt</font>\Audio.cpp**<br/>
	具体修改如下：<br/>
	![](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-11-17/1.jpg)

7. 运行时错误：ESP栈未正确设置。

	<font color="red">
	Run-Time Check Failure #0 - The value of ESP was not properly saved across a function call.  This is usually a result of calling a function declared with one calling convention with a function pointer declared with a different calling convention.
	</font>
	
	错误信息如图：

	![](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-11-17/2.jpg)

	![](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-11-17/3.jpg)

	如果你之前是用Eclipse开发Android Apps，那么你遇到此问题的几率很大，编译器对函数指针指向的函数原型不做校验，如果你在使用函数指针时，函数原型不对，少参，例如该指向funcA(CCNode* n)的函数指针，却指向了无参的funcB(void)，那么在Android、iOS系统上运行时也会好好的，甚至WP8也可能不会引起应用程序崩溃，但是如果port到WinRT上你可能会很不幸！

	所以，好好检查下函数指针相关的地方，应该可以解决掉这个问题。

8. 打包错误：提示namespace不对。如下：
	
	<font color="red">
	The types in the HelloCpp namespace are located in file C:\Program Files\WindowsApps\18174A449D4F9.RaidenX_1.1.0.5_x86__ycdqe0xem0zb4\ThunderPlaneX.winmd that does not match the namespace.
	</font>

	![](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-11-17/4.jpg)

	**Ctrl+Shift+H**全文查找替换成你自己的命名空间就OK。至于命名空间到底是什么或者你希望更改命名空间，可以在 **Project->Properties->Configuration Properties->Linker->Windows Metadata->Windows Metadata File** 里找到，里面有个 **$(RootNamespace)** 宏，就是你的名称空间，如下图：

	![](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-11-17/6.jpg)

9. 要创建用于发布的包，在VS2013中是这个位置：**Store->Create App Packages..** 跟着引导输入你的开发者账户，选择需要的类型就可以打包成功。注意的是在cocos2d-x 2.2下无法打包成功x64的安装包，主要是因为缺少了依赖（去看看cocos2d下的third_party目录，没有x64库的），如果确实需要，下面的参考3中有一个解决方案，可以尝试一下。

	最终的打包结果微软提供了一个验证工具，如果结果是passed，那就可以提交市场了。如果不是，有些不是致命的问题，影响也不大，部分问题会导致你被商店拒绝的，在这个校验结果里面也会有提示。

	![](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-11-17/5.jpg)

	点击查看结果后的页面：<br/>
	![](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-11-17/7.jpg)

10. 如何测试应用内产品的购买？
	
	参见参考点4，说实在的，这个设计比Google相差甚远。甚至还没有WP8的方式直接好用。而发布之前需要更改代码（当然你可以用预编译来控制避免每次测试和发布的时候都要修改代码）

11. 禁用或实现右键导航菜单。

	默认右键点击屏幕会出现两个导航菜单，左边是后退，右边是前进。这两个定义在MainPage.xaml中，如果不做任何处理，提交到市场上可能无法通过内容合规性测试，会被拒绝发布。

	如果我们不需要导航，可以在 **MainPage.xaml** 中将<font color="green"> **Page.BottomAppBar** </font> 元素注释或者删除掉，至于配套的代码，可以删掉可也可以留下；如果需要用，则可以实现**MainPage.xaml.cpp**中的如下两个方法（这两个方法的实现默认是空的）：

	`C++ Code:`

		void MainPage::OnPreviousPressed(Object^ sender, RoutedEventArgs^ args) {}
		void MainPage::OnNextPressed(Object^ sender, RoutedEventArgs^ args) {}

12. 添加Privacy Policy

	微软有一份示例，参见参考点6.另外一份中文资料在这里：[Windows 8 添加隐私策略(C++版)](http://www.cnblogs.com/chenkai/archive/2013/02/19/2917626.html)

	在Cocos2d-x中，这个点可以选择放在 **App.xaml.h** 和 **App.xaml.cpp** 中，在：<br/>
	<font color="green">`void App::OnLaunched(LaunchActivatedEventArgs^ args)`</font><br/>
	方法中增加隐私策略的入口。

13. 关于屏幕旋转，有一个说法是必须支持Landscape，如果你的应用确实只支持Portrait，那么请在测试人员注意事项中予以说明。[stackoverflow中有人提到过这个问题](http://stackoverflow.com/questions/15805440/design-my-windows-store-app-in-potrait)。

### 参考

1. [《Cocos2d-x项目移植到WP8小记》](http://kchu.me/2013/10/24/Cocos2d-x%E9%A1%B9%E7%9B%AE%E7%A7%BB%E6%A4%8D%E5%88%B0WP8%E5%B0%8F%E8%AE%B0/)
2. [Porting Cocos2d-x Games for Windows Phone 8](http://developer.nokia.com/Community/Wiki/Porting_Cocos2d-x_Games_for_Windows_Phone_8)
3. 关于如何打包x64 apps，可以参考这里，但我并未尝试过：<br/>
	[Howto: Build your Metro app for ARM & x64](http://www.cocos2d-x.org/forums/6/topics/17435)
4. 关于应用内产品购买：《[启用应用的应用内购买](http://msdn.microsoft.com/zh-cn/library/windows/apps/hh694067.aspx)》
5. [Guidelines for app settings (Windows Store apps)](http://msdn.microsoft.com/en-us/library/windows/apps/hh770544.aspx)
6. [App settings sample from Microsoft](http://code.msdn.microsoft.com/windowsapps/App-settings-sample-1f762f49/sourcecode?fileId=50851&pathId=1614911592)