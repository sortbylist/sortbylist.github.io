---
title: Java基础回顾
date: 2017-02-08 17:08:59
tags: [Java]
---
# Java 多态
父类对象引用子类实例。子类继承父类后，同一个方法有不同的表现。
- 方法重载实现静态多态性（编译时多态）
- 方法重写实现的动态多态性（运行时多态）


# Java 虚拟机
## Java程序执行过程

- *Java Source (.java File)*
- Java Compiler(javac)
- *Java Byte Code(.class File)*
- **Class Loader**
- Execution Engine
- **Runtime Data Areas**
- Java Virtual Machine

## 类加载器 （ **Class Loader ** ）

- Bootstrap Class Loader
- Extension Class Loader
- System Class Loader
- User-Defined Class Loader

## 运行数据区域 （ **Runtime Data Areas** ）

- PC Register
  每个线程有一个PC计数器。当线程启动时，PC计数器被创建，这个计数器存放当前正在被执行的字节码指令（ JVM 指令 ）的地址。

- JVM Stack
  Java栈中存放着一系列的栈桢（ Stack Frame ），JVM 只能进行压入（ Push ） 和弹出 （ Pop ) 栈帧这两种操作。每当调用 一个方法时，JVM就往栈里压入一个栈帧，方法结束返回时弹出栈帧。如果方法执行时异常，可以调用printStackTrace 等方法来查看栈的情况。
  - Stack Frame
    - Local Variable Array 局部变量队列
    - Operand Stack 操作数栈
    - Reference to Constant Pool常量池引用

- Native Method Stack
  当程序通过 JNI（Java Native Interface）调用本地方法（如 C 或者 C++ 代码）时，就根据本地方法的语言类型建立相应的栈。
- Heap
  堆中存放的是程序创建的对象或者实例。这个区域对 JVM 的性能影响很大。垃圾回收机制处理的正是这一块内存区域。 所以，类加载器加载其实就是根据编译后的 Class 文件，将 java 字节码载入 JVM 内存，并完成对运行数据处的初始化工作，供执行引擎执行。
- Method Area
  方法区域是一个 JVM 实例中的所有线程共享的，当启动一个 JVM 实例时，方法区域被创建。它用于存运行时常量池、有关域和方法的信息、静态变量、类和方法的字节码。不同的 JVM 实现方式在实现方法区域的时候会有所区别。Oracle 的 HotSpot 称之为永久区域（Permanent Area）或者永久代（Permanent Generation）。
- Runtime Constant Pool

  这个区域存放类和接口的常量，除此之外，它还存放方法和域的所有引用。当一个方法或者域被引用的时候，JVM 就通过运行常量池中的这些引用来查找方法和域在内存中的的实际地址。 

前三个属于每个线程独自拥有。后三者是整个JVM实例中所有线程共有的。

## 执行引擎 （Execution Engine ）

类加载器将字节码载入内存之后，执行引擎以Java字节码指令为单元，读取Java字节码。通过解释器将字节码转化成平台相关的机器码，这个过程也可以由即时编译器 （ JIT Compiler ） 来完成。

# Java内存管理

## Java的内存管理就是对象的分配和释放问题。

- 分配

  内存分配由程序完成，使用 new 关键词为对象申请内存空间（基本类型除外），所有的对象都在堆（ Heap ) 中分配空间。

- 释放

  对象的释放由垃圾回收机制（ GC ）决定和执行。

## Java的内存区域分栈内存和堆内存

### 栈内存

在函数中定义的基本类型变量和对象的引用变量都在函数的栈内存中分配。在函数（代码块）中定义一个变量时， java 就在栈中为这个变量分配内存空间，当超过变量的作用域后，java 会自动释放掉为该变量所分配的内存空间。

### 堆内存

用来存放由 new 创建的对象和数组以及对象的实例变量。在堆中分配的内存由 java 虚拟机的自动垃圾回收器来管理。

- 堆的优点是可以动态分配内存大小，生存期也不必事先告诉编译器，因为它是在运行时动态分配内存的。缺点是在运行时动态分配内存，存取速度较慢
- 栈的优点是存取速度比堆要快，仅次于直接位于CPU中的寄存器。另外栈数据可以共享。缺点是存在栈中的数据大小与生存期必须是确定的，缺乏灵活性。

### Java中数据在内存中的存储位置

- 基本数据类型（8种，即int, short, long, byte, float, double, boolean, char）。保存在栈中，并且共享。所以
  ```java
  String s1 = "a";
  String s2 = "a";
  System.out.println(s1==s2);//true
  ```

- 对象

  ```java
  public class Person {
    String name;
    int age;
    public Person(String name, int age) {
      this.name = name;
      this.age = age;
    }
  }

  Person p = new Person("joally", 24);
  //1.Person p声明一个对象Person，其引用命名为p,此时在栈内存中为引用变量p分配内存空间，此时p是空对象，等同于Person p = null;
  //2.p = new Person("joally", 24)在堆内存中为类的成员变量name和age分配内存，并将其初始化为各数据类型的默认值；接着进行显式初始化（类定义时的初始化值）；最后调用构造方法，为成员变量赋值。返回堆内存中对象的引用（相当于首地地址）给引用变量p，以后就可以通过栈内存中的p来引用堆内存中的Person对象了。
  ```

