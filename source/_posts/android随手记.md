---
title: Activity的生命周期和启动模式
date: 2016-04-19 12:56:28
tags: android
---

> 《Android开发艺术探索》笔记

1. 启动新的Activity之前会先调用旧Activity的onPause()方法，因此在onPause()方法中不要做耗时操作

2. Activity可能被杀死的情况

   - 资源相关的系统配置发生变化。可以通过android:configuration来配置什么情况下不销毁重启Activity，常用属性包括`locale`,`oriontation`,`keyboardHidden`，即语言资源变化，横竖屏切换，键盘显示或隐藏。此时系统回调`onConfigurationChanged(Configuration newConfig)`函数。
   - 系统内存不足。系统调用`onSaveInstanceState`->`onStop()`->`onDestroy()`，在`onSaveInstanceState`方法中保存信息；重建时调用，`onCreate()`->`onStart()`->`onRestoreInstanceState`，可以在`onCreate`或`onRestoreInstancestate方法中恢复保存的信息。

3. android:launchMode属性：
   - singleInstance保证类存在单独的task中并仅存在一个实例，适合与程序分离的页面，如闹钟提醒页面；  
   - singleTop表示类在当前task栈顶时，复用，此时调用onNewInstance方法，否则新建实例，适合消息通知类activity；
   - singleTask表示类在当前task栈中存在时，清空在其上面的所有activity实例，直接复用，此时调用onNewInstance方法，否则新建实例，适合用于程序入口。

4. android:taskAffinity属性：

   - 定义该Activity所属的任务栈，默认为程序的包名。配合launchMode为singleTask时使用

5. Activity的Flags

   - FLAG_ACTIVITY_NEW_TASK
     效果等同android:launchMode="singleTask"
   - FLAG_ACTIVITY_SINGLE_TOP
     效果等同android:launchMode="singleTop"
   - FLAG_ACTIVITY_CLEAR_TOP
     清空同一任务栈中在要启动Activity之上的Activity
   - FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS
     不在历史Activity列表中显示，效果等同android:excludeFromRecents="true"
6. Activity所在任务栈由其`taskAffinity`属性决定，默认为应用包名。新启动Activity若未指定TaskAffinity，则置于启动其的Activity所在任务栈的栈顶。非Activity的Context启动的Activity需要指定Flags为`FLAG_ACTIVITY_NEW_TASK`，比如从service中启动Activity，即以singleTask方法启动。
7. 
   