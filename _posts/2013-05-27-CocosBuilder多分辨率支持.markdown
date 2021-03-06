---
layout:     post
title:      "CocosBuilder多分辨率支持"
date:       2013-05-27 10:18
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Cocos2d-x
    - Cocos2d
    - C++
    - Games
    - Android
    - iOS
---

多分辨率支持是手机做手机有系的一个重要部分。CocosBuilder当然也提供了一些高级工具用来支持多分辨率和相对布局，以便我们能够让单个布局文件支持iPhone、iPad以及众多的Android设备。

当我们创建新文件的时候可以选择需要原生支持的分辨率。可以看到每个分辨率都有一些附带属性可以设置，比如宽度、高度，资源扩展名以及全局缩放系数。

![图1. 创建新的布局文件](/attachments/2013-05-27/5-1.png)

*图1. 创建新的布局文件*

CococsBuilder的分辨率设置只会影响它在CococBuilder上的显示，而不会导出到ccbi文件中。尽管如此，CocosBuilder的缺省设置和Cocos2d的缺省设置是一致的。如果你改变了缺省设置，你要注意您的加载代码中的也需要进行相应的修改。

如果需要在新布局创建好后更改分辨率配置，你可以选择在*View*菜单中*Edit Resolutions…*。


## 选择合适的资源

多分辨率支持的一个重要方面就是要为特定的设备选择合适的图片。*Resource extention*参数就是系统用以决定哪个规格的图片应该配给哪个分辨率的设备的依据。举个例子，配置成*ipad hd*的话，那么系统遇到这种分辨率的设备，会优先使用*-ipad*扩展名的资源，如果没有，则选择*-hd*的。如果所有您配置过的扩展名的资源文件都没有找到，那么系统会自动选择不带扩展名的通用资源文件。您可以将您所有的文件包括不同的分辨率扩展名，放置到项目的resource目录下。

为了清晰起见，CocosBuilder的Project视图只会列出不带扩展名的资源。带扩展名的那些会被隐藏掉。因为系统会在运行时选择合适的资源使用。

## 支持Retina显示屏

需要注意的是，CocosBuilder的工作是基于点（Point）的，而不是像素的。这意味着您在CocosBuilder中只能看到非Retina布局。当您在cocos2d中加载ccbi文件时，系统会为您选择*-hd*或者*-ipadhd*（给视网膜屏版iPad使用）的资源。这就是为什么您在编辑分辨率设置时看不到*-ipadhd*选项的原因。

## 资源尺寸

默认情况下，cocos2d只支持用绝对尺寸来设置一个节点的*contentSize*。在CCBReader中包含一个允许设置和父节点之间有相对关系的contentSize的扩展。（如果您需要在程序中控制这个特性，则需要包含*CCNode+CCBRelativePositioning.h*文件。）

在CocosBuilder中有六个不同的选项可以用来设置节点的尺寸。

![图2. CocosBuilder中支持六种尺寸设置模式](/attachments/2013-05-27/5-2.png)

*图2. CocosBuilder中支持六种尺寸设置模式*

1. 绝对尺寸：注意单位不是像素，是点。
2. 百分比：表示和父节点尺寸之间的比例值。如果父节点是根节点，那就是和整个屏幕尺寸之间的比例值。
3. 相对容器尺寸：此选项会在父节点四周剪掉您输入的宽/高数值（边距），它也可以看成是一个父节点内嵌插画的效果。
4. 水平比例缩放/定高：高度固定，宽度可以设置百分比
5. 垂直比例缩放/定宽：宽度固定，高度可以设置百分比
6. 叠加：这个选项会将您的输入值乘以当前分辨率的缩放因子作为新的大小

## 相对位置

CocosBuilder中的相对位置设置都是和父节点的位置以及尺寸有关的。特别地，百分比选项只有在父节点的尺寸大于0的时候才会有效。

![图3. 相对位置设置](/attachments/2013-05-27/5-3.png)

*图3. 相对位置设置*

1. 绝对位置：这个方式和Cocos2d一样，设置控件位置的绝对值，单位是**点**。实际上是相对于左下角原点的位置。
2. 相对左上角：这个设置控件相对于父节点左上角的位置。
3. 相对右上角：这个设置控件相对于父节点右上角的位置。
4. 相对右下角：这个设置控件相对于父节点右下角的位置。
5. 百分比：设置相对于父控件**右下角**的百分比位置，例如*50,50*表示放置于父节点中间。
6. 叠加：这个选项会将您的输入值乘以当前分辨率的缩放因子作为新的位置

## 相对缩放

您可以对任何空间的缩放或者是对一些基于浮点值的属性（例如CCLabelTTF的字体大小）使用相对缩放。

![图4. 相对缩放设置](/attachments/2013-05-27/5-4.png)

*图4. 相对缩放设置*

1. 绝对缩放：CocosBuilder将不会理会您当前使用的分辨率进行缩放。
2. 叠加：这个选项会将您的输入值乘以当前分辨率的缩放因子作为新的缩放值

## 加载ccbi文件时的可选项

分辨率尺寸并不会保存到ccbi文件中，默认情况下当文件加载的时候，屏幕尺寸将被用作父节点的尺寸。但如果你使用了自定义尺寸的话你就要把这个尺寸传递给loader，这时候要用到这两个方法之一：
*nodeGraphFromFile:owner:parentSize:*或者*sceneWithNodeGraphFromFile:owner:parentSize:*。

`Objective-C Code:`

    CGSize mySize = CGSizeMake(100.0f, 100.0f);
    CCNode* myNode = [CCBReader nodeGraphFromFile:@"myNode.ccbi" 
        owner:NULL parentSize:mySize];
        
在加载ccbi文件之前，您还可以设置您希望使用的缩放因子。缺省的缩放因子在iPhone上是1，iPad上是2，根据实际情况有时候其他的缩放因子可能对你来说会更好。

`Objective-C Code:`

    [CCBReader setResolutionScale: 2.5f];
    
## 有用的Tips！

- 最好在项目一开始的时候就设计好多份便率支持，而不是尝试之后去把一个现成的布局转换去适应不同的设备。

- 如果您希望控件上下加边框(letter boxing),*叠加缩放*会是个不错的选择。

- 可以组合设置锚点和相对位置来把控件放置在角落或者边缘。

- 您可以嵌套不同的位置和尺寸选项来完成那些用于支持多分辨率的复杂行为。

- 不要害怕各种各样的选项，它们一眼看去可能很复杂，但是一旦您掌握了它们您就能拥有更多更好的选择来做屏幕布局工作。