- 数组

  当定义一个数组时，在栈内存中创建一个数组引用，通过该引用（即数组名）来引用数组。

  ```java
  int x[]; //在栈内存中创建数组引用x;
  x = new int[3];//在堆内存中分配3个保存int型数据的空间，并初始化为0，堆内存的首地址赋值给x，即通过栈内存中x来引用堆内存中的int数组。
  ```

- 静态变量

  用static修饰的变量和方法，实际上是指定了这些变量和方法在内存中的“固定位置” - static stoage，可以理解为所有实例对象共有的内存空间。静态表示的的是内存的共享，就是它的每一个实例都指向同一个内存地址，它的引用（含间接引用）都指向同一个位置。

Java 程序的多个部分（方法，变量，对象）驻留在内存中以下两个位置：堆和栈。实例变量和对象驻留在堆上，局部变量驻留在栈上。

```java
public class Dog {
Collar c;
String name;
//1. main()方法位于栈上
public static void main(String[] args) {
//2. 在栈上创建引用变量d,但Dog对象尚未存在
Dog d;
//3. 在堆中创建新的Dog对象，并将其赋予d引用变量
d = new Dog();
//4. 将引用变量的一个副本传递给go()方法
d.go(d);
}
//5. 将go()方法置于栈上，并将dog参数作为局部变量
void go(Dog dog){
//6. 在堆上创建新的Collar对象，并将其赋予Dog的实例变量
c =new Collar();
}
//7.将setName()添加到栈上，并将dogName参数作为其局部变量
void setName(String dogName){
//8. name的实例对象也引用String对象
name=dogName;
}
//9. 程序执行完成后，setName()将会完成并从栈中清除，此时，局部变量dogName也会消失，尽管它所引用的String仍在堆上
}
```


# Java泛型

泛型的主要目标是提高 Java 程序的类型安全。消除强制类型转换，提高代码可读性，减少出错机会。

# Java序列化

## 定义

序列化，就是为了保存对象的状态，与之对应的是反序列化，就是把保存的对象状态再读出来。

> 序列化/反序列化，是Java提供一种专门用于保存/恢复对象状态的机制。

一般以下几种情况下，我们可能会用到序列化：

- 想把内存中的对象状态保存到一个文件中或数据库中
- 想用套接字在网络上传送对象
- 想通过**RMI ( Remote Method Invocation 远程连接调用 )** 传输对象时

## `java.io.Serializable`

类通过实现`Serializable`接口，可实现自动序列化

- transient瞬态，用于不序列化的属性
- 静态变量，属于类的状态，不序列化
- 如果想自定义序列化的内容，类中需要实现`writeObject(java.io.ObjectOutputStream o)`/`readObject(java.io.ObjectInputStream o)`方法，分别用于序列化和反序列化。自定义方法第一行可调用`out.defaultWriteObject()`/`out.defaultReadObject()`来调用jdk的默认序列化，然后再实现自己的序列化

## `java.io.Externalizable`

如果一个类要完全负责自己的序列化，则实现 Externalizable 接口，而不是 Serializable 接口。

> Externalizable接口定义包括两个方法`writeExternal()`与`readExternal()`。需要注意的是：声明类实现Externalizable接口会有重大的安全风险。`writeExternal()`与`readExternal()`方法声明为public，恶意类可以用这些方法读取和写入对象数据。如果对象包含敏感信息，则要格外小心。

- `writeExternal(ObjectOutput out)`
- `readExernal(ObjectInput in)`
- `readResolve`/`writeReplace`可以用在单例模式时的序列化和反序列化，用于指定替代的对象

# Java基本数据结构

- boolean、char[2byte，16bit]、short[2byte，16bit]、int[4byte，32bit]、float[4byte，64bit]、long[8byte，64bit]、double[8byte，64bit]
- 基本数据包装类，继承于`java.lang.Number`类，包括：`java.lang.Byte`、`java.lang.Short`、`java.lang.Integer`、`java.lang.Float`、`java.lang.Long`、`java.lang.Double`，以及`java.lang.Boolean`、`java.lang.Character`
- Java字符，原始类型`char`，包装类`java.lang.Character`，jdk6基于Unicode Standard version4.0，jdk7基于Unicode Standard version6.0，jdk8基于Unicode verison6.2.0。
- Unicode编码，使用“U+”然后紧接着一组十六进制的数字表示一个字符。从U+0000到U+FFFF的字符集被称作*Basic Multilingual Plane(BMP)*，又称作`零号平面`，其中的所有字符，要用四个数字（即2个char，16bit，例如U+4AE0，共支持2^16-1=65535个字符）；在零号平面以外的字符则需要使用五个或六个数字，范围为U+0000到U+10FFFF，即通常所说的*Unicode标准量*。
- java.lang.Math、java.util.Date、java.text.SimpleDateFormat、java.text.NumberFormat、java.text.MessageFormat、java.util.Calendar、java.util.GregorianCalendar

<!-- more -->


# Java正则表达式

## java.util.regex包主要包含三个类

### **Pattern类**

一个Pattern对象是正则表达式编译表示。 Pattern 类没有提供公共的构造函数。要创建一个 Pattern 对象，你必须首先调用他的公用静态编译方法来获得 Pattern 对象。这些方法的第一个参数是正则表达式。

### **Matcher类**

一个Matcher对象是用来解释模式和执行与输入字符串相匹配的操作。和Pattern类一样Matcher类也是没有构造方法的，你需要通过调用 Pattern对象的matcher方法来获得Matcher对象。

### **PatternSyntaxExceptiopn类**

一个PatternSyntaxException对象是一个不被检查的异常，来指示正则表达式中的语法错误。

