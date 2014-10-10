---
layout: post
title: "Run scp in background"
description: "Run scp in background"
category: [scp]
tags: [scp, shell]
---

###Problem Description

Run scp in background

###Solution

scp as a background process

#####To execute any linux command in background we use nohup as follows:


````$ nohup SOME_COMMAND &````

#####But the problem with scp command is that it prompts for the password (if password authentication is used). So to make scp execute as a background process do this:


````$ nohup scp file_to_copy user@server:/path/to/copy/the/file > nohup.out 2>&1````

#####Then press ctrl + z which will temporarily suspend the command, then enter the command:


````$ bg````

#####This will start executing the command in backgroud


````nohup scp /mnt/hd/p1/19/5/* ubuntu@182.254.132.226:/storage/data/xiancai3/ > nohup.out 2>&1 ````
