---
layout: post
title: "Angularjs如何避免图片加载错误"
description: "Jekyll installation instruction"
category: [jekyll]
tags: [jekyll, installation]
---

1. ng-src属性中不要包含字符串拼接，直接在方括号中使用变量
   ```
    <img style="height:100%; width:100%;" ng-src="{% raw  %}{{brandInfo.brandInfoPhotoPath}}{% endraw  %}">
   ```