```java
String text = "gogo";
String patternStr = ".g.";
Pattern pattern = Pattern.compile(patternStr);
//Pattern pattern = Pattern.compile(patternStr, Pattern.CASE_INSENSITIVE)
//String[] split = pattern.split(text);
//String patternStr = pattern.pattern();
//boolean isExist = Pattern.matches(patternStr, text);
Matcher matcher = pattern.matcher(text);
boolean isExist = matcher.matches(); //整个文本中是否有匹配
//boolean lookingAt = matcher.lookingAt(); //是否匹配开头
boolean isFound = matcher.find(); //查找下一匹配
//int startPos = matcher.start(); //返回每次匹配的字符串在整个文本中的开始位置
//int endPos = matcher.end(); //结束位置的后一位
String str = matcher.group(); //当前匹配文本中第一个组，从0开始算起
StringBuffer sb = new StringBuffer();
while(matcher.find()) {
int groupCount = matcher.groupCount(); //当前匹配中匹配的组个数
for(int i=1;i<=groupCount;i++) {
System.out.println("found: " + matcher.group(i));//从1开始，0表示整个匹配值
}
matcher.appendReplacement(sb, "done"); //将匹配到的内容替换为指定str,并附加到StringBuffer
}
matcher.appendTail(sb); //将最后matcher中最后剩余的内容附加到StringBuffer
//System.out.println(sb.toString());String text = "gogo";
String patternStr = ".g.";
Pattern pattern = Pattern.compile(patternStr);
//Pattern pattern = Pattern.compile(patternStr, Pattern.CASE_INSENSITIVE)
//String[] split = pattern.split(text);
//String patternStr = pattern.pattern();
//boolean isExist = Pattern.matches(patternStr, text);
Matcher matcher = pattern.matcher(text);
boolean isExist = matcher.matches(); //整个文本中是否有匹配
//boolean lookingAt = matcher.lookingAt(); //是否匹配开头
boolean isFound = matcher.find(); //查找下一匹配
//int startPos = matcher.start(); //返回每次匹配的字符串在整个文本中的开始位置
//int endPos = matcher.end(); //结束位置的后一位
String str = matcher.group(); //当前匹配文本中第一个组，从0开始算起
StringBuffer sb = new StringBuffer();
while(matcher.find()) {
int groupCount = matcher.groupCount(); //当前匹配中匹配的组个数
for(int i=1;i<=groupCount;i++) {
System.out.println("found: " + matcher.group(i));//从1开始，0表示整个匹配值
}
matcher.appendReplacement(sb, "done"); //将匹配到的内容替换为指定str,并附加到StringBuffer
}
matcher.appendTail(sb); //将最后matcher中最后剩余的内容附加到StringBuffer
//System.out.println(sb.toString());
```

## 正则表达式语法

### **字符**

对应其8进制，16进制，或者unicode编码

- `John`=`101`=`\x41`=`\u0041`
- `\\`反斜线
- `\0n`带有八进制值0的字符n(0 <= n <= 7)
- `\0nn`带有八进制值0的字符nn(0 <= n <= 7)
- `\0mnn`带有八进制值0的字符mnn(0 <= m <= 3、0 <= n <= 7)
- `\xhh`带有十六进制值0x的字符hh
- `\uhhhh`带有十六进制值0x的字符hhhh
- `\t`制表符(`\u009`)
- `\n`新行（换行）符(`\u000A`)
- `\r`回车符(`\u000D`)
- `\f`换页符(`\u000C`)
- `\a`报警（bell）符(`\u0007`)
- `\e`转义符(`\u001B`)
- `\cx`对应于x的控制符

### **字符类**

使用方括号`[]`表示字符组。可以匹配方括号中一个或多个字符。方括号本身并不是要匹配的一部分。

`[abc]`、`[a-zA-Z]`、`[a-z&&[def]]`表示匹配d、e或f、`[a-z&&[^bc]]`=`[ad-z]`

### **预定义字符类**

- `.`任何字符（与行结束符可能匹配也可能不匹配）
- `\d`数字：`[0-9]`
- `\D`非数字：`[^0-9]`
- `\s`空白字符：`[ \t\n\x[0B\f\r\]]`
- `\w`单词字符：`[a-zA-Z_0-9]`
- `\W`非单词字符：`[^w]`

### **边界匹配**

- `^`行的开头
- `$`行的结尾
- `\b`单词边界
- `\B`非单词边界
- `\A`输入的开头
- `\G`上一个匹配的结尾
- `\Z`输入的结尾，仅用于最后的结束符（如果有的话）
- `\z`输入的结尾

### **数量词**

数量词匹配分为**贪婪模式**，**饥饿模式**，**独占模式**。饥饿模式匹配尽可能少的文本。贪婪模式匹配尽可能多的文本。独占模式匹配尽可能多的文本，基本导致剩余表达式匹配失败。

- `X?`、`X??`、`X?+`一次或一次也没有
- `X*`、`X*?`、`X*+`零次或多次
- `X+`、`X+?`、`X++`一次或多次
- `X{n}`、`X{n}?`、`X{n}+`正好n次
- `X{n,}`、`X{n,}?`、`X{n,}+`至少n次
- `X{n,m}`、`X{n,m}?`、`X{n,m}+`至少n次，不超过m次

### 逻辑操作符

- XY表示and
- X|Y表示X或Y
- (X)表示捕获组

### 特殊构造（非捕获）

