---
layout:     post
title:      "CocosBuilder动画支持"
date:       2013-06-02 14:01
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Cocos2d-x
    - Cocos2d
    - C++
    - Games
    - CocosBuilder
---

*本文基于《[Working with Animations](https://github.com/cocos2d/CocosBuilder/blob/master/Documentation/6.%20Working%20with%20Animations.md)》。*

CocosBuilder支持用于给角色创建动画。动画编辑器完全支持多分辨率，支持关键帧之间的缓慢动作，骨骼动画，以及多时间线支持等等。

---
#基础

在CocosBuilder主窗口底部可以找到时间线窗口，这里就是创建动画的地方。

![图1. 时间线窗口](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-06-02/6-1.png)

*图1. 时间线窗口*

缺省情况下，你的ccb文件有一个10秒长的时间线。CocosBuilder编辑器中可以编辑每秒30帧的动画，但是当你在实际设备中对动画进行回放时，它会运行在你实际设定的帧率上（默认60fps）。在timeline窗口中，当前时间显示在窗口右上角，格式是 **分:秒:帧数** 。窗口中的蓝色竖线也显示的是当前帧所在的时间点。点击时间框可以改变当前显示的时间长度。

####增加关键帧

CocosBuilder中的动画是基于关键帧的。你可以给节点的不同属性增加关键帧，CocosBuilder回自动在关键帧中自动插入帧序列，包括使用各种不同的ease动画类型。

要增加一个关键帧，首先单击节点名称右边的黑色三角形图标展开该节点视图，在这里你能看到该节点的所有动画属性。一个节点的类型决定它具体都能做些什么动画效果。看到节点之后你就可以按住Option键然后点击timeline窗口中的属性，来在当前时间点创建一个新的关键帧。又或者可以选择一个节点，然后在 *Animation* 菜单中选择 *Insert Keyframe* 来在time marker中创建关键帧。

当你在画布区域改变一个节点的时候，CocosBuilder就会自动创建关键帧。

####编辑关键帧

要编辑一个关键帧，你可以将时间标识移动到该关键帧所在的点，然后选中要编辑的节点。当然你也可以直接双击该关键帧（这个操作能够直接选中节点并移动时间标识到相应位置）。

关键帧的操作，你可以按住鼠标左键拖动出一个矩形选框以便选中多个关键帧；你可以在不同的节点之间复制和粘贴一个或多个关键帧，但是粘贴关键帧时请注意你当前必须选中且只选中了一个节点。粘贴的关键帧会以时间标识当前所在的时间点为起点。

你还可以通过 *Animation* 菜单中的 *Reverse Selected Keyframes* 选项将当前选择的关键帧序列进行逆序。使用*Stretch Selected Keyframes…*选项可以根据缩放因子加快或者减慢帧的播放速度。

####导入图片序列

如果你要使用多个精灵帧来创建一个完整的动画，那在timeline上移动每一个独立帧将是意见非常麻烦而枯燥的事情。而CocosBuilder可以通过导入图片序列来简化这个过程。在左边的工程视图（Project view）中选中你要导入的帧，然后在timeline中选择CCSprite，接着在*Animation*菜单中选择*Create Frames from Selected Resources*选项，你就能看到这些帧将被创建于当前时间标识所在的时间点上。如果要减慢播放速度，你可以选中它们然后使用*Stretch Selected Keyframes…*命令。

####使用easing动画

CocosBuilder提供从Cocos2d中精心选择的一部分Easing动画。要使用easing动画可以在两个关键帧之间点击右键，然后选择你希望使用easing动画类型。

![图2. easing动画](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-06-02/6-2.png "图2. easing动画")

*图2. easing动画*

部分easing动画带了一些可选项，当添加了easing动画之后你可以右键点击该动画然后选择 *Easing Setting...* 调出查看这些可选项。

---
#使用多个Timeline

CocosBuilder动画编辑器的一个非常有用的特性是在一个ccb文件中包含多条时间线，你可以给每条时间线命名为不同的名字，然后在代码中根据名字进行回放。甚至也可以在多条不同的时间线之间实现平滑过渡。

你可以通过时间线弹出窗口选择、增加或者编辑时间线。

![图3. 编辑时间线](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-06-02/6-3.png "")

*图3. 编辑时间线*

在编辑时间线窗口你可以看到对当前所有时间线的一个总览情况。在这里你可以做的操作包括重命名、新增，以及（可选的）设置当ccbi文件被加载时，其中某一条时间线自动进行播放。

![图4. 时间线总览](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-06-02/6-4.png "")

*图4. 时间线总览*

时间线中的没有关键帧集的属性可以用来在各条时间线之间共享其值。例如，如果时间线中的一个节点的position属性没有设置关键帧集，那当你移动这个节点的时候所有其他时间线上的相应属性值都会随之改变。因此有时候这是个非常有用的方法，可以通过在某条时间线上设置共享值，而不是在每一个单独帧上进行独立设置。

####串联时间线

你可以通过串联某些时间线来实现自动地顺序播放它们。你也可以使用这个特性自动循环播放一个时间线。

![图5. 串联时间线](https://raw.githubusercontent.com/kimiazhu/kimiazhu.github.io/master/_posts/attachments/2013-06-02/6-5.png "")

*图5. 串联时间线*

要播放序列中的单个时间线，确保选中 *No chained timeline* ，然后选择并播放你需要的时间线。

####在代码中回放动画

要编程控制你在CocosBuilder中创建的动画，你需要获取 *CCBAnimationManager* 对象，即动画管理器对象。当ccbi文件被加载之后动画管理器会被自动赋予*userObject*节点。

`Objective-C Code:`

    CCNode* myNodeGraph = [CCBReader nodeGraphFromFile:@"myFile.ccbi"];
    CCBAnimationManager* animationManager = myNodeGraph.userObject;
    
动画管理器会被当作一个自动释放的对象来发挥给调用者。要回放一个动画的时候，可以调用 *runAnimationsForSequenceNamed:* 方法，注意如果当前有一个时间线正在播放，调用该方法会导致正在播放的动画立即停止。

`Objective-C Code:`

    [animationManager runAnimationsForSequenceNamed:@"My Timeline"];
    
可选地，你可以使用一个中间过度值来平滑地切换到一个新的timeline，这时候过度时间中的动画将会用线性插值的方式填充。

`Objective-C Code:`

    [animationManager runAnimationsForSequenceNamed:@"My Timeline" 
        tweenDuration:0.5f];
        
你可以使用 *CCBAnimationManagerDelegate* 代理接口，以便当一个时间线播放完成的时候，你也可以收到一个回调方法告诉你播放结束。甚至当另一个时间线串联到序列中的时候你也可以收到回调通知。