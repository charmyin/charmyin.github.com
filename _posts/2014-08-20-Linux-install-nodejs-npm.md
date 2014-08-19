---
layout: post
title: "Linux下nodejs的快速安装"
description: ""
category: [InformationTechnology]
tags: [Linux, Nodejs]
---

**下载文件地址** [http://nodejs.org/download/](http://nodejs.org/download/)

简单说就是解压后，在bin文件夹中已经存在node以及npm，如果你进入到对应文件的中执行命令行一点问题都没有，不过不是全局的，所以将这个设置为全局就好了。

    cd node-v0.10.28-linux-x64/bin
    ls
    ./node -v

这就妥妥的了，node文件夹具体放在哪，叫什么名字随你怎么定。然后设置全局：

    ln -s /home/kun/mysofltware/node-v0.10.28-linux-x64/bin/node /usr/local/bin/node
    ln -s /home/kun/mysofltware/node-v0.10.28-linux-x64/bin/npm /usr/local/bin/npm
