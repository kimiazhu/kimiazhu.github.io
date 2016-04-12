---
layout:     post
title:      "Windows8关联同步Google日历"
date:       2013-11-10 15:24
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Windows
    - Win8
    - Sync
    - Calendar App
---

Windows 8不能直接同步Google日历，这个很蛋疼，还好Gmail还能用。查到一种可以在Win8的Calendar.app上查看Google日历的方法，就是 **订阅** ,订阅是不能修改的，只能看。

下文也提到了一种可以修改的方法，但实际上这个方式太迂回，等于是让你把Google日历和Outlook日历一起使用，其实两者我不想混淆，个人事情还是想记录在Google日历上。

我的Microsoft帐号比较特殊，用的是我的Gmail邮箱做账号名，我是早期注册的，现在貌似已经不允许使用非微软的邮箱注册微软帐号了。所以我用方法二，登录一个Google帐号，日历和邮件都同步过来了。（方法2完成之后，这个微软帐号就已经订阅了Google日历内容了）

[问答的原文在这里。](http://answers.microsoft.com/en-us/windows/forum/windows_8-ecoms/how-to-get-your-google-calendar-events-to-show-in/5fa4c0c5-1967-4b99-89bc-542ff0cb15a2)

[另外这里有一篇官方的文档](http://windows.microsoft.com/en-us/windows/outlook/calendar-import-vs-subscribe)，但是我发现有时候订阅的日历不能更新，重新reset了一下Google日历的私有分享链接，然后在Outlook.com上重新订阅，貌似好了。

======

If you’d like to see your Google Calendar events in the Calendar app, you have two options:
 
1. You can permanently move them into Outlook.com.  When you’re done, all your events will be available in the Windows Calendar app.  From there, you can edit, delete, and update your events. 
 
2. You can subscribe to your Google calendar from Outlook.com.  When you’re done, you can see all your events in the Windows 8 Calendar app, but you won’t be able to add, edit or delete events from the app.  If you want to make updates, you’ll have to go to google.com and sign in.
 
### Option 1:  
Move your Google Calendar events to Outlook.com
 
You can find detailed steps for [moving your Google Calendar events to Outlook by clicking the link](http://go.microsoft.com/fwlink/?LinkId=293780).
 
### Option 2:  
Subscribe to your Google Calendar from Outlook.com
 
- Sign in to your Google account.
- Tap or click Calendar, tap or click the settings icon, and then tap or click Settings.
- Tap or click on the calendar you want to subscribe to and next to the Private address, tap or click **ICAL** .
- Copy the URL under Private address.
- Sign in to Outlook.com.
- Tap or click Subscribe.
- Paste the URL you’d copied earlier in Calendar URL, add a name for this calendar, and tap or click Subscribe to calendar.