- `(?:X)`X，作为非捕获组，匹配X，不捕获匹配的文本，也不给此分组分配组号
- `(?=X)`X，通过零宽度的正lookahead，匹配X前面的位置
- `(?!X)`X，通过零宽度的负lookahead，匹配后面跟的不是X的位置
- `(?<=X)`X，通过零宽度的正lookbehind，匹配X后面的位置
- `(?<!X)`X，通过零宽度的负lookbehind，匹配前面不是X的位置
- `(?>X)`X，作为独立的非捕获组，贪婪子表达式

### 后向引用

- `\n`，n为数字，表示前面第n个捕获组
- `(?<name>X)`X，匹配X，并捕获X到名称为name的组里，等同于`(?'name'X)`
- `\k<name>`匹配前面命名的捕获组，对应于`(?<name>)`或`(?'name')`

### 平衡组/递归匹配

# Java IO
`java.io`包主要涉及文件访问、网络数据流、内存缓冲访问、线程内部通信（管道）、缓冲、过滤、解析、读写文件（Readers/Writers）、读写基本类型数据（long，int etc）、读写对象等输入输出。

## 字节流

### `InputStream`字节输入流

- ByteArrayInputStream
- FileInputStream
- FilterInputStream
  - BufferedInputStream
  - DataInputStream
  - ~~LineNumberInputStream~~
  - PushbackInputStream
- ObjectInputStream
- PipedInputStream
- SequenceInputStream
- StringBufferStream

### `OutputStream`字节输出流

- ByteArrayOutputStream
- FileOutputStream
- FilterOutputStream
  - BufferedOutputStream
  - DataOutputStream
  - PrintStream
- ObjectOutputStream
- PipedOutputStream

## 字符流

### `Reader`字符输入流

- BufferedReader
  - LineNumberReader
- CharArrayReader
- FilterReader
- PushbackReader
- InputStreamReader
  - FileReader
- PipedReader
- StringReader

### `Writer`字符输出流

- BufferedWriter
- CharArrayWriter
- FilterWriter
- OutputStreamWriter
  - FileWriter
- PipedWriter
- PrintWriter
- StringWriter

# Java异常处理

- Throwable
  - Error
    错误，是程序无法处理的错误，表示运行应用程序中较严重问题。属于不可检测的异常（Unchecked Excepiton)
  - Exception
    异常，是程序本身可以处理的异常。分为可检测异常（Checked Exception）、运行时异常（Runtime Exception）。
    - IOException
      可检测异常通常是用户错误或是程序员可预计的，需要在编译时处理，一般用try-catch语句捕获或者用throws子句声明抛出。
    - RuntimeException
      运行时异常程序可以选择捕获处理，也可以不处理，这些异常一般是由程序逻辑错误引起的，程序应该从逻辑角度尽可能避免这类异常的发生。属于不可检测的异常（Unchecked Excepiton)。

# Java集合

## 总体框架

- Java集合是Java提供的工具包，包含了常用的数据结构：集合、链表、队列、栈、数组、映射等。Java集合工具包位置是`java.util.*`

- Java集合主要可以划分为4个部分：List列表、Set集合、Map映射、工具类(Iterator迭代器、Enumeration枚举类、Arrays和Collections)。

- 主要类：`ArrayList`、`LinkedList`、`HashMap`、`LinkedHashMap`、`TreeMap`、`HashSet`、`LinkedHashSet`

- 主体：`Collection`接口、`Map`接口

### 主要接口

  - `Collection`
    - `List`
    - `Set`
      - `SortedSet`
        - `NavigableSet`
    - `Queue`
      - `Deque`
  - `Map`
    - `SortedMap`
      - `NavigableMap`
  - `Iterator`
    - `ListIterator`

### 主要类

  - `AbstractCollection`
    - `AbstractList`
      - `AbstractSequentialList`
        - `LinkedList`
          链表
      - `ArrayList`
        动态数组
    - `AbstractQueue`
      - `PriorityQueue`
    - `AbstractSet`
      - `EnumSet`
      - `HashSet`
        基于HashMap的Set接口的数组和链表实现
        - `LinkedHashSet`
      - `TreeSet`
    - `ArrayDeque`
  - `AbstractMap`
    - `EnumMap`
    - `HashMap`
      Map接口的数组与链表实现
      - `LinkedHashMap`
    - `TreeMap`
    - `WeakHashMap`
    - `IdentityHashMap`

## List

### **ArrayList**

List接口的大小可变数组的实现。实现了所有可选列表操作，并允许包括Null在内的所有元素。

```java
//JDK6
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
     // 默认构造函数
    ArrayList()

    // capacity是ArrayList的默认容量大小。当由于增加数据导致容量不足时，容量会添加上一次容量大小的一半。
    ArrayList(int capacity)

    // 创建一个包含collection的ArrayList
    ArrayList(Collection<? extends E> collection)

    // Collection中定义的API
    boolean             add(E object);
    boolean             addAll(Collection<? extends E> collection);
    void                clear();
    boolean             contains(Object object);
    boolean             containsAll(Collection<?> collection);
    boolean             equals(Object object);
    int                 hashCode();
    boolean             isEmpty();
    Iterator<E>         iterator();
    boolean             remove(Object object);
    boolean             removeAll(Collection<?> collection);
    boolean             retainAll(Collection<?> collection);
    int                 size();
    <T> T[]             toArray(T[] array);
    Object[]            toArray();
    // AbstractCollection中定义的API
    void                add(int location, E object);
    boolean             addAll(int location, Collection<? extends E> collection);
    E                   get(int location);
    int                 indexOf(Object object);
    int                 lastIndexOf(Object object);
    ListIterator<E>     listIterator(int location);
    ListIterator<E>     listIterator();
    E                   remove(int location);
    E                   set(int location, E object);
    List<E>             subList(int start, int end);
    // ArrayList新增的API
    Object               clone();
    void                 ensureCapacity(int minimumCapacity);
    void                 trimToSize();
    void                 removeRange(int fromIndex, int toIndex);
}
```

