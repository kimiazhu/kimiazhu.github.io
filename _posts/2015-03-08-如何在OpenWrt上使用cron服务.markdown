---
layout:     post
title:      "如何在OpenWrt上使用cron服务"
date:       2015-03-08 23:24
author:     "KC"
header-img: "img/post-bg-unix-linux.jpg"
tags:
    - OpenWrt
    - Router
    - PandoraBox
    - Linux
---

## Scheduling Jobs With cron on OpenWrt

This page contains an overview on how to configure cron on a Linksys WRT54GS running OpenWrt, to enable scheduled jobs to be run. 

No additional software needs to be installed on OpenWrt, as it already has the *crond* binary included. 

**Note that OpenWrt whiterussian RC4 and subsequent versions already have cron already pre-configured**, so there's no need to follow the instructions on this page (other than maybe creating the */etc/crontab* symlink for convenience). Cron entries can be specified by adding cron jobs to */etc/crontabs/root*. 


## Configuring crond 

#### Create crontab Directory 

Firstly, create the directory for crontab files: 

	mkdir /etc/crontabs

Note that */var/spool/cron/crontabs/* is the default directory that crond will check for configuration. However, anything in */var/* is lost during a reboot, so we'll use another directory to ensure our crontab file persists between reboots. 

#### Create crontab File 

Cronjobs need to be specified in */etc/crontabs/root*. For now, just create an empty file: 

	touch /etc/crontabs/root

#### Symbolic Link 

For ease of use, I create a symbolic link to the crontab file: 

	ln -sf /etc/crontabs/root /etc/crontab  

This symbolic link is not required by crond, but it allows me to reference the crontab file using */etc/crontab*. 

#### Create Init Script 

Create an init script, */etc/init.d/S60cron*, to start the crond daemon at boot time, with the following contents: 

	#!/bin/sh
	
	# start crond
	/usr/sbin/crond -c /etc/crontabs

and make the script executable: 

	chmod 755 /etc/init.d/S60cron

#### Manually Start crond 

The crond daemon can be manually started using: 

	/etc/init.d/S60cron

Verify that crond successfully started by checking the syslog using: 

	logread

and you should see something similar to this at the end of the logread output: 

	Mar 21 20:29:38 (none) kern.notice crond[687]: crond 2.3.2 dillon, started, log level 8

#### Restart crond 

You'll need to restart *crond* whenever you make changes to the crontab file. This can be achieved using the following: 

	killall crond; /etc/init.d/S60cron

This will stop and restart the crond process, causing it to re-read the crontab file as it starts up, ensuring your changes to the crontab file will take effect. 


## Scheduling Jobs With crond 

Now that you have crond running on OpenWrt, it can be used to perodically run any task that you want. Just add an entry to */etc/crontabs/root* for each task that you want periodically executed. 

For example, if you wanted to run a script every hour, the following would be added to crontab: 

	# run this script every hour
	01 * * * * /path/scriptname > /dev/null

You'll then need to restart *crond* to make this change take effect: 

	killall crond; /etc/init.d/S60cron

Refer to the cron man page for more details on the syntax of the crontab file. 


## Logging by crond 

Note that by default, crond will log a message to the syslog every time it executes a scheduled job. For example, if running a time synchronisation task every 10 minutes, syslog (accessible by running *logread* from a command prompt) will fill up with messages such as this: 

	Dec 22 11:10:01 (none) kern.notice crond[347]: USER root pid 3286 cmd /etc/init.d/S55ntpclient
	Dec 22 11:20:01 (none) kern.notice crond[347]: USER root pid 3290 cmd /etc/init.d/S55ntpclient
	Dec 22 11:21:44 (none) syslog.info -- MARK --
	Dec 22 11:30:01 (none) kern.notice crond[347]: USER root pid 3294 cmd /etc/init.d/S55ntpclient
	Dec 22 11:40:01 (none) kern.notice crond[347]: USER root pid 3298 cmd /etc/init.d/S55ntpclient

The easiest way to stop these from being logged is to configure *crond* to send all log messages to */dev/null*. 
To do so, edit */etc/init.d/S60cron* and change it to: 

	#!/bin/sh
	
	# start crond
	/usr/sbin/crond -c /etc/crontabs -L /dev/null

After restarting crond, it will no longer send any log messages to the syslog. Note that this also means that crond will not log any errors to syslog, so be sure to confirm that crond is operating correctly before configuring it to use /dev/null for logging. 