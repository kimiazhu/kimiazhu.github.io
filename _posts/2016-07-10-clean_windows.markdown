---
layout:     post
title:      "Free System Disk Space on Windows"
date:       2016-07-10 12:52:00 +08:00
lastmod: 	2016-07-10 12:52:00 +08:00
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - Windows
    - Cleaner
---

# Contents：
{:.no_toc}
* any list
{:toc}

The largest folder on my windows is `C:\Windows\Installer`, it takes more than 40GB!

And the second one maybe `C:\Windows\WinSxS`.

I want my space back, but I don't wanna take the risk to delete all the contents.

So my solution is moving them to the other driver and make a soft link to the new target, just like we used to do on Unix-like system.

### 1. : Enter Safe Mode

Restart to `Command prompt with safe mode`.

My PC is Windows 10, It can enter the mode by `Settings` -> `Update & security` -> `Recovery` -> `Advanced startup` -> `Restart now`, 

then following the prompt to start command line window.

### 2. Move Files

the driver label maybe not the same as they in normal mode. you need to use `dir` command to locate the correct folder.

for example: 

[JUST this time]:

F:(safe mode) maps the C:(normal mode)

G:(safe mode) maps the E:(normal mode)

we should enter the follow command to move the files:

```bat
> robocopy F:\Windows\Installer G:\Windows\Installer /MOVE /e
```

### 3. Make Link

Make soft link to the new target to make the files can be used as it lay on the old location.

the target location of mklink is a simple string, so we need to use the path on normal mode.

Just like last sample:

F:(safe mode) maps the C:(normal mode)

G:(safe mode) maps the E:(normal mode)

```bat
> mklink /J F:\Windows\Installer E:\Windows\Installer
```

this time we must use the path relative to the nomal mode.(Driver G: in safe mode is driver E: in normal mode)
