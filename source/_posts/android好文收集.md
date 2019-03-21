---
title: android好文收集
date: 2016-05-19 18:06:18
tags: android
---

### Android
1. [Android事件分发机制完全解析，带你从源码的角度彻底理解(上)](http://blog.csdn.net/guolin_blog/article/details/9097463)
2. [Android事件分发机制完全解析，带你从源码的角度彻底理解(下)](http://blog.csdn.net/guolin_blog/article/details/9153747)

   - onTouch() 方法返回false--->onTouchEvents()--->onClick()  
   - onInterceptTouchEvent默认返回false,优先子类处理点击事件，否则父view处理
3. [给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083)
4. [Android 热修复方案对比](http://jaeger.itscoder.com/android/2016/08/28/android-hot-fix.html)

### Android tips/count
1. [awesome-android-tips](https://github.com/jiang111/awesome-android-tips)
2. [Android学习资源网站大全](https://github.com/zhujun2730/Android-Learning-Resources)
3. [那些年你用过的 Android 开源项目都有什么?](http://diycode.cc/topics/8)
4. [从零开始的Android新项目5 - Repository层(上) Retrofit、Repository组装](http://blog.zhaiyifan.cn/2016/04/30/android-new-project-from-0-p5/)

5. [我所理解的RxJava——上手其实很简单（一）](http://www.jianshu.com/p/5e93c9101dc5)
6. [给 Android 开发者的 RxJava 详解](https://gank.io/post/560e15be2dca930e00da1083#toc_4)
7. [全面了解Nginx到底能做什么](http://www.raye.wang/2017/02/24/quan-mian-liao-jie-nginxdao-di-neng-zuo-shi-yao/)
8. https://github.com/markzhai/AndroidProjectFrom0 <---这是啥？可以吃吗？
9. https://ydmmocoo.github.io/2016/06/22/Android%E9%9D%A2%E8%AF%95%E9%A2%98%E6%95%B4%E7%90%86/
7. https://github.com/xiangzhihong/android-Interview
8. http://www.jianshu.com/p/e99b5e8bd67b

### 面试
1. [Android工程师面试题大全](http://blog.csdn.net/mc_hust/article/details/49517915)
2. [2016Android某公司面试题](http://yuweiguocn.github.io/2016/04/13/interview-2016-big-company/)

### Git
1. [10 个迅速提升你 Git 水平的提示](http://www.oschina.net/translate/10-tips-git-next-level)


### 设计模式
1. [图说设计模式](https://design-patterns.readthedocs.io/zh_CN/latest/)

### 基础

dispatchTouchEvent

onInterceptTouchEvent

onTouchEvent

1. android 事件分发：dispatchTouchEvent->onInterceptTouchEvent->onTouchEvent
2. android activity的生命周期：onCreate-onStart-onResume-onPause-onStop-onRestart-onDestroy
3. activity启动另一个activity时，先调用前一个activity的onPause->另一个activity的onResume

保存数据操作可以在onStop中处理或者onDestroy

1. fragment生命周期：onAttach->onCreate->onCreateView->onActivityCreated->onStart->onResume->onPause->onStop->onDestroyView->onDestroy->onDetach
2. service生命周期：bindService->onCreate->onBind->onUnBind->onDestroy

startService-onCreate->onStartCommond-onStart

1. BroadcastReceiver：registerBroadcast

2. Android设计模式[https://gof.quanke.name/]

   - 创建型->建造者模式、单例模式、简单工厂模式、工厂方法模式、抽象工厂模式、原型模式[6]
   - 结构型->适配器模式、桥接模式、装饰模式、外观模式、享元模式、代理模式、组合模式[7]
   - 行为型->策略模式、观察者模式、迭代器模式、责任链模式、命令模式、模板方法模式、访问者模式[7]

3. 创建型

   - 建造者模式，
   - 单例模式，
   - 简单工厂模式（在一个方法里面作判断,来返回相应的对象实例），
   - 工厂方法模式（通过接口定义，子类实现来返回相应的对象实例），一般是静态方法
   - 抽象工厂模式（通过接口定义，将具有同一主题的单独工厂封装在一起）
   - 单例模式
   - 原型模式（clone)

4. 结构型

   - 适配器模式，客户端：activity->listview.setAdapter(new MyAdapter());一个类的接口转换成用户所期待的，分为类适配器和对象适配器，类适配器表示为class Adapter implements Target extends Adaptee，通过继承来适配，一般不推荐。对象适配器表示为class Adapter implements Target{Adaptee mAdaptee},通过持有被适配对象的实例来适配

     listview.getCount(),getItemPosition->Adapter->Target[ListView所需的getItem,getCount,]->Adaptee[数据集]

   - 桥接模式，将抽象部分与它的实现部分分离，使它们都可以独立地变化

   - 组合模式，组合多个对象形成树形结构以表示具有“整体—部分”关系的层次结构

5. 程序无响应。避免在UI线程执行大量耗时操作，可以使用AsyncTask,HandleThread,Service,Thread
### Retrofit-Gson解析
每次都对应Bean对解析

Class->Callback<Person>

```json
{
  "name":"lee",
  "sex":"male"
}
```

Array->Callback<List<Person>>

```json
[
  {
  	"name":"lee",
  	"sex":"male"
  },
  {
    "name":"hu",
    "sex":"female"
  }
]
```

每次请求都有状态码和状态值后，如果使用data把具体的对象包起来

封装一个Response

class->Callback<Response<Person>>

```json
{
  "code":200,
  "message":"OK",
  "data":{
    "name":"lee",
    "sex":"male"
  }
}
```

Array-Callback<Response<List<Person>>

```json
{
  "code":200,
  "message":"OK",
  "data":{[
    {
      "name":"lee",
      "sex":"male"
    },
    {
      "name":"hu",
      "sex":"female"
    }
  ]}
}
```



Retrofit2可以替换的地方：addConvertFactory(GsonConvertFactory.create())->增加解析器，如gson解析，将最优先的解析器放在前面。addCallAdapterFactory(new ErrorHandleFacotry.create())/addCallAdapterFactory(RxJavaCallAdapter.create())->替换成自定义call。

Gson中的GsonBuilder可以自定义，GsonBuilder().registerTypeAdapter(BaseResponse.class,new TypeAdapter() )与registerTypeAdapterFacotry(ItemTypeAdapterFactory.create())基本一致，用于自定义处理BaseResponse返回的数据



