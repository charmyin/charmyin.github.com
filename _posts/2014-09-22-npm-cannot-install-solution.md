---
layout: post
title: "Solution for npm install not work"
description: "npm install failed , due to net work connection"
category: [nodejs]
tags: [nodejs, npm]
---


**1. Show current configuration**

  ````npm config ls````

**2. Set repository url, Use "http" directly instead of "https"**

  ````npm config set registry="http://registry.npmjs.org"````
