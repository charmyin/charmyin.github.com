---
layout: post
title: "Mysql -- start: Job failed to start"
description: "Mysql- start: Job failed to start- solution"
category: [nodejs]
tags: [nodejs, npm]
---

###Problem Description

	start: Job failed to start
	invoke-rc.d: initscript mysql, action "start" failed.
	dpkg: error processing mysql-server-5.5 (--configure):
	 subprocess installed post-installation script returned error exit status 1
	dpkg: dependency problems prevent configuration of mysql-server:
	 mysql-server depends on mysql-server-5.5; however:
	  Package mysql-server-5.5 is not configured yet.
	dpkg: error processing mysql-server (--configure):
	 dependency problems - leaving unconfigured
	No apport report written because the error message indicates its a followup error from a previous failure.
	Errors were encountered while processing:
	 mysql-server-5.5
	 mysql-server
	E: Sub-process /usr/bin/dpkg returned an error code (1)

**Solution**

删除mysql前 先删除一下 /var/lib/mysql 还有 /etc/mysql

````sudo rm /var/lib/mysql/ -R````

````sudo rm /etc/mysql/ -R````

````sudo apt-get autoremove mysql* --purge````

````sudo apt-get remove apparmor````

````sudo apt-get install mysql-server mysql-common````
