---
title: android随手记
date: 2016-04-19 12:56:28
tags: android
---

1. setResult类的launchMode必须是standard，否则调用类调用startActivityForResult方法后，会立即执行onActivityResult方法

2. launchMode属性：
   - singleInstance保证类仅有一个task内存在一个实例，适合与程序分离的页面，如闹钟提醒页面；  

   - singleTop表示类在当前task栈顶时，复用，此时调用onNewInstance方法，否则新建实例，适合消息通知类activity；

   - singleTask表示类在当前task栈中存在时，清空在其上面的所有activity实例，直接复用，此时调用onNewInstance方法，否则新建实例，适合用于程序入口。