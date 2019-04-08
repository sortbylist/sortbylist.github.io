---

title: Java IO
date: 2019-03-27 10:18:11
tags: [Java,IO]
---

> 参考:
>
> [Java IO 考点及资料整理](https://segmentfault.com/a/1190000013695159#articleHeader13)
>
> [深入分析 Java I/O 的工作机制](https://www.ibm.com/developerworks/cn/java/j-lo-javaio/index.html)

`java.io`包主要涉及文件访问、网络数据流、内存缓冲访问、线程内部通信（管道）、缓冲、过滤、解析、读写文件（Readers/Writers）、读写基本类型数据（long，int etc）、读写对象等输入输出。

# 字节流

## `InputStream`字节输入流

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

### InputStream

```java
public abstract class InputStream implements Closeable {
             int     available();
             void    close();
             void    mark(int readlimit);
             boolean markSupported();
             int     read(byte[] buffer);
abstract     int     read();
             int     read(byte[] buffer, int offset, int length);
synchronized void    reset();
             long    skip(long byteCount);
}
```

### ByteArrayInputStream

ByteArrayInputStream 是字节数组输入流。它继承于InputStream。
它包含一个内部缓冲区，该缓冲区包含从流中读取的字节；通俗点说，它的内部缓冲区就是一个字节数组，而ByteArrayInputStream本质就是通过字节数组来实现的。
我们都知道，InputStream通过read()向外提供接口，供它们来读取字节数据；而ByteArrayInputStream 的内部额外的定义了一个计数器，它被用来跟踪 read() 方法要读取的下一个字节。

```java
// 构造函数;
ByteArrayInputStream(byte[] buf);
ByteArrayInputStream(byte[] buf, int offset, int length);

synchronized int         available();
             void        close();
synchronized void        mark(int readlimit);
             boolean     markSupported();
synchronized int         read();
synchronized int         read(byte[] buffer, int offset, int length);
synchronized void        reset();
synchronized long        skip(long byteCount);
```

ByteArrayInputStream实际上是通过“字节数组”去保存数据。

- 通过ByteArrayInputStream(byte buf[]) 或 ByteArrayInputStream(byte buf[], int offset, int length) ，我们可以根据buf数组来创建字节流对象。
- read()的作用是从字节流中“读取下一个字节”。
- read(byte[] buffer, int offset, int length)的作用是从字节流读取字节数据，并写入到字节数组buffer中。offset是将字节写入到buffer的起始位置，length是写入的字节的长度。
- markSupported()是判断字节流是否支持“标记功能”。它一直返回true。
- mark(int readlimit)的作用是记录标记位置。记录标记位置之后，某一时刻调用reset()则将“字节流下一个被读取的位置”重置到“mark(int readlimit)所标记的位置”；也就是说，reset()之后再读取字节流时，是从mark(int readlimit)所标记的位置开始读取。

### PipedInputStream

在java中，**PipedOutputStream**和**PipedInputStream**分别是管道输出流和管道输入流。
它们的作用是让多线程可以通过管道进行线程间的通讯。在使用管道通信时，必须将PipedOutputStream和PipedInputStream配套使用。
使用管道通信时，大致的流程是：我们在线程A中向PipedOutputStream中写入数据，这些数据会自动的发送到与PipedOutputStream对应的PipedInputStream中，进而存储在PipedInputStream的缓冲中；此时，线程B通过读取PipedInputStream中的数据。就可以实现，线程A和线程B的通信。

PipedInputStream类中属性有`closedByWriter`、`closedByReader`、`connected`、`readSide`、`writeSide`、`buffer`、`in`、`out`，分别代码被写入线程关闭连接，被读取线程关闭连接，是否连接，读取线程，写入线程，内部缓冲byte数组，写入数，读取数。通过协调读取和写入线程，一方面从PipedOutputStream接收数据，一方面当调用`read()`、`read(byte[], int, int)`方法是从内部缓存byte数组中读取数据。

### ObjectInputStream

ObjectInputStream 和 ObjectOutputStream 的作用是，**对基本数据和对象进行序列化操作支持。**

当我们需要读取ObjectOutputStream存储的“基本数据或对象”时，可以创建“文件输入流”对应的ObjectInputStream，进而读取出这些“基本数据或对象”。

注意： 只有支持 java.io.Serializable 或 java.io.Externalizable 接口的对象才能被ObjectInputStream/ObjectOutputStream所操作！

```java
// 构造函数;
ObjectInputStream(InputStream input);

//public函数;
int     available();
void     close();
void     defaultReadObject();
int     read(byte[] buffer, int offset, int length);
int     read();
boolean     readBoolean();
byte     readByte();
char     readChar();
double     readDouble();
ObjectInputStream.GetField     readFields();
float     readFloat();
void     readFully(byte[] dst);
void     readFully(byte[] dst, int offset, int byteCount);
int     readInt();
String     readLine();
long     readLong();
final Object     readObject();
short     readShort();
String     readUTF();
Object     readUnshared();
int     readUnsignedByte();
int     readUnsignedShort();
synchronized void     registerValidation(ObjectInputValidation object, int priority);
int     skipBytes(int length);
```

### FileInputStream

FileInputStream 是文件输入流，它继承于InputStream。
通常，我们使用FileInputStream从某个文件中获得输入字节。

```java
FileInputStream(File file); // 构造函数1：创建“File对象”对应的“文件输入流”
FileInputStream(FileDescriptor fd); // 构造函数2：创建“文件描述符”对应的“文件输入流”
FileInputStream(String path); // 构造函数3：创建“文件(路径为path)”对应的“文件输入流”

int available(); // 返回“剩余的可读取的字节数”或者“skip的字节数”
void close(); // 关闭“文件输入流”
FileChannel getChannel(); // 返回“FileChannel”
final FileDescriptor getFD(); // 返回“文件描述符”
int read(); // 返回“文件输入流”的下一个字节
int read(byte[] buffer, int byteOffset, int byteCount); // 读取“文件输入流”的数据并存在到buffer，从byteOffset开始存储，存储长度是byteCount。
long skip(long byteCount); // 跳过byteCount个字节
```
### FilterInputStream

**FilterInputStream** 的作用是用来“**封装其它的输入流，并为它们提供额外的功能**”。它的常用的子类有BufferedInputStream和DataInputStream。

**BufferedInputStream**的作用就是为“输入流提供缓冲功能，以及mark()和reset()功能”。

**DataInputStream** 是用来装饰其它输入流，它“允许应用程序以与机器无关方式从底层输入流中读取基本 Java 数据类型”。应用程序可以使用DataOutputStream(数据输出流)写入由DataInputStream(数据输入流)读取的数据。

**FilterInputStream**很明显使用了**装饰器模式 ( Decorator )**。

### BufferedInputStream

BufferedInputStream 是缓冲输入流。它继承于**FilterInputStream**。

BufferedInputStream 的作用是为另一个输入流添加一些功能，例如，提供“缓冲功能”以及支持“mark()标记”和“reset()重置方法”。

BufferedInputStream 本质上是通过一个内部缓冲区数组实现的。例如，在新建某输入流对应的BufferedInputStream后，当我们通过read()读取输入流的数据时，BufferedInputStream会将该输入流的数据分批的填入到缓冲区中。每当缓冲区中的数据被读完之后，输入流会再次填充数据缓冲区；如此反复，直到我们读完输入流数据位置。

```java
BufferedInputStream(InputStream in);
BufferedInputStream(InputStream in, int size);

synchronized int     available();
void     close();
synchronized void     mark(int readlimit);
boolean     markSupported();
synchronized int     read();
synchronized int     read(byte[] buffer, int offset, int byteCount);
synchronized void     reset();
synchronized long     skip(long byteCount);
```

### DataInputStream

DataInputStream 是数据输入流。它继承于FilterInputStream。

DataInputStream 是**用来装饰其它输入流，它“允许应用程序以与机器无关方式从底层输入流中读取基本 Java 数据类型”。**应用程序可以使用DataOutputStream(数据输出流)写入由DataInputStream(数据输入流)读取的数据。

```java
DataInputStream(InputStream in); 
final int read(byte[] buffer, int offset, int length); 
final int read(byte[] buffer); 
final boolean readBoolean(); 
final byte readByte(); 
final char readChar(); 
final double readDouble(); 
final float readFloat(); 
final void readFully(byte[] dst); 
final void readFully(byte[] dst, int offset, int byteCount); 
final int readInt(); 
final String readLine(); 
final long readLong(); 
final short readShort(); 
final static String readUTF(DataInput in); 
final String readUTF(); 
final int readUnsignedByte(); 
final int readUnsignedShort(); 
final int skipBytes(int count); 
```


## `OutputStream`字节输出流

- ByteArrayOutputStream
- FileOutputStream
- FilterOutputStream
  - BufferedOutputStream
  - DataOutputStream
  - PrintStream
- ObjectOutputStream
- PipedOutputStream

### OutputStream

```java
public abstract class OutputStream implements Closeable, Flushable {
         void    close();
         void    flush();
         void    write(byte[] buffer, int offset, int count);
         void    write(byte[] buffer);
abstract void    write(int oneByte);
}
```

### ByteArrayOutputStream

ByteArrayOutputStream 是字节数组输出流。它继承于OutputStream。
ByteArrayOutputStream 中的数据被写入一个 byte 数组。缓冲区会随着数据的不断写入而自动增长，“新容量”的初始化 = “旧容量”x2。可使用 toByteArray() 和 toString() 获取数据。

```java
// 构造函数;
ByteArrayOutputStream();
ByteArrayOutputStream(int size);

             void    close();
synchronized void    reset();
             int     size();
synchronized byte[]  toByteArray();
             String  toString(int hibyte);
             String  toString(String charsetName);
             String  toString();
synchronized void    write(byte[] buffer, int offset, int len);
synchronized void    write(int oneByte);
synchronized void    writeTo(OutputStream out);
```

ByteArrayOutputStream实际上是将字节数据写入到“字节数组”中去。

- 通过ByteArrayOutputStream()创建的“字节数组输出流”对应的字节数组大小是32。
- 通过ByteArrayOutputStream(int size) 创建“字节数组输出流”，它对应的字节数组大小是size。
- write(int oneByte)的作用将int类型的oneByte换成byte类型，然后写入到输出流中。
- write(byte[] buffer, int offset, int len) 是将字节数组buffer写入到输出流中，offset是从buffer中读取数据的起始偏移位置，len是读取的长度。
- writeTo(OutputStream out) 将该“字节数组输出流”的数据全部写入到“输出流out”中。

### PipedOutputStream

在java中，**PipedOutputStream**和**PipedInputStream**分别是管道输出流和管道输入流。
它们的作用是让多线程可以通过管道进行线程间的通讯。在使用管道通信时，必须将PipedOutputStream和PipedInputStream配套使用。
使用管道通信时，大致的流程是：我们在线程A中向PipedOutputStream中写入数据，这些数据会自动的发送到与PipedOutputStream对应的PipedInputStream中，进而存储在PipedInputStream的缓冲中；此时，线程B通过读取PipedInputStream中的数据。就可以实现，线程A和线程B的通信。

PipedOutputStream类中持有PipedInputStream实例，调用write/flush等方法时均对PipedInputStream实例进行操作。

### ObjectOutputStream

ObjectInputStream 和 ObjectOutputStream 的作用是，**对基本数据和对象进行序列化操作支持。**

创建“文件输出流”对应的ObjectOutputStream对象，该ObjectOutputStream对象能提供对“基本数据或对象”的持久存储。

注意： 只有支持 java.io.Serializable 或 java.io.Externalizable 接口的对象才能被ObjectInputStream/ObjectOutputStream所操作！

 ```java
// 构造函数;
ObjectOutputStream(OutputStream output);

// public函数;
void     close();
void     defaultWriteObject();
void     flush();
ObjectOutputStream.PutField     putFields();
void     reset();
void     useProtocolVersion(int version);
void     write(int value);
void     write(byte[] buffer, int offset, int length);
void     writeBoolean(boolean value);
void     writeByte(int value);
void     writeBytes(String value);
void     writeChar(int value);
void     writeChars(String value);
void     writeDouble(double value);
void     writeFields();
void     writeFloat(float value);
void     writeInt(int value);
void     writeLong(long value);
final void     writeObject(Object object);
void     writeShort(int value);
void     writeUTF(String value);
void     writeUnshared(Object object);
 ```

### FileOutputStream

FileOutputStream 是文件输出流，它继承于OutputStream。

通常，我们使用FileOutputStream 将数据写入 File 或 FileDescriptor 的输出流。

```java
FileOutputStream(File file); // 构造函数1：创建“File对象”对应的“文件输入流”；默认“追加模式”是false，即“写到输出的流内容”不是以追加的方式添加到文件中。
FileOutputStream(File file, boolean append); // 构造函数2：创建“File对象”对应的“文件输入流”；指定“追加模式”。
FileOutputStream(FileDescriptor fd); // 构造函数3：创建“文件描述符”对应的“文件输入流”；默认“追加模式”是false，即“写到输出的流内容”不是以追加的方式添加到文件中。
FileOutputStream(String path); // 构造函数4：创建“文件(路径为path)”对应的“文件输入流”；默认“追加模式”是false，即“写到输出的流内容”不是以追加的方式添加到文件中。
FileOutputStream(String path, boolean append); // 构造函数5：创建“文件(路径为path)”对应的“文件输入流”；指定“追加模式”。

void close(); // 关闭“输出流”
FileChannel getChannel(); // 返回“FileChannel”
final FileDescriptor getFD(); // 返回“文件描述符”
void write(byte[] buffer, int byteOffset, int byteCount); // 将buffer写入到“文件输出流”中，从buffer的byteOffset开始写，写入长度是byteCount。
void write(int oneByte); // 写入字节oneByte到“文件输出流”中
```

### FilterOutputStream

FilterOutputStream 的作用是用来“封装其它的输出流，并为它们提供额外的功能”。它主要包括BufferedOutputStream, DataOutputStream和PrintStream。

BufferedOutputStream的作用就是为“输出流提供缓冲功能”。

DataOutputStream 是用来装饰其它输出流，将DataOutputStream和DataInputStream输入流配合使用，“允许应用程序以与机器无关方式从底层输入流中读写基本 Java 数据类型”。

PrintStream 是用来装饰其它输出流。它能为其他输出流添加了功能，使它们能够方便地打印各种数据值表示形式。

**FilteroutputStream**很明显使用了**装饰器模式 ( Decorator )**。

### BufferedOutputStream

BufferedOutputStream 是缓冲输出流。它继承于FilterOutputStream。

BufferedOutputStream 的作用是为另一个输出流提供“缓冲功能”。

```java
BufferedOutputStream(OutputStream out); 
BufferedOutputStream(OutputStream out, int size); 

synchronized void close(); 
synchronized void flush(); 
synchronized void write(byte[] buffer, int offset, int length); 
synchronized void write(int oneByte); 
```

BufferedOutputStream的源码非常简单，这里就BufferedOutputStream的思想进行简单说明：BufferedOutputStream通过字节数组来缓冲数据，当缓冲区满或者用户调用flush()函数时，它就会将缓冲区的数据写入到输出流中。

### DataOutputStream

**DataOutputStream** 是数据输出流。它继承于FilterOutputStream。

**DataOutputStream** 是用来装饰其它输出流，将DataOutputStream和**DataInputStream**输入流配合使用，“允许应用程序以与机器无关方式从底层输入流中读写基本 Java 数据类型”。

### PrintStream

PrintStream 是打印输出流，它继承于FilterOutputStream。

PrintStream 是用来装饰其它输出流。它能为其他输出流添加了功能，使它们能够方便地打印各种数据值表示形式。

与其他输出流不同，PrintStream 永远不会抛出 IOException；它产生的IOException会被自身的函数所捕获并设置错误标记， 用户可以通过 checkError() 返回错误标记，从而查看PrintStream内部是否产生了IOException。

另外，**PrintStream 提供了自动flush 和 字符集设置功能**。所谓自动flush，就是往PrintStream写入的数据会立刻调用flush()函数。

```java
/* 
* 构造函数
*/
// 将“输出流out”作为PrintStream的输出流，不会自动flush，并且采用默认字符集
// 所谓“自动flush”，就是每次执行print(); , println(); , write(); 函数，都会调用flush(); 函数；
// 而“不自动flush”，则需要我们手动调用flush(); 接口。
PrintStream(OutputStream out); 
// 将“输出流out”作为PrintStream的输出流，自动flush，并且采用默认字符集。
PrintStream(OutputStream out, boolean autoFlush); 
// 将“输出流out”作为PrintStream的输出流，自动flush，采用charsetName字符集。
PrintStream(OutputStream out, boolean autoFlush, String charsetName); 
// 创建file对应的FileOutputStream，然后将该FileOutputStream作为PrintStream的输出流，不自动flush，采用默认字符集。
PrintStream(File file); 
// 创建file对应的FileOutputStream，然后将该FileOutputStream作为PrintStream的输出流，不自动flush，采用charsetName字符集。
PrintStream(File file, String charsetName); 
// 创建fileName对应的FileOutputStream，然后将该FileOutputStream作为PrintStream的输出流，不自动flush，采用默认字符集。
PrintStream(String fileName); 
// 创建fileName对应的FileOutputStream，然后将该FileOutputStream作为PrintStream的输出流，不自动flush，采用charsetName字符集。
PrintStream(String fileName, String charsetName); 

// 将“字符c”追加到“PrintStream输出流中”
PrintStream append(char c); 
// 将“字符序列从start(包括); 到end(不包括); 的全部字符”追加到“PrintStream输出流中”
PrintStream append(CharSequence charSequence, int start, int end); 
// 将“字符序列的全部字符”追加到“PrintStream输出流中”
PrintStream append(CharSequence charSequence); 
// flush“PrintStream输出流缓冲中的数据”，并检查错误
boolean checkError(); 
// 关闭“PrintStream输出流”
synchronized void close(); 
// flush“PrintStream输出流缓冲中的数据”。
// 例如，PrintStream装饰的是FileOutputStream，则调用flush时会将数据写入到文件中
synchronized void flush(); 
// 根据“Locale值(区域属性); ”来格式化数据
PrintStream format(Locale l, String format, Object... args); 
// 根据“默认的Locale值(区域属性); ”来格式化数据
PrintStream format(String format, Object... args); 
// 将“float数据f对应的字符串”写入到“PrintStream输出流”中，print实际调用的是write函数
void print(float f); 
// 将“double数据d对应的字符串”写入到“PrintStream输出流”中，print实际调用的是write函数
void print(double d); 
// 将“字符串数据str”写入到“PrintStream输出流”中，print实际调用的是write函数
synchronized void print(String str); 
// 将“对象o对应的字符串”写入到“PrintStream输出流”中，print实际调用的是write函数
void print(Object o); 
// 将“字符c对应的字符串”写入到“PrintStream输出流”中，print实际调用的是write函数
void print(char c); 
// 将“字符数组chars对应的字符串”写入到“PrintStream输出流”中，print实际调用的是write函数
void print(char[] chars); 
// 将“long型数据l对应的字符串”写入到“PrintStream输出流”中，print实际调用的是write函数
void print(long l); 
// 将“int数据i对应的字符串”写入到“PrintStream输出流”中，print实际调用的是write函数
void print(int i); 
// 将“boolean数据b对应的字符串”写入到“PrintStream输出流”中，print实际调用的是write函数
void print(boolean b); 
// 将“数据args”根据“Locale值(区域属性); ”按照format格式化，并写入到“PrintStream输出流”中
PrintStream printf(Locale l, String format, Object... args); 
// 将“数据args”根据“默认Locale值(区域属性); ”按照format格式化，并写入到“PrintStream输出流”中
PrintStream printf(String format, Object... args); 
// 将“换行符”写入到“PrintStream输出流”中，println实际调用的是write函数
void println(); 
// 将“float数据对应的字符串+换行符”写入到“PrintStream输出流”中，println实际调用的是write函数
void println(float f); 
// 将“int数据对应的字符串+换行符”写入到“PrintStream输出流”中，println实际调用的是write函数
void println(int i); 
// 将“long数据对应的字符串+换行符”写入到“PrintStream输出流”中，println实际调用的是write函数
void println(long l); 
// 将“对象o对应的字符串+换行符”写入到“PrintStream输出流”中，println实际调用的是write函数
void println(Object o); 
// 将“字符数组chars对应的字符串+换行符”写入到“PrintStream输出流”中，println实际调用的是write函数
void println(char[] chars); 
// 将“字符串str+换行符”写入到“PrintStream输出流”中，println实际调用的是write函数
synchronized void println(String str); 
// 将“字符c对应的字符串+换行符”写入到“PrintStream输出流”中，println实际调用的是write函数
void println(char c); 
// 将“double数据对应的字符串+换行符”写入到“PrintStream输出流”中，println实际调用的是write函数
void println(double d); 
// 将“boolean数据对应的字符串+换行符”写入到“PrintStream输出流”中，println实际调用的是write函数
void println(boolean b); 
// 将数据oneByte写入到“PrintStream输出流”中。oneByte虽然是int类型，但实际只会写入一个字节
synchronized void write(int oneByte); 
// 将“buffer中从offset开始的length个字节”写入到“PrintStream输出流”中。
void write(byte[] buffer, int offset, int length); 
```

### System.out.Println

- out的定义

  一个PrintStream实例

  ```java
  public final class System {
  public final static PrintStream out = null;
  }
  ```

- out的初始化

  ```java
  pubilc final class System {
  private static void initializeSystemClass() {
      //省略其他代码...
    FileOutputStream fdOut = new FileOutputStream(FileDescriptor.out);
    setOut0(new PrintStream(new BufferedOutputStream(fdOut, 128), true));
    }
  }
  ```

- FileDescriptor.out是啥

  ```java
  public final class FileDescriptor {
  private int fd;
  /**
  * A handle to the standard output stream. Usually, this file
  * descriptor is not used directly, but rather via the output stream
  * known as {@code System.out}.
  * @see java.lang.System#out
  */
  public static final FileDescriptor out = new FileDescriptor(1);
      private FileDescriptor(int fd) {
      this.fd = fd;
      useCount = new AtomicInteger();
      }
  }
  ```

  FileDescriptor.out是一个FileDescriptor实例，其fd=1，正是标准输出的标识符。

- setOut0方法

  setOut0()是一个native本地方法，其功能是设置out为传入的参数

  ```java
  public final class System {
  private static native void setOut0(PrintStream out);
  }
  ```

- System.out.println

  所以System.out其实是标准输出的PrintStream实例，而println方法正是调用的PrintStream的println方法。这样PrintStream中的多个输出方法均可以通过System.out调用。

# 字符流

## `Reader`字符输入流

- BufferedReader
  - LineNumberReader
- CharArrayReader
- FilterReader
- PushbackReader
- InputStreamReader
  - FileReader
- PipedReader
- StringReader

### Reader

```java
public abstract class Reader implements Readable, Closeable {
	protected Reader();
    protected Reader(Object lock);
    public int read(Charbuffer target) throws IOException;
    public int read() throws IOException;
    public int read(char cbuf[]) throws IOException;
    abstract public int read(char cbuf[], int off, int len) throws IOException;
    public long skip(long n) throws IOException;
    public boolean read() throws IOException;
    public boolean markSupported();
    public void mark(int readAheadLimit) throws IOException;
    public void reset() throws IOException;
    abstract public void close() throws IOException;
}
```

### CharArrayReader

CharArrayReader 是字符数组输入流。它和**ByteArrayInputStream**类似，只不过ByteArrayInputStream是字节数组输入流，而CharArray是字符数组输入流。CharArrayReader 是用于读取字符数组，它继承于Reader。操作的数据是以字符为单位！

```java
CharArrayReader(char[] buf);
CharArrayReader(char[] buf, int offset, int length);

void      close();
void      mark(int readLimit);
boolean   markSupported();
int       read();
int       read(char[] buffer, int offset, int len);
boolean   ready();
void      reset();
long      skip(long charCount);
```
### PipedReader

PipedReader 是字符管道输入流，它继承于Writer。

PipedWriter和PipedReader的作用是可以通过管道进行线程间的通讯。在使用管道通信时，必须将PipedWriter和PipedReader配套使用。

```java
public PipedReader();
public PipedReader(int pipeSize);
public PipedReader(PipedWriter src) throws IOException;
public PipedReader(PipedWriter src, int pipeSize) throws IOException;

public void close()  throws IOException;
public void connect(PipedWriter src) throws IOException;
public synchronized int read()  throws IOException;
public synchronized int read(char cbuf[], int off, int len)  throws IOException;
public synchronized boolean ready() throws IOException;
```

### InpustreamReader

InputStreamReader和OutputStreamWriter 是字节流通向字符流的桥梁：它使用指定的 charset 读写字节并将其解码为字符。

InputStreamReader 的作用是将“字节输入流”转换成“字符输入流”。它继承于Reader。

```java
public InputStreamReader(InputStream in);
public InputStreamReader(InputStream in, Charset cs);
public InputStreamReader(InputStream in, CharsetDecoder dec);
public InputStreamReader(InputStream in, String charsetName) throws UnsupportedEncodingException;

public void close() throws IOException;
public String getEncoding();
public int read(char cbuf[], int offset, int length) throws IOException;
public int read() throws IOException;
public boolean ready() throws IOException;
```

InputStreamReader内部持有一个`StreamDecoder sd`的私有属性，大部分的功能由`StreamDecoder`实现。

### FileReader

FileReader 是用于读取字符流的类，它继承于InputStreamReader。要读取原始字节流，请考虑使用 FileInputStream。

```java
package java.io;

public class FileReader extends InputStreamReader {

    public FileReader(String fileName) throws FileNotFoundException {
        super(new FileInputStream(fileName));
    }

    public FileReader(File file) throws FileNotFoundException {
        super(new FileInputStream(file));
    }

    public FileReader(FileDescriptor fd) {
        super(new FileInputStream(fd));
    }
}
```

### BufferedReader

BufferedReader 是缓冲字符输入流。它继承于Reader。

BufferedReader 的作用是为其他字符输入流添加一些缓冲功能。

```java
BufferedReader(Reader in);
BufferedReader(Reader in, int size);

void     close();
void     mark(int markLimit);
boolean  markSupported();
int      read();
int      read(char[] buffer, int offset, int length);
String   readLine();
boolean  ready();
void     reset();
long     skip(long charCount);
```

## `Writer`字符输出流

- BufferedWriter
- CharArrayWriter
- FilterWriter
- OutputStreamWriter
  - FileWriter
- PipedWriter
- PrintWriter
- StringWriter

### Writer

```java
public abstract class Writer implements Appendable, Closeable, Flushable {
	protected Writer();
    protected Writer(Object lock);
    public void write(int c) throws IOException;
    public void write(char cbuf[]) throws IOException;
    abstract public void write(char cbuf[], int off, int len) throws IOException;
    public void write(String str) throws IOException;
    public void write(String str, int off, int len) throws IOException;
    public Writer append(Chrequence csq) throws IOException;
    public Writer apend(CharSequence csq, int start, int end) throws IOException;
    public Write append(char c) throws IOException;
    abstract public void flush() throws IOException;
    abstract public void close() throws IOException;
}
```

### CharArrayWriter

CharArrayReader 用于写入数据符，它继承于Writer。操作的数据是以字符为单位！

```java
CharArrayWriter();
CharArrayWriter(int initialSize);

CharArrayWriter     append(CharSequence csq, int start, int end);
CharArrayWriter     append(char c);
CharArrayWriter     append(CharSequence csq);
void     close();
void     flush();
void     reset();
int     size();
char[]     toCharArray();
String     toString();
void     write(char[] buffer, int offset, int len);
void     write(int oneChar);
void     write(String str, int offset, int count);
void     writeTo(Writer out);
```

### PipedWrite

PipedWriter 是字符管道输出流，它继承于Writer。

PipedWriter和PipedReader的作用是可以通过管道进行线程间的通讯。在使用管道通信时，必须将PipedWriter和PipedReader配套使用。

```java
public PipedWriter();
public PipedWriter(PipedReader snk)  throws IOException;

public void close()  throws IOException;
public synchronized void connect(PipedReader snk) throws IOException;
public synchronized void flush() throws IOException;
public void write(char cbuf[], int off, int len) throws IOException;
public void write(int c)  throws IOException;
```

### OutputStreamWriter

InputStreamReader和OutputStreamWriter 是字节流通向字符流的桥梁：它使用指定的 charset 读写字节并将其解码为字符。

OutputStreamWriter 的作用是将“字节输出流”转换成“字符输出流”。它继承于Writer。

```java
public OutputStreamWriter(OutputStream out);
public OutputStreamWriter(OutputStream out, Charset cs);
public OutputStreamWriter(OutputStream out, CharsetEecoder enc);
public OutputStreamWriter(OutputStream out, String charsetName) throws UnsupportedEncodingException;

public void close() throws IOException;
public void flush() throws IOException;
public String getEncoding();
public int write(char cbuf[], int offset, int length) throws IOException;
public int write(int c) throws IOException;
public boolean write(String str, int off, int len) throws IOException;
```

OutputStreamWriter内部持有一个`StreamEecoder se`的私有属性，大部分的功能由`StreamDecoder`实现。

### FileWriter

FileWriter 是用于写入字符流的类，它继承于OutputStreamWriter。要写入原始字节流，请考虑使用 FileOutputStream。

```java
package java.io;

public class FileWriter extends OutputStreamWriter {

    public FileWriter(String fileName) throws IOException {
        super(new FileOutputStream(fileName));
    }

    public FileWriter(String fileName, boolean append) throws IOException {
        super(new FileOutputStream(fileName, append));
    }

    public FileWriter(File file) throws IOException {
        super(new FileOutputStream(file));
    }

    public FileWriter(File file, boolean append) throws IOException {
        super(new FileOutputStream(file, append));
    }

    public FileWriter(FileDescriptor fd) {
        super(new FileOutputStream(fd));
    }
}
```

### BufferedWriter

BufferedWriter 是缓冲字符输出流。它继承于Writer。
BufferedWriter 的作用是为其他字符输出流添加一些缓冲功能。

```java
// 构造函数
BufferedWriter(Writer out); 
BufferedWriter(Writer out, int sz); 
 
void    close();                              // 关闭此流，但要先刷新它。
void    flush();                              // 刷新该流的缓冲。
void    newLine();                            // 写入一个行分隔符。
void    write(char[] cbuf, int off, int len); // 写入字符数组的某一部分。
void    write(int c);                         // 写入单个字符。
void    write(String s, int off, int len);    // 写入字符串的某一部分。
```

### PrintWriter

PrintWriter 是字符类型的打印输出流，它继承于Writer。

PrintStream 用于向文本输出流打印对象的格式化表示形式。它实现在 **PrintStream** 中的所有 print 方法。它不包含用于写入原始字节的方法，对于这些字节，程序应该使用未编码的字节流进行写入。

```java
PrintWriter(OutputStream out);
PrintWriter(OutputStream out, boolean autoFlush);
PrintWriter(Writer wr);
PrintWriter(Writer wr, boolean autoFlush);
PrintWriter(File file);
PrintWriter(File file, String csn);
PrintWriter(String fileName);
PrintWriter(String fileName, String csn);

PrintWriter     append(char c);
PrintWriter     append(CharSequence csq, int start, int end);
PrintWriter     append(CharSequence csq);
boolean     checkError();
void     close();
void     flush();
PrintWriter     format(Locale l, String format, Object... args);
PrintWriter     format(String format, Object... args);
void     print(float fnum);
void     print(double dnum);
void     print(String str);
void     print(Object obj);
void     print(char ch);
void     print(char[] charArray);
void     print(long lnum);
void     print(int inum);
void     print(boolean bool);
PrintWriter     printf(Locale l, String format, Object... args);
PrintWriter     printf(String format, Object... args);
void     println();
void     println(float f);
void     println(int i);
void     println(long l);
void     println(Object obj);
void     println(char[] chars);
void     println(String str);
void     println(char c);
void     println(double d);
void     println(boolean b);
void     write(char[] buf, int offset, int count);
void     write(int oneChar);
void     write(char[] buf);
void     write(String str, int offset, int count);
void     write(String str);
```

# File

## File

File 是“**文件**”和“**目录路径名**”的抽象表示形式。

File 直接继承于Object，实现了Serializable接口和Comparable接口。实现Serializable接口，意味着File对象支持序列化操作。而实现Comparable接口，意味着File对象之间可以比较大小；File能直接被存储在有序集合(如TreeSet、TreeMap中)。

```java
// 静态成员
public static final String pathSeparator; // 路径分割符":"
public static final char pathSeparatorChar; // 路径分割符':'
public static final String separator; // 分隔符"/"
public static final char separatorChar; // 分隔符'/'

// 构造函数
File(File dir, String name);
File(String path);
File(String dirPath, String name);
File(URI uri);

// 成员函数
boolean canExecute(); // 测试应用程序是否可以执行此抽象路径名表示的文件。
boolean canRead(); // 测试应用程序是否可以读取此抽象路径名表示的文件。
boolean canWrite(); // 测试应用程序是否可以修改此抽象路径名表示的文件。
int compareTo(File pathname); // 按字母顺序比较两个抽象路径名。
boolean createNewFile(); // 当且仅当不存在具有此抽象路径名指定名称的文件时，不可分地创建一个新的空文件。
static File createTempFile(String prefix, String suffix); // 在默认临时文件目录中创建一个空文件，使用给定前缀和后缀生成其名称。
static File createTempFile(String prefix, String suffix, File directory); // 在指定目录中创建一个新的空文件，使用给定的前缀和后缀字符串生成其名称。
boolean delete(); // 删除此抽象路径名表示的文件或目录。
void deleteOnExit(); // 在虚拟机终止时，请求删除此抽象路径名表示的文件或目录。
boolean equals(Object obj); // 测试此抽象路径名与给定对象是否相等。
boolean exists(); // 测试此抽象路径名表示的文件或目录是否存在。
File getAbsoluteFile(); // 返回此抽象路径名的绝对路径名形式。
String getAbsolutePath(); // 返回此抽象路径名的绝对路径名字符串。
File getCanonicalFile(); // 返回此抽象路径名的规范形式。
String getCanonicalPath(); // 返回此抽象路径名的规范路径名字符串。
long getFreeSpace(); // 返回此抽象路径名指定的分区中未分配的字节数。
String getName(); // 返回由此抽象路径名表示的文件或目录的名称。
String getParent(); // 返回此抽象路径名父目录的路径名字符串；如果此路径名没有指定父目录，则返回 null。
File getParentFile(); // 返回此抽象路径名父目录的抽象路径名；如果此路径名没有指定父目录，则返回 null。
String getPath(); // 将此抽象路径名转换为一个路径名字符串。
long getTotalSpace(); // 返回此抽象路径名指定的分区大小。
long getUsableSpace(); // 返回此抽象路径名指定的分区上可用于此虚拟机的字节数。
int hashCode(); // 计算此抽象路径名的哈希码。
boolean isAbsolute(); // 测试此抽象路径名是否为绝对路径名。
boolean isDirectory(); // 测试此抽象路径名表示的文件是否是一个目录。
boolean isFile(); // 测试此抽象路径名表示的文件是否是一个标准文件。
boolean isHidden(); // 测试此抽象路径名指定的文件是否是一个隐藏文件。
long lastModified(); // 返回此抽象路径名表示的文件最后一次被修改的时间。
long length(); // 返回由此抽象路径名表示的文件的长度。
String[] list(); // 返回一个字符串数组，这些字符串指定此抽象路径名表示的目录中的文件和目录。
String[] list(FilenameFilter filter); // 返回一个字符串数组，这些字符串指定此抽象路径名表示的目录中满足指定过滤器的文件和目录。
File[] listFiles(); // 返回一个抽象路径名数组，这些路径名表示此抽象路径名表示的目录中的文件。
File[] listFiles(FileFilter filter); // 返回抽象路径名数组，这些路径名表示此抽象路径名表示的目录中满足指定过滤器的文件和目录。
File[] listFiles(FilenameFilter filter); // 返回抽象路径名数组，这些路径名表示此抽象路径名表示的目录中满足指定过滤器的文件和目录。
static File[] listRoots(); // 列出可用的文件系统根。
boolean mkdir(); // 创建此抽象路径名指定的目录。
boolean mkdirs(); // 创建此抽象路径名指定的目录，包括所有必需但不存在的父目录。
boolean renameTo(File dest); // 重新命名此抽象路径名表示的文件。
boolean setExecutable(boolean executable); // 设置此抽象路径名所有者执行权限的一个便捷方法。
boolean setExecutable(boolean executable, boolean ownerOnly); // 设置此抽象路径名的所有者或所有用户的执行权限。
boolean setLastModified(long time); // 设置此抽象路径名指定的文件或目录的最后一次修改时间。
boolean setReadable(boolean readable); // 设置此抽象路径名所有者读权限的一个便捷方法。
boolean setReadable(boolean readable, boolean ownerOnly); // 设置此抽象路径名的所有者或所有用户的读权限。
boolean setReadOnly(); // 标记此抽象路径名指定的文件或目录，从而只能对其进行读操作。
boolean setWritable(boolean writable); // 设置此抽象路径名所有者写权限的一个便捷方法。
boolean setWritable(boolean writable, boolean ownerOnly); // 设置此抽象路径名的所有者或所有用户的写权限。
String toString(); // 返回此抽象路径名的路径名字符串。
URI toURI(); // 构造一个表示此抽象路径名的 file: URI。
URL toURL(); // 已过时。 此方法不会自动转义 URL 中的非法字符。建议新的代码使用以下方式将抽象路径名转换为 URL：首先通过 toURI 方法将其转换为 URI，然后通过 URI.toURL 方法将 URI 装换为 URL。
```
## FileDescriptor

FileDescriptor 是“文件描述符”。
FileDescriptor 可以被用来表示开放文件、开放套接字等。
以FileDescriptor表示文件来说：当FileDescriptor表示某文件时，我们可以通俗的将FileDescriptor看成是该文件。但是，我们不能直接通过FileDescriptor对该文件进行操作；若需要通过FileDescriptor对该文件进行操作，则需要新创建FileDescriptor对应的FileOutputStream，再对文件进行操作。

FileDescriptor 类中三个类属性`in`/`out`/`err`，分别为标准输入/标准输出/标准错误，Java提供了 `System.in`/`System.out`/`System.error`对应这三个属性，以简化对其的使用。

```java

/**
* A handle to the standard input stream. Usually, this file
* descriptor is not used directly, but rather via the input stream
* known as {@code System.in}.
*
* @see java.lang.System#in
*/
public static final FileDescriptor in = standardStream(0);

/**
* A handle to the standard output stream. Usually, this file
* descriptor is not used directly, but rather via the output stream
* known as {@code System.out}.
* @see java.lang.System#out
*/
public static final FileDescriptor out = standardStream(1);

/**
* A handle to the standard error stream. Usually, this file
* descriptor is not used directly, but rather via the output stream
* known as {@code System.err}.
*
* @see java.lang.System#err
*/
public static final FileDescriptor err = standardStream(2);
```

## RandomAccessFile

RandomAccessFile 是随机访问文件(包括读/写)的类。它支持对文件随机访问的读取和写入，即我们可以从指定的位置读取/写入文件数据。
需要注意的是，RandomAccessFile 虽然属于java.io包，但它不是InputStream或者OutputStream的子类；它也不同于FileInputStream和FileOutputStream。 FileInputStream 只能对文件进行读操作，而FileOutputStream 只能对文件进行写操作；但是，RandomAccessFile 同时支持文件的读和写，并且它支持随机访问。

```java
public class RandomAccessFile implements DataOutput, DataInput, Closeable {

    RandomAccessFile(File file, String mode);
    RandomAccessFile(String fileName, String mode);

    void     close();
    synchronized final FileChannel     getChannel();
    final FileDescriptor     getFD();
    long     getFilePointer();
    long     length();
    int     read(byte[] buffer, int byteOffset, int byteCount);
    int     read(byte[] buffer);
    int     read();
    final boolean     readBoolean();
    final byte     readByte();
    final char     readChar();
    final double     readDouble();
    final float     readFloat();
    final void     readFully(byte[] dst);
    final void     readFully(byte[] dst, int offset, int byteCount);
    final int     readInt();
    final String     readLine();
    final long     readLong();
    final short     readShort();
    final String     readUTF();
    final int     readUnsignedByte();
    final int     readUnsignedShort();
    void     seek(long offset);
    void     setLength(long newLength);
    int     skipBytes(int count);
    void     write(int oneByte);
    void     write(byte[] buffer, int byteOffset, int byteCount);
    void     write(byte[] buffer);
    final void     writeBoolean(boolean val);
    final void     writeByte(int val);
    final void     writeBytes(String str);
    final void     writeChar(int val);
    final void     writeChars(String str);
    final void     writeDouble(double val);
    final void     writeFloat(float val);
    final void     writeInt(int val);
    final void     writeLong(long val);
    final void     writeShort(int val);
    final void     writeUTF(String str);
}
```

RandomAccessFile共有4种模式："r", "rw", "rws"和"rwd"。

```shell
"r"    以只读方式打开。调用结果对象的任何 write 方法都将导致抛出 IOException。  
"rw"   打开以便读取和写入。
"rws"  打开以便读取和写入。相对于 "rw"，"rws" 还要求对“文件的内容”或“元数据”的每个更新都同步写入到基础存储设备。  
"rwd"  打开以便读取和写入，相对于 "rw"，"rwd" 还要求对“文件的内容”的每个更新都同步写入到基础存储设备。  
```

**"rw", "rws", "rwd" 的区别。**
  当操作的文件是存储在本地的基础存储设备上时(如硬盘, NandFlash等)，"rws" 或 "rwd", "rw" 才有区别。
  当模式是 "rws" 并且 操作的是基础存储设备上的文件；那么，每次“更改文件内容[如write()写入数据]” 或 “修改文件元数据(如文件的mtime)”时，都会将这些改变同步到基础存储设备上。
  当模式是 "rwd" 并且 操作的是基础存储设备上的文件；那么，每次“更改文件内容[如write()写入数据]”时，都会将这些改变同步到基础存储设备上。
  当模式是 "rw" 并且 操作的是基础存储设备上的文件；那么，关闭文件时，会将“文件内容的修改”同步到基础存储设备上。至于，“更改文件内容”时，是否会立即同步，取决于系统底层实现。