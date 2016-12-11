---
title: Dagger2详解2
date: 2016-12-05 09:56:10
tags: [android,Dagger2,依赖注入]
---

前面我写了[Dagger2中2种注入的基本用法](2016/10/03/Dagger2详解1)，这篇文章来讲下Dagger2比较高级一点的用法。本文也是个人学习思考所得，如有错误之处，还请大家多多评论交流。

### @Component间的依赖

在[Dagger2中2种注入的基本用法](2016/10/03/Dagger2详解1)中，我们按照大多数情况，按页面划分Component类，既一个页面对应一个Component类，这样便于管理。事实上除了在Activity的生存周期内的属性需要注入，很多情况下在Application生存周期内也有属性可以通过Dagger2来注入，这种属性一般是供整个应用来使用的。

因此如大部分教程一样，我们会定义一个全局的Component类来对应于应用的Application类

```java
//MainApplicationModule.java
@Module
public class MainApplicationModule {
  private MainApplication application;
  MainApplicationModule(MainApplication application) {
    this.application = application;
  }
  
  @Provides
  public Context providerApplicationContext() {
    return application;
  }
}

//MainApplicationComponent.java
@Component(module = MainApplicationModule.class)
public interface MainApplicationComponent{
  void inject(MainApplication mainApplication);
} 

//MainApplication.java
public class MainApplication extends Application {
  private MainApplicationComponent component;
  
  @Override
  public void onCreate() {
    component = DaggerMainApplicationComponent.builder()
                .MainApplicationModule(new MainApplicationModule(this))
                .build();
    //注入
    component().inject(this);
  }
}
```

在MainModule中我们提供了ApplicationContext类型的注入。前面说到Application类的提供的注入是供整个应用使用的，那么如果MainActivity需要使用到MainModule中提供的ApplicationContext类型怎么办？这里就要用到Component间的依赖了。

```java
//MainActivityComopnent.java
//我们将MainActivityComponent修改如下，在@Component注解中加上dependencies参数
//此时MainActivityComponent即依赖于MainApplicationComponent
@Component(dependencies = MainApplicationComponent.class, module = MainActivityModule.class)
public interface MainActivityComponent{
  void inject(MainActivity mainActivity);
}

//MainApplicationComponent.java
//MainApplicationComponent要提供自己的ApplicationContext给其他Component类使用，
//需要在自己的类中定义相应的返回类型
@Component(module = MainApplicationModule.class)
public interface MainApplicationComponent{
  void inject(MainApplication mainApplication);
  //增加Context返回类型的方法
  Context getApplicationContext();
}

//MainApplication.java
public class MainApplication extends Application {
  private MainApplicationComponent component;
  //...
  //增加一个方法来使MainActivity可以获得MainApplication中的MainApplicationComponent
  public MainApplicationComponent getComponent() {
    return component;
  }
}

//MainActivity.java
public class MainActivity extends AppCompatActivity {
  MainActivityComponent mainComponent;
  
  @Override
  protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    //依赖的Component在子Component初始化需要提供实例，因此在MainApplication中定义getComponent()方法
    //可以查看Dagger2生成的代码，通过调用MainApplicationComponent中的getContext()方法来注入ApplicationContext
    mainComponent = DaggerMainActivityComponent.builder()
      .applicationComponent(((MainApplication) getApplication()).getComponent())
      .build();
    //注入
    mainComponent.inject(this);
  }
}
```

总结一下：需要依赖其他Component（一般称为子Component）需要在@Comopnent注解中定义dependencies参数，而被依赖的Component（一般称为父Component），需要将可为子Component提供的注入类型，定义成方法，以返回值来提供。

### @SubComponent

@SubComponent从名字上就可以看出来其意思：子Component。上节讲的Component间的依赖也有子Component，那这2种子Component有什么区别呢。

其中最大的区别，就是前一种情况，父Component需要显式的增加相关返回值的方法；而@SubComponent定义了相关的子Component后，仅需在父Component中定义返回该子Component的方法。

另外从项目开发进程的角度，前一种情况，一般是定义了父Component后，比如MainApplicationComponent，一般这种Component可能会被其他Compoennt依赖，为了方便以后扩展，我们无论是否有子Compoenent依赖，都先将其可以提供的注入依赖定义为方法。而当某个子Component需要扩展时，则在@Comopnent注解中定义dependencies参数即可。

而后一种情况，MainActivityComponent所注入的Activity，可能我们没办法预见其会需要一个子Component（这种子Component一般是跟其有关系的Activity或者Fragment），因此，如果MainActivity页面后面需要托管某个Fragment，比如MainFragment，则先用@SubComponent注释MainFragmentComponent，然后再修改MainActivityComponent，加上返回MainFragmentComponent的方法。

即，前一种情况，父Component对子Component无感知，而后一种情况则恰恰相反，子Component不清楚其父Component。