ArrayList包含了两个重要的对象：`elementData` 和 `size`。

- `elementData` 是"Object[]类型的数组"，它保存了添加到ArrayList中的元素。实际上，elementData是个动态数组，我们能通过构造函数 ArrayList(int initialCapacity)来执行它的初始容量为initialCapacity；如果通过不含参数的构造函数ArrayList()来创建ArrayList，则elementData的容量默认是10。elementData数组的大小会根据ArrayList容量的增长而动态的增长，具体的增长方式，请参考源码分析中的`ensureCapacity()`函数。

- `size`则是动态数组的实际大小。

`fast-fail`是一种错误检测机制。当在多线程环境下同时对`ArrayList`进行读写操作时，会抛出`ConcurrentModificationException`异常。可使用`CopyOnWriteArray`,因为`CopyOnWriteArray`的所有写操作都会copy一份新的数组，而通过`Iterator`遍历时不支持写操作，因为不会抛出`ConcurrentModificationException`异常。

### **LinkedList**

List接口的链接列表实现。实现所有可选列表操作，并且允许包括Null在内的所有元素。

是一个继承于`AbstractSequentialList`的双向链表。可以被当作堆栈、队列 或双端队列进行操作。

```java
//JDK6
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable {}

// 默认构造函数
LinkedList();
// 创建一个LinkedList，保护Collection中的全部元素。
LinkedList(Collection<? extends E> collection);
    
//LinkedList的API
boolean       add(E object);
void          add(int location, E object);
boolean       addAll(Collection<? extends E> collection);
boolean       addAll(int location, Collection<? extends E> collection);
void          addFirst(E object);
void          addLast(E object);
void          clear();
Object        clone();
boolean       contains(Object object);
Iterator<E>   descendingIterator();
E             element();
E             get(int location);
E             getFirst();
E             getLast();
int           indexOf(Object object);
int           lastIndexOf(Object object);
ListIterator<E>     listIterator(int location);
boolean       offer(E o);
boolean       offerFirst(E e);
boolean       offerLast(E e);
E             peek();
E             peekFirst();
E             peekLast();
E             poll();
E             pollFirst();
E             pollLast();
E             pop();
void          push(E e);
E             remove();
E             remove(int location);
boolean       remove(Object object);
E             removeFirst();
boolean       removeFirstOccurrence(Object o);
E             removeLast();
boolean       removeLastOccurrence(Object o);
E             set(int location, E object);
int           size();
<T> T[]       toArray(T[] contents);
Object[]      toArray();
```

- `AbstractSequentialList`

  实现了`get(int index)`、`set(int index, E element)`、`add(int index, E element)`、`remove(int index)`函数，这些接口都是随机访问List的。

- LinkedList的本质是双向链表。LinkedList继承于AbstractSequentialList，并且实现了Dequeue接口。 

- LinkedList包含两个重要的成员：header 和 size。

  - header是双向链表的表头，它是双向链表节点所对应的类Entry的实例。Entry中包含成员变量： previous, next, element。其中，previous是该节点的上一个节点，next是该节点的下一个节点，element是该节点所包含的值。
  - size是双向链表中节点的个数。

### List总结

![List框架图](/images/list_struct.jpg)

- List 是一个接口，它继承于Collection的接口。它代表着有序的队列。
- AbstractList 是一个抽象类，它继承于AbstractCollection。AbstractList实现List接口中除size()、get(int location)之外的函数。
- AbstractSequentialList 是一个抽象类，它继承于AbstractList。AbstractSequentialList 实现了"链表中，根据index索引值操作链表的全部函数"。
- ArrayList, LinkedList, Vector, Stack是List的4个实现类。
  - ArrayList 是一个**数组队列，相当于动态数组**。它由数组实现，随机访问效率高，随机插入、随机删除效率低。
  - LinkedList 是一个**双向链表**。它也可以被当作堆栈、队列或双端队列进行操作。LinkedList随机访问效率低，但随机插入、随机删除效率高。
  - Vector 是**矢量队列，和ArrayList一样，它也是一个动态数组，由数组实现**。但是ArrayList是非线程安全的，而Vector是线程安全的。
  - Stack 是**栈，它继承于Vector**。它的特性是：先进后出(FILO, First In Last Out)。
- 如果涉及到“栈”、“队列”、“链表”等操作，应该考虑用List，具体的选择哪个List，根据下面的标准来取舍。
  - 对于需要快速插入，删除元素，应该使用LinkedList。
  - 对于需要快速随机访问元素，应该使用ArrayList。
    对于“单线程环境” 或者 “多线程环境，但List仅仅只会被单个线程操作”，此时应该使用非同步的类(如ArrayList)。
  - `Vector`和`Stack`使用`synchronized`关键字来实现线程安全，效果效低，不建议使用。多线程环境需要线程安全的`List`建议使用`CopyOnWriteList`。


## Map

