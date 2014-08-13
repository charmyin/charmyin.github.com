---
layout: post
title: "解决@JsonFormat输出小于1的BUG"
description: "@JsonFormat的timezone错误！"
category: [InformationTechnology]
tags: [Java, SpringMVC]
---

## 问题描述： 时区问题导致的日期不一致
> 通过@JsonFormat输出日期时，由于默认时区与当地时区不一致原因，导致日期与原有日期有出入

![有问题代码]({{ site.url }}/assets/images/2014-08-13/jsonformat1.png)

## 解决方法
> 加入时区
![问题解决]({{ site.url }}/assets/images/2014-08-13/jsonformat2.png)

