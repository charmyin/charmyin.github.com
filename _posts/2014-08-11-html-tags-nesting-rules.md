---
layout: post
title: "html标签的嵌套规则（转）"
description: "好好牢记的嵌套规则～"
category: [html]
tags: [html, rules, tags]
---

   **XHTML的标签有许多：``` div、ul、li、dl、dt、dd、h1~h6、p、a、addressa、span、 strong…… ``` 我们在运用这些标签搭建页面结构的时候，是可以将它们无限嵌套的，但是，嵌套也需要有一定规则，不能任由自己的个人习惯胡乱嵌套，那么，html标签的嵌套规则有哪些呢？**

> ![HtmlTagImg]({{ site.url }}/assets/images/2014-08-11/htmltags1.jpg)

### 一、HTML 标签包括 块级元素(block)、内嵌元素（inline）

#### 1、块级元素

   一般用来搭建网站架构、布局、承载内容……它包括以下这些标签：
　 ``` address、blockquote、center、dir、div、dl、dt、dd、fieldset、form、h1~h6、hr、isindex、menu、noframes、noscript、ol、p、pre、table、ul ``` 

####  2、内嵌元素

　　一般用在网站内容之中的某些细节或部位，用以“强调、区分样式、上标、下标、锚点”等等，下面这些标签都属于内嵌元素：
　　``` a、abbr、acronym、b、bdo、big、br、cite、code、dfn、em、font、i、img、input、kbd、label、q、s、samp、select、small、span、strike、strong、sub、sup、textarea、tt、u、var ``` 

### 二、HTML 标签的嵌套规则

#### 1. 块元素可以包含内联元素或某些块元素，但内联元素却不能包含块元素，它只能包含其它的内联元素：
　　``` <div><h1></h1><p></p></div> ``` —— 对

　　``` <a href=”#”><span></span></a> ``` —— 对

　　``` <span><div></div></span> ``` —— 错

#### 2. 块级元素不能放在<p>里面：
　　``` <p><ol><li></li></ol></p> ```  —— 错

　　``` <p><div></div></p> ``` —— 错

#### 3. 有几个特殊的块级元素只能包含内嵌元素，不能再包含块级元素，这几个特殊的标签是：
　　``` h1、h2、h3、h4、h5、h6、p、dt ``` 

#### 4. li 内可以包含 div 标签 —— 这一条其实不必单独列出来的，但是网上许多人对此有些疑惑，就在这里略加说明：
>　　li 和 div 标 签都是装载内容的容器，地位平等，没有级别之分（例如：h1、h2 这样森严的等级制度^_^），要知道，li 标签连它的父级 ul 或者是 ol 都 可以容纳的，为什么有人会觉得 li 偏偏容纳不下一个 div 呢？别把 li 看得那么小气嘛，别看 li 长得挺瘦小，其实 li 的胸襟很大 滴……

#### 5. 块级元素与块级元素并列、内嵌元素与内嵌元素并列：
　　``` <div><h2></h2><p></p></div> ```  —— 对

　　``` <div><a href=”#”></a><span></span></div> ``` —— 对

　　``` <div><h2></h2><span></span></div> ``` —— 错