### **HashMap**

- HashMap是基于哈希表的Map接口的非同步实现。此实现提供所有可选的映射操作，并允许使用null值和null键。此类不保证映射的顺序，特别是它不保证该顺序永久不变。
- HashMap实际上是一个“链表散列”的数据结构，即数组和链表的结合体。
- HashMap在底层将key-value当成一个整体进行处理，这个整体就是一个Entry对象。HashMap底层采用一个Entry[]数组来保存所有的key-value对，存储时，会根据hash算法来决定其在数组中的存储 位置，再根据equals方法决定期在数组位置上的链表中的存储位置，取出时，根据hash算法找到其在数组中的存储位置，再根据equals方法从该位置上的链表中取出该Entry。
- HashMap的元素大小超过capacity*loadFactor时，就会扩容为原先容量的2倍。capacity为原先容量，loadFactor为负载因子，默认为0.75，负载因子越大，对空间利用越充分，但相应查找效率变低，反之，数据过于稀疏，空间造成浪费。
- Fail-Fast机制，快速失败，HashMap不是线程安全的，当其他线程对其进行修改，当前线程会直接抛出ConcurrentModificationException。
- HashMap是通过"拉链法"实现的哈希表。它包括几个重要的成员变量：table, size, threshold, loadFactor, modCount。
  - table是一个Entry[]数组类型，而Entry实际上就是一个单向链表。哈希表的"key-value键值对"都是存储在Entry数组中的。 
  - size是HashMap的大小，它是HashMap保存的键值对的数量。 
  - threshold是HashMap的阈值，用于判断是否需要调整HashMap的容量。threshold的值="容量*加载因子"，当HashMap中存储数据的数量达到threshold时，就需要将HashMap的容量加倍。
  - loadFactor就是加载因子。 
  - modCount是用来实现fail-fast机制的。

```java
public class HashMap<K,V>
    extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable { }

// 默认构造函数。
HashMap()

// 指定“容量大小”的构造函数
HashMap(int capacity)

// 指定“容量大小”和“加载因子”的构造函数
HashMap(int capacity, float loadFactor)

// 包含“子Map”的构造函数
HashMap(Map<? extends K, ? extends V> map)

//API
void                 clear();
Object               clone();
boolean              containsKey(Object key);
boolean              containsValue(Object value);
Set<Entry<K, V>>     entrySet();
V                    get(Object key);
boolean              isEmpty();
Set<K>               keySet();
V                    put(K key, V value);
void                 putAll(Map<? extends K, ? extends V> map);
V                    remove(Object key);
int                  size();
Collection<V>        values();
```

### TreeMap

- TreeMap是一个**有序**的key-value集合，它是通过`红黑树`实现的。
- TreeMap 实现了NavigableMap接口，意味着它**支持一系列的导航方法。**比如返回有序的key集合。
- TreeMap基于**{% post_link 红黑树 %}（Red-Black tree）实现**。该映射根据**其键的自然顺序进行排序**，或者根据**创建映射时提供的 Comparator 进行排序**，具体取决于使用的构造方法。
- TreeMap的基本操作 containsKey、get、put 和 remove 的时间复杂度是 log(n) 。
- 另外，TreeMap是**非同步**的。 它的iterator 方法返回的**迭代器是fail-fastl**的。

```java
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable {}

// 默认构造函数。使用该构造函数，TreeMap中的元素按照自然排序进行排列。
TreeMap()

// 创建的TreeMap包含Map
TreeMap(Map<? extends K, ? extends V> copyFrom)

// 指定Tree的比较器
TreeMap(Comparator<? super K> comparator)

// 创建的TreeSet包含copyFrom
TreeMap(SortedMap<K, ? extends V> copyFrom)
    
// TreeMap的API
Entry<K, V>                ceilingEntry(K key)；
K                          ceilingKey(K key)；
void                       clear()；
Object                     clone()；
Comparator<? super K>      comparator()；
boolean                    containsKey(Object key)；
NavigableSet<K>            descendingKeySet()；
NavigableMap<K, V>         descendingMap()
Set<Entry<K, V>>           entrySet()；
Entry<K, V>                firstEntry()；
K                          firstKey()；
Entry<K, V>                floorEntry(K key)；
K                          floorKey(K key)；
V                          get(Object key)；
NavigableMap<K, V>         headMap(K to, boolean inclusive)；
SortedMap<K, V>            headMap(K toExclusive)；
Entry<K, V>                higherEntry(K key)；
K                          higherKey(K key)；
boolean                    isEmpty()；
Set<K>                     keySet()；
Entry<K, V>                lastEntry()；
K                          lastKey()；
Entry<K, V>                lowerEntry(K key)；
K                          lowerKey(K key)；
NavigableSet<K>            navigableKeySet()；
Entry<K, V>                pollFirstEntry()；
Entry<K, V>                pollLastEntry()；
V                          put(K key, V value)；
V                          remove(Object key)；
int                        size()；
SortedMap<K, V>            subMap(K fromInclusive, K toExclusive)
NavigableMap<K, V>         subMap(K from, boolean fromInclusive, K to, boolean toInclusive)；
NavigableMap<K, V>         tailMap(K from, boolean inclusive)；
SortedMap<K, V>            tailMap(K fromInclusive)；
```

![TreeMap结构](/images/treemap_struct.jpg)

