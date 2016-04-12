---
layout:     post
title:      "在Cocos2d-x中使用CocosBuilder"
date:       2013-05-24 14:01
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Cocos2d-x
    - Cocos2d
    - C++
    - Games
    - CocosBuilder
---

>本文基于《[Connecting with cocos2d-x](https://github.com/cocos2d/CocosBuilder/blob/master/Documentation/4c.%20Connecting%20with%20cocos2d-x.md)》

## 使用自定义类

连接Cocos2d-x代码到CocosBuilder需要实现两个自定义类：

* 自定义的Loader类，继承自*cocos2d::extension::CCLayerLoader*

* 自定义的Layer类，继承自*cocos2d::extension::CCBSelectorResolver*, *cocos2d::extension::CCBMemberVariableAssigner* 以及 *cocos2d::extension::CCNodeLoaderListener*

并且在自定义Loader类中，需要增加如下初始化用的代码：

`C++ Code:`

	public:
		CCB_STATIC_NEW_AUTORELEASE_OBJECT_METHOD(CustomLayerLoaderClass, loader);    	CCB_VIRTUAL_NEW_AUTORELEASE_CREATECCNODE_METHOD(CustomLayerClass));
    	
CCBReader会使用Loader类的loader()方法来初始化您的自定义Layer。

