---
layout:     post
title:      "CocosBuilder中创建节点插件"
date:       2013-06-07 01:51
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Cocos2d-x
    - Cocos2d
    - C++
    - CocosBuilder
    - Games
---

*本文基于《[CocosBuilder - Create Node Plug-in](https://github.com/cocos2d/CocosBuilder/blob/021d741909b2ffa1ecf9d35027ba67ec76546c66/Documentation/X1.%20Creating%20Node%20Plug-ins.md)》。*

节点插件能用来为CocosBuilder增加自定义类型的对象，这些对象可以从CocosBuilder中导出来，最终也可以被用回到CocosBuilder中。这些自定义类必须都是CCNode的直接或间接子类。

---
## 1. 建立工程

在XCode中建立插件工程的最简单方式是直接复制*Plugin Nodes*目录中的example工程。

现在，在文件视图中打开工程，点击两次工程名字（不是双击哦），重命名成你希望的名字。

要编辑并测试这个插件，首先需要确定你已经在 *build* 目录构建好了一份CocosBuilder的拷贝，然后点击 *Run* 按钮进行测试。系统会开始编译插件并且拷贝到CocosBuilder的PlugIns目录中。要测试或者调试它的话，在build目录中双击CocosBuilder应用程序，你就可以在Mac OS的Console.app里看到输出了，为了过滤掉无关的日志信息，可以将Console的过滤器设置成CocosBuilder。

---
## 2. 插件基本架构

要创建一个可用的插件，你需要把你自己的类直接添加到工程中，并且编辑 *CCBPProperties.plist* 文件。当自定义类被加载的时候CocosBuilder/CCBReader会调用 *alloc* 和缺省的 *init* 方法创建对象，然后给对象的所有属性赋上值。因此这需要你的自定义类只能使用 *init* 方法进行初始化，而不是使用一个自定义初始化方法。

你创建的这些类将会在运行时绑定到Cocos2d库，同时只有头文件会被包含到工程中。如果你的节点对象用到了多于一个类，则其他那些类不应当被其他插件包含，否则框架在加载插件时候可能会出现冲突。

你可以将许多类型的节点类放到插件中，在 *CCBPProperties.plist* 设置它们中的可以被编辑的属性。有时候你还可能需要把用于显示和加载进App的时候用不同的类来进行去区别。通常最简单的做法是扩展节点的一个子类并且，覆盖它的一些方法来实现。例如你可以在子类中禁用动画或者用动作然后将它用于显示。如果你使用了这个方法，那么你应该将子类的命名前增加 *CCBP* 前缀（例如当你的父类名为CCMyCocosNode的话，子类可以命名为CCBPMyCocosNode）。

---
## 3. 将插件添加到CocosBuilder中

请注意，只有那些很通用的插件才应该默认包含到CocosBuilder中。例如，核心的cocos2d类以及那些被CocosBuilder捆绑使用的GUI组件。

为了创建一个内建插件，你需要执行以下步骤：

1. 为插件创建一个新的 *Bundle* 类型的目标。
2. 在插件目标的构建设置里，设置 *Wrapper Extension* 为 *ccbPlugNode* ，设置 *Other Linker Flags* 为 *-undefined dynamic_lookup* 。
3. 选择CocosBuilder目标，进入到 *Build Phases* 。通过拖放将你的新插件从左边的 *Targets* 添加到 *Target Dependencies* 中。
4. 还是在 *Build Phases* 中，添加你的插件到 *Copy PlugIns* 阶段。
5. 当你增加新的类和资源到插件中的时候，必须注意你只应该将它们添加到插件目标中。

---
## 4. CCBPProperties.plist的格式

插件的大多数行为还是和 *CCBPProperties.plist* 有关。它定义了你的类的所有可以在CocosBuilder中进行编辑的属性，当然还有其他一些关于插件的信息。下面来看看这个文件的大体结构。

### 1) 必填关键字

|Key             |Type    |备注                                        |
|----------------|--------|-------------------------------------------|
|className       |String  |加载进App时要执行的主类的类名，例如，CCMyCocosNode|
|editorClassName |String  |在CocosBuilder编辑器中使用的类名（通常和className一致），例如：CCBPMyCocosNode|
|inheritsFrom    |String  |父类名。例如CCSprite。                        |
|canHaveChildren |String  |表示此插件可以接受其下添加子类                    | 
|properties      |Array   |属性列表                                     |

### 2) 可选关键字

|Key                   |Type       |备注                                                |
|----------------------|-----------|---------------------------------------------------|
|propertiesOverridden  |Array      |格式和properties-key一样，但是会把父类中的对应属性的值覆盖掉 |
|spriteFrameDrop       |Dictionary |指示当一个精灵帧添加到该节点时应该创建什么类型的对象。例如CCCCMenuItemImage应该创建一个CCMenu类型的对象。参见下面的SpriteFrameDrop。|
|requireChildClass     |Array      |这是一个String列表，定义了该节点都允许使用那些类作为子节点。例如CCMenu只允许CCMenuItemImage作为子节点|
|requireParentClass    |String     |指定了该节点是否只能作为子节点出现。例如CCMenuItemImage就只能作为CCMenu的子节点出现|

---
## 5. PlugInProperty

一个PlugInProperty（插件属性）属性在CocosBuilder中应该如何展示，以及它应该如何被CCBReader加载到App中。这是一个包含以下Key的字典。至于哪个属性 *type* 是受支持的以及它如何（将缺省直）序列化则定义在 *Property Types* 文档中。

### 1) 必填关键字

|Key        |Type  |备注                                                |
|-----------|------|---------------------------------------------------|
|type       |String|属性类型，比如Point或者Float                           |
|displayName|String|显示名称，例如Content Size                            |
|name       |String|属性的名字，对于Separator, SeparatorSub and StartStop是非必须的|

### 2) 可选关键字

|Key            |Type   |备注                                                |
|---------------|-------|---------------------------------------------------|
|default        |n/a    |缺省值                                              |
|readOnly       |Boolean|只读，例如CCSprite的contentSize是只读的                |
|dontSetInEditor|Boolean|YES表示编辑器不会读取或者设置该值。相应地，该值会被存储在独立的变量中，并且需要指定缺省值|
|platform       |String |设置这个值是否只被特定平台使用，可用值包括 *Mac* 和 *iOS*   |
|affectsProperties|Array|String数组。数组中定义了当前值如果改变了会影响到哪些其他属性也需要同步进行更新。比如改变了精灵的texture，则精灵的contentSize也会受到影响从而改变|
|extra          |String |不同的type有不同的用法。参见Property Types文档获取详细解释 |

---
## 6. SpriteFrameDrop

*SpriteFrameDrop* 结构体用来定义将CocosBuilder左边的资源列表中的物件拖放到Timeline中的该类的节点上时发生的行为。

### 必填关键字

|Key            |Type   |备注                                                |
|---------------|-------|---------------------------------------------------|
|className      |String |需要创建的类的名字，例如：CCSprite                       |
|property       |String |需要分配给拖放到的节点的该精灵的属性名称，例如CCSprite拖放到CCLayer中的时候应该赋值displayFrame|

---
## 参考

* [摆脱无尽的贴坐标，自制CocosBuilder插件（CCBPProperties.plist详解）](http://www.cnblogs.com/Ringo-D/archive/2013/05/21/3090951.html)