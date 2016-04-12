---
layout:     post
title:      "CocosBuilder中创建导出插件"
date:       2013-06-13 01:17
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Cocos2d
    - Cocos2d-x
    - C++
    - CocosBuilder
    - Games
---

CocosBuilder支持创建自定义的导出插件。CocosBuilder中自带了一个ccbi-exporter用于执行缺省的插件导出功能。但你也很容易就可以增加新的输出格式。

---

## 1. 创建一个新的插件工程

创建一个插件工程的最简单的方式就是复制并且重命名一个现有的示例工程：打开 *PlugIn Exporters* 目录并复制 *Export Example* 目录，将之命名为你的新的到处插件的名称。

现在，在新目录中打开XCode功能，两次点击（缓慢点击，而非双击）工程名可以触发重命名操作，注意你在这里要使用的项目名字，将来会在执行 *Publish As…* 发布插件操作时使用。

接着会需要你重命名更多的工程内容醒目，请保持每一项都选中了的情况下点击 *Rename* 按钮。

插件使用了一个自定义类来到处你的CocosBuilder文档。每个插件的类名都应该是唯一的。因此，你应当将插件的main类进行重命名，最简单的方式是打开文件，右键点击类名并选择 *Refactor* -> *Rename* 。然后键入 **CCBX** 为前缀的容易识别的类名。最后你必须确保Info.plist文件中的 *Principal class* 属性的值也是这个新类名。

要编译和测试该插件的话，首选需要确保在 *build* 目录中已经构建了一个CocosBuilder的备份。然后在插件工程中点击 *Run* 按钮来对插件进行编译，并复制到CocosBuilder的PlugIns目录中。然后你可以打开build目录中的CocosBuilder程序进行测试。所有的输出可以在Console.app中检视，为了更清晰地看到输出，你可以可以设置个过滤器为CocosBuilder。 


---

## 2. 编写插件

导出插件本身的结构非常简单。在main类中必须包含两个方法，这两个方法CocosBuilder会适时回调。

1. *extension* 方法。
	
	此方法只需要简单地将导出文件将要使用的扩展名以字符串的形式返回给调用者。例如缺省的ccbi导出器的extension方法如下：
	
	`Objective-C Code: `
	
		- (NSString*) extension
		{
		    return @"ccbi";
		}

2. *exportDocument* 方法。

	此方法就是程序的主要逻辑所在。此方法会接受一个包含整个文档的NSDictionary对象参数。文档的结构可以参照 *[CCB File Format](https://github.com/cocos2d/CocosBuilder/blob/021d741909b2ffa1ecf9d35027ba67ec76546c66/Documentation/X3.%20CCB%20File%20Format.md)* 。此方法将会创建一个包含需要导出的文件内容的自动释放的NSData对象。示例文件中使用 *NSPropertyListSerialization* 来创建了一个文档的plist格式的表示形式（就像保存为ccb文件格式类似）。
	
	`Objective-C Code: `
	
		- (NSData*) exportDocument:(NSDictionary*)doc
		{
		    return [NSPropertyListSerialization dataFromPropertyList:doc 
		    	format:NSPropertyListBinaryFormat_v1_0 
		    	errorDescription:NULL];
		}


---

## 3. 参考

* [CocosBuilder - Create export plug-in](https://github.com/cocos2d/CocosBuilder/blob/021d741909b2ffa1ecf9d35027ba67ec76546c66/Documentation/X2.%20Creating%20Export%20Plug-ins.md)