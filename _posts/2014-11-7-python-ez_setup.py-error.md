---
layout: post
title: "Python ez_setup.py error solution"
description: "setuptools,pip,install,UnicodeDecodeError: 'ascii' codec can't decode byte."
category: [python]
tags: [python, solution]
---

---------------------------------------

**Origin**  [setuptools,pip,install,UnicodeDecodeError: 'ascii' codec can't decode byte.原因和解决方案](http://blog.csdn.net/hugleecool/article/details/17996993)

###Description

setuptools,pip,install,UnicodeDecodeError: 'ascii' codec can't decode byte.原因和解决方案
ascii解决方案

昨天重装Python2.7.6时，为了安装第三方库，我去下pip。为了装pip，又得先装 ez_setup.py。结果装ez_setup时，遇到了问题，报错：
[html] view plaincopy在CODE上查看代码片派生到我的代码片

      UnicodeDecodeError: 'ascii' codec can't decode byte 0xb0 in position 1: ordinal not in range(128)
      Something went wrong during the installation.
      See the error message above.

网上找了一大圈，发现也有人在bitbucket提了相同的问题，同时这个stackoverflow的问题也与之类似。

现在发现，这应该都是同一个问题。原因与注册表有关，可能与某些国产软件对注册表的改写的gbk格式导致python无法进行第三方库的安装操作。


------------------------------

###Solution

解决方法：打开C:\Python27\Lib下的 mimetypes.py 文件，找到大概256行（你可以用Notepad++的搜索功能）的

‘default_encoding = sys.getdefaultencoding()’。

在这行前面添加三行：

[python] view plaincopy在CODE上查看代码片派生到我的代码片

      if sys.getdefaultencoding() != 'gbk':
          reload(sys)
          sys.setdefaultencoding('gbk')
      default_encoding = sys.getdefaultencoding()



保存，就解决问题了。
