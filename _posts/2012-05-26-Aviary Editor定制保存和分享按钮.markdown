---
layout:     post
title:      "Aviary Editor定制保存和分享按钮"
date:       2012-05-26 15:00
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Android
    - Aviary
    - Image Proccess
---

# 1、需求
默认情况下Aviary编辑器处理完图片会将之以指定文件名保存。文档中给出的方式是调用startActivityForResult()方法，即调用完成之后会返回原Activity。并且保存的图片貌似不会通知媒体库，导致系统相册中无法看见该图片。（HTC EVO 4G测试）

现在的需求是：

1. 保存图片后通知媒体库，并且跳回相机拍照页面，同时页面要更新左下角缩略图。
2. 增加“分享”按钮，分享功能是保存图片，然后跳转到分享页面，并将图片连接传递到分享Activity。跳转到分享页面。

# 2、分析和解决
PS: 大致看过一下Aviary的条款，貌似在修改方面有限制，这两点可以视为是对原有功能的增强，不过仍是要改动lib工程中的部分代码。

这次只奔着快速解决问题的目的，程序代码结构先不管。Aviary有很多值得研究的地方，但在这里就不做讨论了。

### 1) 需求1
注意到顶部工具栏的的代码在ToolbarView类中。保存按钮的引用变量为**mSaveButton**。

```java
// ToolbarView.java;
mSaveButton.setOnClickListener( new OnClickListener() {

	@Override
	public void onClick( View v ) {
		if ( mListener != null && 
				mCurrentState == STATE.STATE_SAVE && isClickable() ) {
			mListener.onSaveClick();
		}
	}
} );
```

ToolbarView中的保存按钮的单击动作在做三个判断后调用了监听器的**mListener.onSaveClick()**方法。该监听器接口只有一个实现，在**FeatherActivity.java**中：

```java
//FeatherActivity.java:
/**
 * User clicked on the save button.<br /> 
 * Start the save process
 */
@Override
public void onSaveClick() {
	
	if( mFilterManager.getEnabled() ){
		mFilterManager.onSave();

		if ( mFilterManager != null ) {
			Bitmap bitmap = mFilterManager.getBitmap();
			if ( bitmap != null ) {
				performSave( bitmap );
			}
		}
	}
}
```

继续跟踪到**performSave()**方法中起了一个后台线程调用**doSave()**方法，然后对bitmap手工进行了**recycle()**。找到最终我们关心的**doSave()**方法：

```java
//FeatherActivity.java:
/**
 * Do save.
 *
 * @param bitmap the bitmap
 */
protected void doSave( Bitmap bitmap ) {

	// result extras
	Bundle extras = new Bundle();

	// if the request intent has EXTRA_OUTPUT declared
	// then save the image into the output uri and return it
	if ( mSaveUri != null ) {
		OutputStream outputStream = null;
		String scheme = mSaveUri.getScheme();
		try {
			if ( scheme == null ) {
				outputStream = new FileOutputStream( mSaveUri.getPath() );
			} else {
				outputStream = getContentResolver().openOutputStream( mSaveUri );
			}
			if ( outputStream != null ) {
				int quality = Constants.getValueFromIntent( Constants.EXTRA_OUTPUT_QUALITY, 90 );
				bitmap.compress( mOutputFormat, quality, outputStream );
			}
		} catch ( IOException ex ) {
			logger.error( "Cannot open file", mSaveUri, ex );
		} finally {
			IOUtils.closeSilently( outputStream );
		}
		onSetResult( RESULT_OK, new Intent().setData( mSaveUri ).putExtras( extras ) );
	} else {
		// no output uri declared, save the image in a new path
		// and return it

		String url = Media.insertImage( getContentResolver(), bitmap, "title", "modified with Aviary Feather" );
		Uri newUri = null;
		if ( url != null ) {
			newUri = Uri.parse( url );
			getContentResolver().notifyChange( newUri, null );
		}
		onSetResult( RESULT_OK, new Intent().setData( newUri ).putExtras( extras ) );
	}

	final Bitmap b = bitmap;
	mHandler.post( new Runnable() {

		@Override
		public void run() {
			mImageView.clear();
			b.recycle();
		}
	} );

	finish();
}
```

我们发现当我们给定了输出URI的时候，系统仅仅是将图片输出到指文件，是不会通知媒体库的。那么我们在mSaveUri != null的分支中增加一句：

```java
bitmap.compress( mOutputFormat, quality, outputStream );
//added by [Kimia Zhu] start
getContentResolver().notifyChange( mSaveUri, null );
//added by [Kimia Zhu] end
```

通知媒体库的问题得到解决。

update: 发现部分环境不起作用，可以改成以下代码（不推荐），扫描整个目录（当然，代码需要优化，目录名不建议放在这里，可以采用常量或者参数传入，或者直接从目标目录解压。）：

```java
	sendBroadcast(new Intent(Intent.ACTION_MEDIA_MOUNTED, Uri.parse("file://"
		                + Environment.getExternalStorageDirectory())));
```

关于更新缩略图，按照官方原来的模式，是调用**startActivityForResult()**方法来调用打开Aviary编辑器，并不结束当前Activity，当Aviary编辑器结束自身时，就返回到之前的页面（我们这里是相机）。但是这种方式无法满足我们的需求。我们是希望在用户编辑完图片时，可以选择保存到相册或是分享到微博。这里就总共有两个跳转目标：

1. 保存到相册，那么Aviary编辑器结束后应该返回相机重新拍照；

2. 分享到微博，那么Aviary编辑器结束后应该跳转到分享页面。

而按照编辑器原来的**startActivityForResult()**的方式它只能回到相机，我们需要改动编辑器的部分代码。（当然可以在流程上做改动，在调用编辑器前，增加一个activity，编辑器统一返回到这个activity，然后再做跳转，但这种方式在用户体验上并不好。另外一种可选的方式是直接跳回相机，由相机再做分发，但这样明显有效率问题，多做了一次到相机的无意义跳转，并且可能会由于横竖屏切换导致用户会感觉跳转有些卡。）

所以这个地方仍然只能改动编辑器，要求调用方在调用编辑器的**FeatherActivity**之后，结束自身，之后的跳转反相，由编辑器里面来重新执行**startActivity()**打开页面。

由于Aviary Editor本身是一个lib工程，和调用方的依赖关系是相机工程依赖编辑器工程，在编译器编译时编辑器中无法直接调用相机或者分享类的Activity，这个问题可以通过反射解决，在系统运行时就能拿到相机和分享类。


### 2)需求2

需求2需要增加一个分享按钮，同样是**ToolbarView.java**这个Widget。对应的layout是**feather_toolbar.xml**文件。

这里不讨论Style，只管在原有模块上多加一个功能，在**android:text="@string/save"**的**button_apply**下面增加一个按钮：

```xml
<!-- feather_toolbar.xml -->
<Button
    android:id="@+id/button_share"
    android:layout_width="wrap_content"
    android:layout_height="match_parent"
    android:text="@string/share" 
    style="@style/FeatherToolbarButton" 
    android:layout_toLeftOf="@id/button_apply" 
    android:focusable="false" 
    android:focusableInTouchMode="false"/>
```

我们直接沿用原有的样式可以，日后需要更改样式，可以有统一的地方来调整。比较麻烦的是**strings.xml**，这里要增加一个share的文案，要翻译成那么多种语言实在不是容易的事情。（我打算只留下中文和英文，其他语言考虑直接删掉好了。）

代码上仿造原有的保存按钮的方式写，逻辑也很简单，保存之后，把uri发给share页面，这里的share页面调用，仍然需要以反射完成。