- TreeMap实现继承于AbstractMap，并且实现了NavigableMap接口。
- TreeMap的本质是R-B Tree(红黑树)，它包含几个重要的成员变量： root, size, comparator。
    - root 是红黑数的根节点。它是Entry类型，Entry是红黑数的节点，它包含了红黑数的6个基本组成成分：key(键)、value(值)、left(左孩子)、right(右孩子)、parent(父节点)、color(颜色)。Entry节点根据key进行排序，Entry节点包含的内容为value。 
    - 红黑数排序时，根据Entry中的key进行排序；Entry中的key比较大小是根据比较器comparator来进行判断的。
    - size是红黑数中节点的个数。


### LinkedHashMap

- HashMap是无序的，HashMap在put时根据key的hascode进行hash，然后放入对应的地方。
- LinkedHashMap是Map接口的哈希表和链表实现，具有可预知的迭代顺序。允许Null值和Null键。
- LinkedHashMap是HashMap的子类，它保留插入的顺序。底层仍使用哈希表与链表保存所有元素，唯一不同是链表为双向链表，增加前一个元素的地址。
- LinkedHashMap可以通过插入顺序或者访问顺序来迭代其元素。

```java
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>{}

// 默认构造函数。使用该构造函数，LinkedHashMap中的元素按照插入顺序排列
LinkedHashMap();

// 设置LinkedHashMap的初始容量
LinkedHashMap(int initialCapacity) {}

//设置初始容量和加载因子
LinkedHashMap(int initialCapacity, float loadFactor) {}

//设置初始容量、加载因子和是否按访问顺序排列元素顺序
LinkedHashMap(int initialCapacity,
                         float loadFactor,
                         boolean accessOrder) {}

//API
void clear();
boolean containsValue(Object value);
Set<Entry<K, V>> entrySet();
V get();
V getOrDefault(Object key, V defaultValue);
Set<K> keySet();
Collection<V> values();
```

#### **LinkedHashMap 与 LRUcache**

由于LinkedHashMap支持设置`accessOrder`为true后，通过访问顺序来排序元素，因此可以实现**LRUcache( Last Recent Use cache最后访问缓存)**。

### WeakHashMap

- WeakHashMap和HashMap都是通过"拉链法"实现的散列表。它们的源码绝大部分内容都一样，这里就只是对它们不同的部分就是说明。
- WeakReference是“弱键”实现的哈希表。它这个“弱键”的目的就是：实现对“键值对”的动态回收。当“弱键”不再被使用到时，GC会回收它，WeakReference也会将“弱键”对应的键值对删除。
- “弱键”是一个**弱引用(WeakReference)**，在Java中，`WeakReference`和`ReferenceQueue `是联合使用的。
- WeakHashMap中亦是如此：如果弱引用所引用的对象被垃圾回收，Java虚拟机就会把这个弱引用加入到与之关联的引用队列中。 接着，WeakHashMap会根据“引用队列”，来删除WeakHashMap中已被GC回收的‘弱键’对应的键值对”。
- 另外，理解上面思想的重点是通过 `expungeStaleEntries()` 函数去理解。

```java
public class WeakHashMap<K,V>
    extends AbstractMap<K,V>
    implements Map<K,V> {}

// 默认构造函数。
WeakHashMap()

// 指定“容量大小”的构造函数
WeakHashMap(int capacity)

// 指定“容量大小”和“加载因子”的构造函数
WeakHashMap(int capacity, float loadFactor)

// 包含“子Map”的构造函数
WeakHashMap(Map<? extends K, ? extends V> map)
    
//API
void                   clear();
Object                 clone();
boolean                containsKey(Object key);
boolean                containsValue(Object value);
Set<Entry<K, V>>       entrySet();
V                      get(Object key);
boolean                isEmpty();
Set<K>                 keySet();
V                      put(K key, V value);
void                   putAll(Map<? extends K, ? extends V> map);
V                      remove(Object key);
int                    size();
Collection<V>          values();
```

### Map总结

![Map框架图](/images/map_struct.jpg)

- Map 是**映射接口**，Map中存储的内容是**键值对**(key-value)。
- AbstractMap 是**继承于Map的抽象类，它实现了Map中的大部分API**。其它Map的实现类可以通过继承AbstractMap来减少重复编码。
- SortedMap 是继承于Map的接口。SortedMap中的内容是**排序的键值对**，排序的方法是通过比较器(Comparator)。
- NavigableMap 是继承于SortedMap的接口。相比于SortedMap，NavigableMap有一系列的导航方法；如"获取大于/等于某对象的键值对"、“获取小于/等于某对象的键值对”等等。 
- HashMap, Hashtable, TreeMap, WeakHashMap这4个类是“键值对”映射的实现类

  - **HashMap** 是基于“拉链法”实现的散列表。一般用于单线程程序中。
  - **Hashtable** 也是基于“拉链法”实现的散列表。它一般用于多线程程序中。
  - **WeakHashMap** 也是基于“拉链法”实现的散列表，它一般也用于单线程程序中。相比HashMap，WeakHashMap中的键是“弱键”，当“弱键”被GC回收时，它对应的键值对也会被从WeakHashMap中删除；而HashMap中的键是强键。
  - **TreeMap** 是有序的散列表，它是通过红黑树实现的。它一般用于单线程中存储有序的映射。

## Set

### **HashSet**

- HashSet 是一个**没有重复元素的集合**。

- 基于HashMap实现，仅存储键，通过对象的hashCode()方法和equals()方法来保证键值不重复，允许Null值。

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable { }

// 默认构造函数
public HashSet() 

