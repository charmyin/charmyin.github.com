---
layout: post
title: "Debug js in chrome by insert debugger"
description: "Debug js in chrome by insert debugger"
category: [javascript]
tags: [debug, javascript, js]
---

###Problem Description

	需要调试js的时候，我们可以给需要调试的地方通过debugger打断点，代码执行到断点就会暂定，这时候通过单步调试等方式就可以调试js代码

**Solution:**

使用javascript的断点, 在需要打断点的地方添加debugger：

		if (waldo) {
			debugger;
		}