```java
//MainActivityComponent.java
@Component(dependencies = MainApplicationComponent.class, module = MainActivityModule.class)
public interface MainActivityComponent{
  void inject(MainActivity mainActivity);
  //增加MainFragmentComponent返回值的方法，
  //表示MainFragmentComponent是MainActivityComponent的子Component
  MainFragmentComponent getMainFragmentComponent();
}

//MainFragmentComponent.java
//子Component
@SubComponent
public interface MainFragmentComponent {
  //注入MainFragment
  void inject(MainFragment mainFragment);
}

//MainActivity.java
public class MainActivity extends AppCompatActivity {
  MainActivityComponent mainComponent;
  //...
  //增加方法，返回MainActivityComponent，方便MainFragment调用
  public MainActivityComponent getMainActivityComponent() {
    return mainComponent;
  }
}

//MainFragment.java
public class MainFragment extends Fragment {
  MainFragmentComponent mainFragmentComponent;
  public void onActivityCreated(Bundle savedInstanceState) {
    //通过MainActivityComponent类中的方法来获取子Component
    mainFragmentComponent = ((MainActiivty) getActivity).getMainActivityComponent()
      .getMainFragmentComponent();
    //注入
    mainFragmentComponent.inject(this);
  }
}
```

### @Scope

@Scope用来限定@Module或@Inject提供的注入元素的作用域。在Java开发中，最常用的设计模式就是单例模式了，单例模式即类在整个应用生命周期或者某一段生命周期中只有一个实例。而Java提供了@Singleton用于注解仅单个实例的类。Dagger2使用@Scope来注解自定义的作用域类，自定义的@Scope类可以作用于@Component类或者@Provides方法，表示整个@Component提供的注入元素均为单例，或者某个@Provides方法提供的注入元素为单例。

此处的单例，是跟随@Component类的实例生存周期一致的，而@Component类如果是在Application中初始化，则很明示整个APP应用生命周期内，该@Component提供的注入元素为单例，如果是在Activity内onCreate方法中初始化，则只要onCreate方法不再次被调用（一般就是Activity的生命周期），则@Component提供的注入元素为单例。

所以无论是@Singleton还是自定义的@Scope，其实是跟随@Component的被初始化情况来决定单例的作用域的，而他们更大的作用，其实是为了增强代码可读性。比如@ApplicationScope、@ActivityScope。其中@Module提供注入方法注解的@Scope必须与@Module类注解的@Scope一致，比如均为@Singleton或者@PerActivity。而@Component之间有依赖关系的，如果都有@Scope修饰，则不能为相同@Scope注解，否则编译会报错。

```java
//PerActivity.java
//自定义Scope
//表示单个Activity生命周期注入的实例均为单例[根据字面意思，只是方便理解，并不是真的有这层限制]
@Scope
public @interface PerActivity {
}

//MainApplicationModule.java
@Module
public class MainApplicationModule {
  private MainApplication application;
  MainApplicationModule(MainApplication application) {
    this.application = application;
  }
  
  //表示单例，此时相应的Component类也要注解@Singleton
  @Singleton
  @Provides
  public Context providerApplicationContext() {
    return application;
  }
}

//MainApplicationComponent.java
//MainApplicationComponent注入的类的生命周期内，其所注入的元素均为单例
@Singleton
@Component(module = MainApplicationModule.class)
public interface MainApplicationComponent{
  void inject(MainApplication mainApplication);
  //增加Context返回类型的方法
  Context getApplicationContext();
}

//MainActivityComponent.java
//MainActivityComponent依赖MainApplicationComponent，
//如果其也声明为单例，则不能与MainApplicationComponent声明的@Singleton注解相同
//（虽然作用是一样的，但要表明是不同的作用域）
@PerActivity
@Component(dependencies = MainApplicationComponent.class, module = MainActivityModule.class)
public interface MainActivityComponent{
  void inject(MainActivity mainActivity);
  //增加MainFragmentComponent返回值的方法，
  //表示MainFragmentComponent是MainActivityComponent的子Component
  MainFragmentComponent getMainFragmentComponent();
}


//PerFragment.java
//自定义Scope
//表示单个Fragment生命周期注入的实例均为单例[根据字面意思，只是方便理解，并不是真的有这层限制]
@Scope
public @interface PerFragment {
}

//MainFragmentComponent.java
//子Component声明的作用域也不能与其父Component（即MainActivityComponent）一致
//其实@PerFragment与@PerActivity代码是一样的
@PerFragment
@SubComponent
public interface MainFragmentComponent {
  //注入MainFragment
  void inject(MainFragment mainFragment);
}
```

### @IntoSet/@ElementsIntoSet

@IntoMap和@ElementsIntoSet其实很好理解，可以用来注入集合。@IntoMap注入Map，@ElementsIntoSet注入Set。看代码就明白了：

```java
@Module
public class MainActivityModule{
  //...
  @IntoMap
  @StringKey("name") //声明Map中的key值
  @Provides
  String providerName() {
    return "name";
  }
  
  @IntoMap
  @StringKey("pwd")
  @Provides
  String providerPwd() {
    return "pwd";
  }
  
  @ElementsIntoSet
  String providerName() {
    return "lilei";
  }
  
  @ElementsIntoSet
  String providerMoreName() {
    return "zhangshang";
  }
}

public class MainActivity extends AppCompatActivity {
  
  @Inject
  HashMap<String> nameMap; //map->{"name"->"name","pwd"->"pwd"}
  @Inject
  HashSet<String> nameSet; //set->{"lilei","zhangshang"}
}
```



Dagger2的知识点差不多就这些了。@IntoMap和@ElementsIntoSet其实很简单，不过可能用到的没那么多，因此放到这一章文章中。

以上内容均为个人学习理解，如果有错误之处，可以评论告诉我。共同学习，共同进步。



