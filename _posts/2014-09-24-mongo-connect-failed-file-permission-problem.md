---
layout: post
title: "Mongodb not work when changed default db path"
description: "Mongodb not work when changed default db path"
category: [mongodb]
tags: [mongodb]
---

###Problem 

````Thu Sep 25 00:39:03 [initandlisten] exception in initAndListen: 10309 Unable create/open lock file: /yammy/appSpace/yammy/db/mongod.lock````

````errno:13 Permission denied Is a mongod instance already running?, terminating'````

**Solution**

````chown -R mongodb:nogroup db````