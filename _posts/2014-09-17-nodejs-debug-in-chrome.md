---
layout: post
title: "nodejs基于Chrome浏览器的调试器"
description: "基于Chrome浏览器的调试器"
category: [Nodejs]
tags: [nodejs, javascript, debug]
---

Via [基于Chrome浏览器的调试器](http://www.cnblogs.com/moonz-wu/archive/2012/01/15/2322120.html)


既然我们可以通过V8的调试插件来调试，那是否也可以借用Chrome浏览器的JavaScript调试器来调试呢？node-inspector模块提供了这样一种可能。我们需要先通过npm来安装node-inspector

	```npm install -g node-inspector  // -g 导入安装路径到环境变量```

node-inspector是通过websocket方式来转向debug输入输出的。因此，我们在调试前要先启动node-inspector来监听Nodejs的debug调试端口。

![1]({{ site.url }}/assets/images/2014-09-17/1.png)

默认情况下node-inspector的端口是8080，可以通过参数--web-port=[port]来设置端口。在启动node-inpspector之后，我们可以通过--debug或--debug-brk来启动nodejs程序。通过在浏览器输入http://[ip address]:8080/debug?port=5858，我们会得到如下的调试窗口：

![2]({{ site.url }}/assets/images/2014-09-17/2.png)

 这三种方法各自有优缺点，我个人比较欣赏node-inspector的方式。