![在root节点中指定自定义解析类](https://github.com/cocos2d/CocosBuilder/raw/master/Documentation/4-1.png?raw=true)

*图1. 在root节点中指定自定义解析类*

注意CCBReader不能使用任何自定义的init()方法，如果确实想使用一个自定义的init()方法，你必须自己在构造函数中调用它。

## 绑定到成员变量

当ccbi文件被加载后，文件中的对象引用可以被绑定到实际对象的成员变量。并且如果文档的root节点被赋予了自定义类的话，这些成员变量也可以是文档的root节点。

![在特定节点指定需要绑定到的对象属性](https://github.com/cocos2d/CocosBuilder/raw/master/Documentation/4-2.png?raw=true)

*图2. 在特定节点指定需要绑定到的对象属性*

要绑定到一个对象，您只需简单地在头文件中声明它。

要初始化成员变量，您可以使用以下方式覆盖自定义Layer中的onAssignCCBMemberVariable()方法：

`C++ Code:`

	CCB_MEMBERVARIABLEASSIGNER_GLUE(this, "sprtBurst", CCSprite *, this->mSprtBurst);
	
上面的*sprBurst*就是要在CocosBuilder中设置的属性名。

在CocosBuilder中选择需要设置的对象，在*Code Connection*中将*Don't assign*下拉菜单的值改成*Doc root var*或者*Owner var*，然后在右边的文本框中键入成员变量的名字即可。

## 给菜单增加回调

为何让一个CCMenuItemImage响应点击操作，需要给它增加在指定一个回调方法：选中CCMenuItemImage控件，在*selector*框中填入需要回调的方法名，并且把Target设置成*Document root*或*Owner*。

![给菜单指定回调](https://github.com/cocos2d/CocosBuilder/raw/master/Documentation/4-3.png?raw=true)

*图3. 给菜单指定回调函数*

回调方法会将CCMenuItemImage作为唯一参数传递给您指定的方法（它使用的是id类型，参数名称通常叫做*sender*）。当然您也可以选择不去理会这个参数。

在您的自定义类中，必须覆盖*onResolveCCBCCMenuItemSelector()*方法并且增加如下代码：

`C++ Code:`

	CCB_SELECTORRESOLVER_CCMENUITEM_GLUE(this, "pressedA:", 
		MenuTestLayer::onMenuItemAClicked);
	
在上面代码示例中，*MenuTestLayer*是我们的自定义类的类名。

*MenuTestLayer::onMenuItemAClicked*可以做类似如下的定义：

`C++ Code:`

	void MenuTestLayer::onMenuItemAClicked(cocos2d::CCObject *pSender) {	}
	
## 给CCControl增加回调

为CCControl增加回调方法和CCMenuItemImage类似，只是多了一些额外选项。

![给CCControl增加回调](https://github.com/cocos2d/CocosBuilder/raw/master/Documentation/4-4.png?raw=true)

*图4. 给CCControl增加回调*

如图所示，在你希望接收到回调的事件类型前打上勾。对于CCControlButton控件通常只要关心*Up inside*回调就足够了。选择Target类型为*Document root*或*Owner*，以及您希望回调到的方法名称。这个回调可选两个参数：sender（比如CCControl）以及事件类型。事件类型都定义在了*CCControl.h*头文件中。

在自定义类中，需要覆盖*onResolveCCBCCControlSelector()*方法并且增加如下代码：

`C++ Code:`

	CCB_SELECTORRESOLVER_CCCONTROL_GLUE(this, "pressedMenus:", MenuTestLayer::onPressedMenus);
	
在上面代码中，*MenuTestLayer*是我们的自定义类的类名。

*MenuTestLayer::onPressedMenus*可以做类似如下的定义：

`C++ Code:`

	void HelloCocosBuilderLayer::onMenuTestClicked(CCObject * pSender, 
		cocos2d::extension::CCControlEvent pCCControlEvent) {
	}
	
## 加载ccb-files的更多选项

CocosBuilder文档，或者说是ccb-files，需要被发布成一个紧凑的二进制格式文件：ccbi，才能被加载到您的应用程序中。一旦发布他们就能够很容易被使用，简单地使用一行代码即可完成。当您要加载一个节点图的时候，将CCBReader.h和CCBReader.m文件加入到Cocos2D项目中，然后调用*nodeGraphFromFile()*方法如下：

`C++ Code:`

	CCBReader *ccbReader = 
		new cocos2d::extension::CCBReader(ccNodeLoaderLibrary);
	CCNode* myNode = ccbReader->readNodeGraphFromFile("MyNodeGraph.ccbi");
	
可以通过以下两种方式初始化*ccNodeLoaderLibrary*：

1. 如果您在使用着自定义类：

`C++ Code:`

	CCNodeLoaderLibrary * ccNodeLoaderLibrary = 
		CCNodeLoaderLibrary::newDefaultCCNodeLoaderLibrary(); 
	ccNodeLoaderLibrary->registerCCNodeLoader("HelloCocosBuilderLayer", 
		HelloCocosBuilderLayerLoader::loader());
		
在这种情况下，*HelloCocosBuilderLayer*是您的自定义类中名字。

2. 如果您没有使用一个自定义类，您可以初始化一个缺省的NodeLoader：

`C++ Code:`

	CCNodeLoaderLibrary * ccNodeLoaderLibrary = 
		CCNodeLoaderLibrary::newDefaultCCNodeLoaderLibrary();
		
您可以将返回值强制转换到ccbi文件中的根节点的类型，或者您希望根据您的用法决定如何强制转换使用。举个例子，如果您要加载一个粒子系统，可以使用如下代码：

`C++ Code:`

	CCParticleSystem* myParticles = (CCParticleSystem*) 
		ccbReader->readNodeGraphFromFile("MyParticleSystem.ccbi");

这里提供了一个快捷操作，CCBReader可以将您的节点图包装到一个scene中。因此您可以在场景中调用*sceneWithNodeGraphFromFile*方法加载ccbi-file:

`C++ Code:`

	CCScene* myScene = ccbReader->
		sceneWithNodeGraphFromFile("MyScene.ccbi");
		
## 传递一个Owner变量

有时候我们可能要访问一个成员变量或者回调方法中使用根节点以外的其他节点，那么我们就需要传递一个Owner给CCBReader。要获得这个赋予Owner的变量或回调方法，请确保当您声明成员变量的名字或者回调名字的时候，CocosBuilder中该owner处于选中状态。然后调用CCBReader的*nodeGraphFromFile(file, owner)*或者*sceneWithNodeGraphFromFile(file, owner)*方法加载ccbi文件。

`C++ Code:`

	HelloCocosBuilderLayer *pOwner = new HelloCocosBuilderLayer();
	CCNode* myNode = ccbReader->readNodeGraphFromFile(
		"MyNodeGraph.ccbi", pOwner);
		
## 在子ccb-file中访问变量和回调

如果您正在使用一个指定了根节点作为目标的子ccb文件，Owner目标就是您所需要传给CCBReader的。

###示例

可以参照*HelloCocosBuilderLayer.h*，*HelloCocosBuilderLayer.cpp*，以及*HelloCocosBuilderLayerLoader.h*，这些代码在Cocos2d-x附带的TestCPP项目中的ExtensionTest里面。

## 设置缩放和设计大小（尺寸）

对于基于CocosBuilder的Cocos2d-x项目，AppDelegate需要设置游戏去读取正确目录下的正确资源。而这些资源可能基于设备的屏幕尺寸（会有所不同）。您也需要设置缩放因子并且设计一个分辨率给GL View。

对于竖屏模式，您可以增加如下代码到*AppDelegate.cpp*的*AppDelegate::applicationDidFinishLaunching()*方法：

`C++ Code:`

    CCSize designSize = CCSizeMake(320, 480);    CCSize resourceSize = CCSizeMake(320, 480);    CCSize screenSize = CCEGLView::sharedOpenGLView()->getFrameSize();        std::vector<std::string> searchPaths;    std::vector<std::string> resDirOrders;        TargetPlatform platform = CCApplication::sharedApplication()
    	->getTargetPlatform();    if (platform == kTargetIphone || platform == kTargetIpad)    {
    	// Resources/Published-iOS        searchPaths.push_back("Published-iOS");         CCFileUtils::sharedFileUtils()->setSearchPaths(searchPaths);            if (screenSize.height > 768)        {            resourceSize = CCSizeMake(1536, 2048);            resDirOrders.push_back("resources-ipadhd");        }        else if (screenSize.height > 640)        {            resourceSize = CCSizeMake(768, 1536);            resDirOrders.push_back("resources-ipad");        }else if (screenSize.height > 480)        {            resourceSize = CCSizeMake(640, 960);            resDirOrders.push_back("resources-iphonehd");        }        else        {            resDirOrders.push_back("resources-iphone");        }            CCFileUtils::sharedFileUtils()->setSearchResolutionsOrder(
        	resDirOrders);    }    else if (platform == kTargetAndroid || platform == kTargetWindows)    {            if (screenSize.height > 960)        {            resourceSize = CCSizeMake(640, 960);            resDirOrders.push_back("resources-large");        }        else if (screenSize.height > 480)        {            resourceSize = CCSizeMake(480, 720);            resDirOrders.push_back("resources-medium");        }        else        {            resourceSize = CCSizeMake(320, 568);            resDirOrders.push_back("resources-small");        }            CCFileUtils::sharedFileUtils()->setSearchResolutionsOrder(
        	resDirOrders);    }        pDirector->setContentScaleFactor(resourceSize.width/
    	designSize.width);        CCEGLView::sharedOpenGLView()->setDesignResolutionSize(
    	designSize.width, designSize.height, kResolutionShowAll);
    	
对于横屏模式，您可以改变分辨率的顺序，即将(320, 480)改成(480, 320)，将(640, 960)改成(960,640)等等。

--EOF--