// 带集合的构造函数
public HashSet(Collection<? extends E> c) 

// 指定HashSet初始容量和加载因子的构造函数
public HashSet(int initialCapacity, float loadFactor) 

// 指定HashSet初始容量的构造函数
public HashSet(int initialCapacity) 

// 指定HashSet初始容量和加载因子的构造函数,dummy没有任何作用
HashSet(int initialCapacity, float loadFactor, boolean dummy) 

//API
boolean         add(E object);
void            clear();
Object          clone();
boolean         contains(Object object);
boolean         isEmpty();
Iterator<E>     iterator();
boolean         remove(Object object);
int             size();
```

### TreeSet

- TreeSet 是一个**有序的集合**。

- 基于TreeMap实现，仅存储键。其他属性和TreeMap类似

```java
public class TreeSet<E> extends AbstractSet<E>        
    implements NavigableSet<E>, Cloneable, java.io.Serializable{}

// 默认构造函数。使用该构造函数，TreeSet中的元素按照自然排序进行排列。
TreeSet()

// 创建的TreeSet包含collection
TreeSet(Collection<? extends E> collection)

// 指定TreeSet的比较器
TreeSet(Comparator<? super E> comparator)

// 创建的TreeSet包含set
TreeSet(SortedSet<E> set)
    
//API
boolean                   add(E object)；
boolean                   addAll(Collection<? extends E> collection)；
void                      clear()；
Object                    clone()；
boolean                   contains(Object object)；
E                         first()；
boolean                   isEmpty()；
E                         last()；
E                         pollFirst()；
E                         pollLast()；
E                         lower(E e)；
E                         floor(E e)；
E                         ceiling(E e)；
E                         higher(E e)；
boolean                   remove(Object object)；
int                       size()；
Comparator<? super E>     comparator()；
Iterator<E>               iterator()；
Iterator<E>               descendingIterator()；
SortedSet<E>              headSet(E end)；
NavigableSet<E>           descendingSet()；
NavigableSet<E>           headSet(E end, boolean endInclusive)；
SortedSet<E>              subSet(E start, E end)；
NavigableSet<E>           subSet(E start, boolean startInclusive, E end, boolean endInclusive)；
NavigableSet<E>           tailSet(E start, boolean startInclusive)；
SortedSet<E>              tailSet(E start)；    
```

### Set总结

![Set架构图](/images/set_struct.jpg)

- Set 是继承于Collection的接口。它是一个不允许有重复元素的集合。
- AbstractSet 是一个抽象类，它继承于AbstractCollection，AbstractCollection实现了Set中的绝大部分函数，为Set的实现类提供了便利。
- HastSet 和 TreeSet 是Set的两个实现类。
  - HashSet依赖于HashMap，它实际上是通过HashMap实现的。HashSet中的元素是无序的。
  - TreeSet依赖于TreeMap，它实际上是通过TreeMap实现的。TreeSet中的元素是有序的。

# Java Network

- **TCP/IP**

  - Socket

    ```java
    Socket socket = new Socket("192.168.1.1", 80);

    //发送数据
    OputputStream out = socket.getOutputStream();
    out.write("some data".getBytes());
    out.flush();
    out.close();

    //读取数据
    InputStream in = socket.getInputStream();
    int data = in.read();
    ...
    in.close();

    //关闭连接
    socket.close();
    ```

  - ServerSocket

    ```java
    ServerSocket serverSocket = new ServerSocket(80);
    boolean isStopped = false;
    while(!isStopped) {
      Socket clientSocket = serverSocket.accept(); //block
      //关闭客户端Socket
      clientSocket.close();
      //关闭服务端Socket
      serverSocket.close();
    }
    ```

- **UDP**

  - DatagramSocket

    ```java
    //发送数据
    byte[] buffer = new byte[65508]; //max length in a single UDP packet
    InetAddress address = InetAddress.getByName("192.168.1.1");
    DatagramPacket packet = new DatagramPacket(buffer, buffer.length, address, 80);
    DatagramSocket datagramSocket = new DatagramSocket();
    datagramSocket.send(packet);
    //接收数据
    DatagramSocket datagramSocket = new DatagramSocket(80);
    byte[] buffer = new byte[1024];
    DatagramPacket packet = new DatagramPacket(buffer,buffer.length);
    datagramSocket.receive(packet); //block
    byte[] receivedBuffer = packet.getData();
    ```

- **HttpURLConnection**

  ```java
  //Get请求
  URL url = new URL("http://www.baidu.com");
  HttpURLConnection urlConnection = (HttpURLConnection)url.openConnection();
  urlConnection.setRequestMethod("GET");
  InputStream input = urlConnection.getInputStream();
  int data = input.read();
  while(data != -1) {
    System.out.print((char) data);
    data = input.read();
  }
  //POST请求
  urlConnection.setDoOutput(true);
  urlConnection.setDoInput(true);
  urlConnection.setRequestMethod("POST");
  OutputStream output = urlConnection.getOutputStream();
  byte[] postData = new byte[1024];
  output.write(postData);
  output.close();

  input.close();
  ```

- **InetAddress/InetSocketAddress**

  - InetAddress

    ```java
    //获取IP地址
    InetAddress address = InetAddress.getByName("www.baidu.com");
    InetAddress address = InetAddress.getLocalHost();
    ```

  - InetSocketAddress

    ```java
    SocketAddress socketAddress = new InetSocketAddress(InentAddress.getLocalHost(), 80);
    ```