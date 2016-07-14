---
title: android学习笔记(4)
date: 2015-11-19 14:32:27
tags: android
---
> Android四大组件
> <code>Activity</code>
>    <code>Service</code>
>    <code>BroadCastReceiver</code>
>   <code>Content Provider</code>

## Activity
1. ### Activity的生命周期
* onCreate():创建
* onStart():运行
    * onResume():获取焦点
    * onPause():失去焦点
    * onStop():暂停
    * onDestroy():销毁
    * onRestart():重启
1. ### Activity四种状态
* Activ/Running 活动状态
* Pause 暂停状态
    * Stopped 停止状态
    * Killed 非活动状态

## Fragment
1. ### Fragment的生命周期
* onAttach()
* onCreate()
* onCreateView()
* onActivityCreated()
* onStart()
* onResume()
* onPause()
* onStop()
* onDestoryView()
* onDestory()
* onDetach()

## Intent
1. ### Intent实现页面之间跳转
   * startActivity(intent)
   * startActivityForResult(intent, requestCode, options)
      * 传递类 onActivityResult(int requestCode, int resultCode, Intent data)
      * 接收类 setResult(int resultCode,Intent intent)