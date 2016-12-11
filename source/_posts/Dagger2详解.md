---
title: Dagger2详解1
date: 2016-10-03 12:03:33
tags: [android,Dagger2,依赖注入]
---

看了许多Dagger2的文章，主要包括：

- [Android：dagger2让你爱不释手系列 ](http://www.jianshu.com/p/cd2c1c9f68d4)
- [Dagger2图文完全教程](https://github.com/luxiaoming/dagger2Demo)


但感觉有的文章光有原理没有例子，有的光讲例子，原理又讲的比较少。因此我也来写一篇Dagger2的文章，尽量结合2者，供大家交流学习。
### @Inject、@Qualifier

如果我们项目中自己定义的类，比如User类，需要注入到MainActivity中，用来代替`new User()`，那么在User类中使用@Inject注解User的构造函数表示提供注入，@Inject注解MainActivity中User类的实例属性来表示注入处。

```java
//MainActivity.java
public class MainActivity extends AppCompatActivity {
  @Inject
  User user; //我需要new User()
}


//User.java
public class User {
  String name;
  String pwd;

  @Inject //我来提供new User()
  public User() {

  }
}
```
如果User类有多个构造函数，则需要@Qualifier定义的注解来区分，比如官方的@Named注解

@Qualifier主要是为解决实例提供方有提供多个相同类型的实例时，加以区分，在这里多种构造函数算其中一种情况

```java
//Dagger2官方提供的@Named注解源码
@Qualifier
@Documented
@Retention(RUNTIME)
public @interface Named {

  /**
   * The name.
   */
  String value() default "";
}

//User.java
public class User {
  String name;
  String pwd;

  @Inject //我来提供new User()
  @Named("user")
  public User() {

  }

  @Named("user_with_value") //我来提供new User(String, String)
  public User(String name, String pwd) {
    this.name = name;
    this.pwd = pwd;
  }
}

//MainActivity.java
public class MainActivity extends AppCompatActivity {
  @Inject
  @Named("user")
  User user;

  @Inject
  @Named("user_with_value")
  User userWithValue;
}
```

### @Module、@Component和@Provides

Module的引入是为了解决第三库提供的实例注入问题，比如User类是某个第三方库的文件，那么我们没办法在其构造函数上增加@Inject注解。这时候可以通过@Module注解的类中以@Provides注解相应返回类型为User的方法来提供实例

**如果你原先的代码现在需要引入Dagger2，也可以通过@Module方式来提供注入，这样基本不用修改原有代码，另外可以把<u>`某个页面`</u>需要注入的实例全部在Module类中提供，也方便维护。而这里的<u>`某个页面`</u>具体要怎么划分，是单个activity或者单个功能，就看情况而定了，目前大多数推荐是以页面来划分Module**

我们选择以页面切分Module，以下Module中提供的依赖供MainActivity页面使用

```java
//MainActivityModule.java
@Module
public class MainActivityModule {
  //返回值表示提供User类的实例注入，Dagger2官方推荐providerXX作方法名
  //如果是在Module中提供多个相同返回类型，也需要用到@Qualified注解过的注解来区分
  @Provides
  @Named("user")
  public User providerUser() {
    return new User();
  }

  //这里的参数类型为String，如果有@Provides注解的方法返回值为String，就会被自动注入，
  //因此这里比较好的提供参数呢？？我也不知道[摊手]
  @Provides
  @Named("user_with_value") 
  public User providerValueableUser(String name, String pwd) {
    return new User(name, pwd);
  }
}
```

如果通过@Inject和@Module都有提供User类的注入依赖，按照Dagger2的处理，优先使用@Module类中提供的注入。

如果User类仅在MainActivityModule类中有定义提供注入的方法，那么在MainActivity.java类中@Inject的属性如何初始化呢？这时候Dagger2提供@Component类来关联：

```java
//MainActivityComponent.java
//定义module值来表示MainComponent可以提供MainActivityModule中的注入
@Component(module = MainActivityModule.class) 
public interface MainActivityComponent {
  //Dagger2推荐injectXX(Object)方法来定义Component要注入的地方
  //Dagger2通过自动生成代码，把MainActivity中要注入的实例，
  //与MainActivityModule中提供的实例初始化方法相关联
  //相当于User user[MainActivity提供] = new User()[MainActivityModule提供]
  void inject(MainActivity mainActivity);
}
```

这时，需要把MainComponent与MainActivity关联，这部分代码用到了Dagger2自动生成的代码。所以需要先编译工程，然后在MainActivity.java中加上以下代码：

```java
//MainActivity.java
public class MainActivity extends AppCompatActivity {
  @Inject
  @Named("user")
  User user; //从MainActivityModule中注入

  MainActivityComponent mainComponent; //Component需要初始化

  @Override
  protected void onCreate(@Nullable Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    //DaggerMainActivityComponent是Dagger2通过编译自动生成的，
    //可以去看看生成的代码是怎么把@Module和@Inject关联起来的
    mainComponent = DaggerMainActivityComponent.builder()
        .build();
    //注入
    mainComponent.inject(this);
  }
}
```

以上讲解了：

1. @Inject注解在`需要注入`和`提供注入`处；

2. @Inject注解在`需要注入`处；

   @Module通过@Provides`提供注入`；

   @Component在`需要注入页面`处初始化，来连接@Inject和@Module

二种情况，也是Dagger2最基本的情况。
下一篇会再讲Dagger2复杂一点的用法。包括
- 多个@Component间的依赖
```java
@Component(dependencies = MainApplicationComponent.class,
           modules ={MainActivityModule.class,ModuleB.class,ModuleA.class,MyModule.class})
```
- 子Component`@SubComponent`
- 作用域`@Scope`
```java
@Scope
@Documented
@Retention(RUNTIME)
public @interface Singleton {}
```
- Map值`@IntoMap`

- Set值`@ElementsIntoSet`

- ...

也不知道啥时候会有下一篇！期待吧！

以上内容均为个人学习理解，如果有错误之处，可以评论告诉我。共同学习，共同进步。