---
title: php基础1
date: 2014-04-17 11:55:57
tags: php
---

1. 变量是否为NULL，用`isset($var)`来判断；变量如果为0，False,NULL,'',用`empty($var)`判断。这两个都不是函数，而是系统自带的结构。不属于函数。
2. 包含文件有4种写法，`include('/path/to/file')`,`require('/path/to/file')`,`include_once('/path/to/file')`,`require_once('/path/to/file')`,其中*_once的表示被包含文件只包含一次，不管被包含了几次（可能不小心这样。。。。。）。include包含文件如果不存在，会抛错误，但会继续执行脚本；require包含文件，会抛错误，然后停止执行脚本。
3. to be continued.

