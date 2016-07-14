---
title: jdk7与mysql的连接问题
date: 2014-04-23 12:51:37
tags: [java,mysql]
---

今天第一天上班，工作就是熟悉一下环境。

svn checkout下来代码以后，就准备跑。结果tomcat报jdbc连接不到数据库。

``` 
com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure
The last packet sent successfully to the server was 0 milliseconds ago. The driver
has not received any packets from the server.
```

然后网上各种wait_timeout啊，connect url写错，mysql启动参数多了--skip-networking等等，但试过都没用。

然后我就开始一下午的逐个排查。

   **整个结构：**

- IDE问题
- tomcat问题
- mysql-java-connector-5.1.x.jar包问题
- 代码问题[ssh2框架]
- mysql问题

**解决：**

1. IDE用的MyEclipse，于是下个Eclipse for JavaEE,运行结果依旧。
   1. tomcat版本7.0,不管这个，直接写个java类，在main方法里通过jdbc连接mysql，这样代码的问题也排除了。运行结果还是一样报错。
   2. 从官网下载最新的connector/J驱动，加载，还是有问题。
   3. mysql用的32位5.5版本，但系统是64位的，虽然通过IDE运行代码报错，但第三方工具比如navicat for mysql可以连接到本地mysql,用命令行也可以。暂时不管，再下64位5.5版本mysql。运行代码还是报错。

这个就没辙了。环境基本都换新了。也不会出现wait_timeout参数这一类问题。

最后，求助同事。同事看了下，表示他那边运行相同代码没问题。 这时候，我突然想到jdk版本来了。于是问他机器jdk版本多少。他说是jdk6。

好吧。这时候，我只好最后试一试，下了jdk6u38，再重新编译。运行代码，顺利连上mysql，并获取数据。

好吧，原因我也不知道，可以这台电脑装的jdk7有问题。记在这里。

以前从没用过jdk7,新入职，这台电脑正好装的jdk7,居然就碰上这么匪夷所思的问题。 花费了一下午的时间。晕。