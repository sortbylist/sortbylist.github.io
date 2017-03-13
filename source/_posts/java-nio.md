---
title: Java NIO回顾
date: 2017-02-08 17:29:05
tags: [Java,NIO]
---

### Java NIO

- Channels通道（连接）
  - FileChannel
  - DatagramChannel
  - SocketChannel
  - ServerSocketChannel
  - Pipe.SinkChannel
  - Pipe.SourceChannel
- Buffers缓冲区
  - ByteBuffer
  - CharBuffer
  - DoubleBuffer
  - FloatBuffer
  - LongBuffer
  - IntBuffer
  - ShortBuffer
  - MappedByteBuffer
- Selectors
- Pipe（管道）

#### FileChannel

FileChannel是一个连接到文件的通道。可以通过文件通道读写文件。

FileChannel无法设置为非阻塞模式，它总是运行在阻塞模式下。

> 在Java NIO中，如果两个通道中有一个是FileChannel，那你可以直接将数据从一个channel（译者注：channel中文常译作通道）传输到另外一个channel。

- FileChannel的基本用法

  ```java
  RandomAccessFile aFile = new RandomAccessFile("data/nio-data.txt", "rw");
  FileChannel inChannel = aFile.getChannel(); //获取FileChannel

  ByteBuffer buf = ByteBuffer.allocate(48);
  int bytesRead = inChannel.read(buf); //读取FileChannel中数据至ByteBuffer
  long pos = inChannel.position(); //获取FileChannel的当前位置
  inChannel.position(0); //设置FileChannel的当前位置,在此位置之后进行读或写
  buf.flip(); //ByteBuffer从写模式转到读模式
  inChannel.write(buf);//再将ByteBuffer中数据写入FileChannel
  long fileSize = inChannel.size(); //返回此通道的文件的当前大小
  inChannel.truncate(fileSize / 2); //截取文件
  inChannel.force(true); //强制数据写入磁盘，参数表示是否同时将文件元数据（权限信息等）写到磁盘上。
  inChannel.close(); //关闭
  ```

- transferFrom()

  > FileChannel的transferFrom()方法可以将数据从源通道传输到FileChannel中（译者注：这个方法在JDK文档中的解释为将字节从给定的可读取字节通道传输到此通道的文件中）。

  ```java
  RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
  FileChannel fromChannel = fromFile.getChannel();

  RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
  FileChannel toChannel = toFile.getChannel();

  long position = 0;
  long count = fromChannel.size();
  toChannel.transferFrom(fromChannel, position, count);
  ```

  > 此外要注意，在SoketChannel的实现中，SocketChannel只会传输此刻准备好的数据（可能不足count字节）。因此，SocketChannel可能不会将请求的所有数据(count个字节)全部传输到FileChannel中。

- transferTo()

  > transferTo()方法将数据从FileChannel传输到其他的channel中。

  ```java
  RandomAccessFile fromFile = new RandomAccessFile("fromFile.txt", "rw");
  FileChannel fromChannel = formFile.getChannel();

  RandomAccessFile toFile = new RandomAccessFile("toFile.txt", "rw");
  FileChannel toChannel = toFile.getChannel();

  long position = 0;
  long count=  fromChannel.size();
  fromChannel.transferTo(position, count, toChannel);
  ```

#### SocketChannel

> Java NIO中的SocketChannel是一个连接到TCP网络套接字的通道。可以通过以下2种方式创建SocketChannel：
> 1. 打开一个SocketChannel并连接到互联网上的某台服务器。
> 2. 一个新连接到达ServerSocketChannel时，会创建一个SocketChannel。

- SocketChannel基本用法

  ```java
  //通过调用系统级默认 SelectorProvider 对象的 openSocketChannel 方法来创建新的通道。 
  SocketChannel socketChannel = SocketChannel.open(); //打开套接字通道
  //如果通道处于非阻塞模式，则发起一个非阻塞连接操作。如果立即建立连接（使用本地连接时就是如此），则返回true。否则此方法返回 false，并且必须在以后通过调用 finishConnect 方法来完成该连接操作。
  boolean isConnectNow = socketChannel.connect(new InetSocketAddress("www.baidu.com"), 80); //连接到某个套接字
  if(!isConnectNow) {
    socketChannel.finishConnect();
  }
  ByteBuffer buf = ByteBuffer.allocate(48);
  int bytesRead = socketChannel.read(buf);//从SocketChannel读取数据到ByteBuffer
  buf.flip();//反转ByteBuffer,此时limit=position,postion=0，一般与compact()配合使用
  socketChannel.write(buf); //写入SocketChannel;
  ```

- SocketChannel非阻塞模式

  > 非阻塞模式与选择器搭配会工作的更好，通过将一或多个SocketChannel注册到Selector，可以询问选择器哪个通道已经准备好了读取，写入等。

  ```java
  socketChannel.configureBlocking(false);
  socketChannel.connect(new InetSocketAddress("http://baidu.com"), 80);
  while(!socketChannel.finishConnect()) {
    //waitting for connecting...
  }
  ByteBuffer buf = ByteBuffer.allocate(48);
  socketChannel.read(buf);

  while(buf.hasRemaing()) {
    socketChannel.write(buf);
  }
  ```

#### ServerSocketChannel

针对面向流的侦听套接字的可选择通道。 

- ServerSocketChannel基本用法

  ```java
  ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
  serverSocketChannel.socket().bind(new InetSocketAddress(9999));

  serverSocketChannel.configureBlocking(false); //非阻塞模式
  while(true) {
    SocketChannel socketChannel = serverSocketChannel.accept();
    //...
    if(socketChannel != null) {
      
    }
  }
  serverSocketChannel.close();
  ```

#### DatagramChannel

> Java NIO中的DatagramChannel是一个能收发UDP包的通道。因为UDP是无连接的网络协议，所以不能像其它通道那样读取和写入。它发送和接收的是数据包。

- DatagramChannel基本用法

  ```java
  DatagramChannel channel = DatagramChannel.open();
  channel.socket().bind(new InetSocketAddress(9999));

  ByteBuffer buf = ByteBuffer.allocate(48);
  buf.clear();
  channel.receive(buf); //接收数据，如果buf大小不够，则会被丢弃

  buf.flip();
  int bytesSend = channel.send(buf, new InetSocketAddress("baidu.com", 80)); //发送数据
  //连接到特定的地址，让其只能从特定地址收发数据，减少安全检查开销
  channel.connect(new InetSocketAddress("baidu.com", 80));
  int bytesRead = channel.read(buf);
  int bytesWritten = channel.write(buf);
  ```

  ​

#### ByteBuffer

数据从通道读入缓冲区，从缓冲区写入到通道。

缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。

> 缓冲区是特定基本类型元素的线性有限序列。除内容外，缓冲区的基本属性还包括容量、限制和位置

- ByteBuffer的基本用法

  1. 写入数据到ByteBuffer
  2. 调用`flip()`方法
  3. 从ByteBuffer中读取数据
  4. 调用`clear()`方法或者`compact()`方法

  ```java
  RandomAccessFile file = new RandomAccessFile("data/data.txt", "rw");
  FileChannel fileChannel = file.getChannel();
  //分配
  ByteBuffer buffer = ByteBuffer.allocate(1024);
  int bytesRead = fileChannel.read(buffer);
  while(bytesRead != -1) {
    buffer.flip(); //ready to read
    while(buffer.hasRemaining()) {
      System.out.print((char)buffer.get());
    }
    buffer.clear(); //ready for write
    bytesRead = fileChannel.read(buffer);
  }
  file.close();
  ```

- 0<=mask<=position<=limit<=capacity

- Buffer分配与包装

  - ByteBuffer buffer = ByteBuffer.allocate(1024)
  - ByteBuffer buffer = ByteBuffer.wrap(new byte[1024])

- Buffer写入

  - fileChannel.write(buffer)
  - buffer.put(127)

- Buffer读取

  - buffer.get()、buffer.get(byte[])
  - buffer.rewind()，令position=0，limit保持不变，表示重读Buffer

- Buffer重写

  - buffer.clear()，（未真正）清空buffer，准备重写，此时position=0，limit=capacity。
  - buffer.compact()，将未读数据拷贝到Buffer起始处，然后令position=size+1,limit=capacity

- Buffer标记

  - buffer.mark()，标记当前positon，令mark=position
  - buffer.reset() ，重置position，令postion=mark

- Buffer分片

  slice()方法根据现有的缓冲区创建一个子缓冲区。大小为Buffer的剩余空间，从当前位置开始共享数据。

  ```java
  ByteBuffer buffer = ByteBuffer.allocate(10);
  for(int i=0; i< buffer.capacity();i++) {
    buffer.put((byte)i);
  }
  buffer.position(3);
  buffer.limit(7);
  ByteBuffer slice = buffer.slice();
  for(int i=0; i < slice.capacity();i++) {
    byte b = slice.get(i);
    b * = 11;
    slice.put(i, b);
  }
  buffer.position(0);
  buffer.limit(buffer.capacity());
  while(buffer.remaining()>0) {
    System.out.println(buffer.get()); //buffer中元素被改变了
  }
  ```

  > 缓冲区片对于促进抽象非常有帮助。可以编写自己的函数处理整个缓冲区，而且如果想要将这个过程应用于子缓冲区上，您只需取主缓冲区的一个片，并将它传递给您的函数。这比编写自己的函数来取额外的参数以指定要对缓冲区的哪一部分进行操作更容易。

- 只读Buffer

  ```java
  ByteBuffer readBuffer = buffer.asReadOnlyBuffer();
  ```

- 直接和间接Buffer

  > 字节缓冲区要么是*直接的*，要么是*非直接的*。如果为直接字节缓冲区，则 Java 虚拟机会尽最大努力直接在此缓冲区上执行本机 I/O 操作。也就是说，在每次调用基础操作系统的一个本机 I/O 操作之前（或之后），虚拟机都会尽量避免将缓冲区的内容复制到中间缓冲区中（或从中间缓冲区中复制内容）。

  ```java
  ByteBuffer directBuffer = ByteBuffer.allocateDirect(1024);
  ```

- 内存映射文件Buffer

  ```java
  FileChannel fileChannel = file.getChannel();
  MappedByteBuffer mbb = fileChannel.map(FileChannel.MapMode.READ_WRITE, 0, 1024);
  ```

- Buffer比较

  - buffer.equals()，满足：相同类型(byte/char/int等)，剩余的（postion至limit之间的）byte/char等的个数相等，所有剩余的byte/char等都相同时，equals返回true。
  - buffer.compareTo()，如果满足下列条件，则认为一个Buffer“小于”另一个Buffer：
    1. 第一个不相等的元素小于另一个Buffer中对应的元素 。
    2. 所有元素都相等，但第一个Buffer比另一个先耗尽(第一个Buffer的元素个数比另一个少)。

#### Scatter/Gatter

> 分散（scatter）从Channel中读取是指在读操作时将读取的数据写入多个buffer中。因此，Channel将从Channel中读取的数据“分散（scatter）”到多个Buffer中。

> 聚集（gather）写入Channel是指在写操作时将多个buffer的数据写入同一个Channel，因此，Channel 将多个Buffer中的数据“聚集（gather）”后发送到Channel。

> scatter / gather经常用于需要将传输的数据分开处理的场合，例如传输一个由消息头和消息体组成的消息，你可能会将消息体和消息头分散到不同的buffer中，这样你可以方便的处理消息头和消息体。

- Scattering Reads

  数据从一个channel读取到多个buffer中。

  Scattering Reads在移动下一个buffer前，必须填满当前的buffer，这也意味着它不适用于动态消息。

  ```java
  ByteBuffer header = ByteBuffer.allocate(128);
  ByteBuffer body = ByteBuffer.allocate(1024);

  ByteBuffer[] bufferArray = {head, body};
  //按顺序将将从channel中读取的数据写入到buffer
  ////按顺序将将从channel中读取的数据写入到buffer
  channel.read(bufferArray);
  ```

- Gattering Writes

  数据从多个buffer写入到同一个channel。

  Gathering Writes能较好的处理动态消息。

  ```java
  ByteBuffer header = ByteBuffer.allocate(128);
  ByteBuffer body   = ByteBuffer.allocate(1024);

  //write data into buffers

  ByteBuffer[] bufferArray = { header, body };

  channel.write(bufferArray);
  ```

#### 文件锁

> 文件锁定要么是*独占的*，要么是*共享的*。

```java
RandomAccessFile file = new RandomAccessFile("lockfile.txt","rw");
FileChannel channel = file.getChannel();
FileLock lock = file.lock(start, end ,false);
//do someting
lock.release();
```



#### Selector

> Selector（选择器）是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。这样，一个单独的线程可以管理多个channel，从而管理多个网络连接。

Selector可以监听Channel的4种类型的事件：`Connect`、`Accept`、`Read`、`Write`。通道触发了一个事件就表示该事件已经就绪。这4种事件用SelectionKey的4个常量分别表示为：`SelectionKey.OP_CONNECT`、`SelectionKey.OP_ACCEPT`、`SelectionKey.OP_READ`、`SelectionKey.OP_WRITE`。

SelectionKye由Channel向Selector注册后返回，包含以下属性：`interest集合`、`ready集合`、`Channel`、`Selector`、`附加的对象（可选）`。

- selector基本使用

  ```java
  Selector selector = Selector.open(); //创建
  SocketChannel socketChannel = SocketChannel.open();
  socketChannel.configureBlocking(false); //非阻塞模式
  socketChannel.connect(new InetSocketAddress("www.baidu.com", 80));
  //注册Channel到Selector
  SelectionKey selectionKey = socketChannel.register(selector, SelectionKey.OP_READ); 
  ```

- interest集合

  ```java
      int interestSet = selectionKey.interestOps();
      boolean isInterestedInAccept = (interestSet & SelectionKey.OP_ACCEPT) == SelectionKey.OP_ACCEPT;
      boolean isInterestedInConnect = (interestSet & SelectionKey.OP_CONNECT) == SelectionKey.OP_CONNECT;
      boolean isInterestedInRead    = (interestSet & SelectionKey.OP_READ) == SelectionKey.OP_READ;
      boolean isInterestedInWrite   = (interestSet & SelectionKey.OP_WRITE) == SelectionKey.OP_WRITE;
  ```

- ready集合

  ```java
      int readySet = selectionKey.readyOps();
      selectionKey.isAcceptable();
      selectionKey.isConnectable();
      selectionKey.isReadable();
      selectionKey.isWritable();
  ```

- Channel + Selector

  ```java
      Channel channel = selectionKey.channel();
      Selector selector = selectionKey.selector();
  ```

- 附加对象

  ```java
      selectionKey.attach(theObject);
      Object attactedObj = selectionKey.attachment();
      //或者在注册时附加
      SelectionKey key = channel.register(selector, SelectionKey.OP_READ, theObject); 
  ```

- 通过Selector选择通道

  ```
  可能通过以下几种select()重载方法去选择相关事件已经准备就绪的通道。
  ```

  - int select()阻塞到至少有一个通道在你注册的事件上就绪了
  - int select(long timeout)和select()一样，除了最长会阻塞timeout毫秒(参数)
  - int selectNow()不会阻塞，不管什么通道就绪都立刻返回

- selectedKeys()

  一旦调用了select()方法，并且返回值表明有一个或多个通道就绪，则可以调用selector的selectedKey()方法，访问“已选择键集(selected key set)”中的就绪通道。

  ```java
      Set selectedKeys = selector.selectedKeys();
      Iterator keyIterator = selectedKeys.iterator();
      while(keyIterator.hasNext()) {
        Selectionkey key = keyIterator.next();
        if(key.isAcceptable()) {
          // a connection was accepted by a ServerSocketChannel.
        } else if(key.isConnectable()) {
          // a connection was established with a remote server.
        } else if(key.isReadable()) {
          // a channel is ready for reading
        } else if(key.isWritable()) {
          // a channel is ready for writing
        }
        keyIterator.remove();//手动移除。下次该通道变成就绪时，Selector会再次将其放入已选择键集中
      }
  ```

- wakeUp()

  ```
  某个线程调用select()方法后阻塞了，即使没有通道已经就绪，调用此方法即可让其立即返回。
  ```

- close()

  ```
  关闭Selector，且使注册到该Selector上的所有SelectionKey实例无效。通道本身并不会关闭。
  ```

- 完整示例

  ```java
      SocketChannel socketChannel = SocketChannel.open();
      socketChannel.configureBlocking(false);
      socketChannel.connect(new InetSocketAddress("www.baidu.com", 80));

      Selector selector = Selector.open();
      SelectionKey selectionKey = socketChannel.register(selector, SelectionKey.OP_READ | SelectionKey.OP_WRITE);
      while(true) {
        int readyChannels = selector.select();
        if(readyChannels == 0) continue;
        Set selectedKeys = selector.selectedKeys();
        Iterator keyIterator = selectedKeys.iterator();
        while(keyIterator.hasNext()) {
          SelectionKey key = (SelectionKey) keyIterator.next();
          if(key.readyOps() & SelectionKey.OP_READ) {
            //...
          }
          keyIterator.remove();
        }
      }
  ```

  ```java
      import java.io.IOException;
      import java.net.InetSocketAddress;
      import java.net.ServerSocket;
      import java.nio.ByteBuffer;
      import java.nio.channels.SelectionKey;
      import java.nio.channels.Selector;
      import java.nio.channels.ServerSocketChannel;
      import java.nio.channels.SocketChannel;
      import java.util.Iterator;
      import java.util.Set;

      /**
  * 
  * 异步 I/O 中的核心对象名为 Selector。
  * 
  * @author yangwm Apr 30, 2010 5:14:55 PM
  */
  public class MultiPortEcho {

   private int ports[];
   private ByteBuffer echoBuffer = ByteBuffer.allocate(1024);

   public MultiPortEcho(int ports[]) throws IOException {
     this.ports = ports;

     go();
   }

   private void go() throws IOException {
     // Create a new selector
     Selector selector = Selector.open();

     // Open a listener on each port, and register each one
     // with the selector
     for (int i = 0; i < ports.length; ++i) {
       ServerSocketChannel ssc = ServerSocketChannel.open();
       ssc.configureBlocking(false);
       ServerSocket ss = ssc.socket();
       InetSocketAddress address = new InetSocketAddress(ports[i]);
       ss.bind(address);

       SelectionKey key = ssc.register(selector, SelectionKey.OP_ACCEPT);

       System.out.println("Going to listen on " + ports[i]);
     }

     while (true) {
       int num = selector.select();

       if (num > 0) {

         Set selectedKeys = selector.selectedKeys();
         Iterator it = selectedKeys.iterator();

         while (it.hasNext()) {
           SelectionKey key = (SelectionKey) it.next();

           if ((key.readyOps() & SelectionKey.OP_ACCEPT) == SelectionKey.OP_ACCEPT) {
             // Accept the new connection
             ServerSocketChannel ssc = (ServerSocketChannel) key.channel();
             SocketChannel sc = ssc.accept();
             sc.configureBlocking(false);

             // Add the new connection to the selector
             SelectionKey newKey = sc.register(selector, SelectionKey.OP_READ);

             System.out.println("Got connection from " + sc);
             it.remove();
           } else if ((key.readyOps() & SelectionKey.OP_READ) == SelectionKey.OP_READ) {
             // Read the data
             SocketChannel sc = (SocketChannel) key.channel();

             // Echo data
             int bytesEchoed = 0;
             while (true) {
               echoBuffer.clear();

               int r = sc.read(echoBuffer);

               if (r <= 0) {
                 break;
               }

               echoBuffer.flip();

               sc.write(echoBuffer);
               bytesEchoed += r;
             }

             System.out.println("Echoed " + bytesEchoed + " from " + sc);
             it.remove();
           }

         }

         //System.out.println("going to clear");
         //selectedKeys.clear();
         //System.out.println("cleared");

       }
     }
   }

   /**
    * @param args
    */
   public static void main(String[] args) throws Exception {
     if (args.length <= 0) {
       System.err.println("Usage: java MultiPortEcho port [port port ...]");
       System.exit(1);
     }

     int ports[] = new int[args.length];

     for (int i = 0; i < args.length; ++i) {
       ports[i] = Integer.parseInt(args[i]);
     }

     new MultiPortEcho(ports);
   }
  }
  //在现实的应用程序中，您需要通过将通道从 Selector 中删除来处理关闭的通道。而且您可能要使用多个线程。这个程序可以仅使用一个线程，因为它只是一个演示，但是在现实场景中，创建一个线程池来负责 I/O 事件处理中的耗时部分会更有意义。
  ```

#### Pipe

> Java NIO管道是2个线程之间的**单向**数据连接。Pipe有一个source通道和一个sink通道。数据会被写到sink通道，从source通道读取。

- Pipe基本用法

  ```java
  Pipe pipe = Pipe.open(); //打开管道
  Pipe.SinkChannel sinkChannel = pipe.sink(); //获取写入通道
  ByteBuffer buf = ByteBuffer.allocate(1024);
  buf.put("i can tell".getBytes());
  buf.flip();
  while(buf.hasRemaining()) {
    sinkChannel.write(buf); //写入通道
  }
  Pipe.SourceChannel sourceChannel = pipe.source();
  buf.clear();
  sourceChannel.read(buf); //读取通道数据
  ```

  ​

#### Java NIO和IO

- IO
  - 面向流
  - 阻塞IO
- NIO
  - 面向缓冲
  - 非阻塞IO
  - 选择器

#### Java NIO：Non-blocking Server非阻塞服务器

- Github地址：[java-nio-server](https://github.com/jjenkov/java-nio-server)

- 阻塞IO--->线程阻塞，一个连接一个线程---->高并发时服务器过多线程，服务无响应---->使用线程池，空闭的线程仍然阻塞，无法释放----->加大线程池的core线程数，不具有伸缩性----->最终仍没办法解决高并发的情况。

- 非阻塞IO---->线程不阻塞---->结合Selector，通过单一线程可以管理多个请求------>有效解决高并发的情况。

  - 读取不完整消息，等到得到完整消息后，通过通道下发到其他组件进行处理。

  - 存储不完整的消息（Storing Partial Messages）

    - 拷贝消息时数据量尽可能小；以顺序的字节存储。

    - 不完整的消息存储在内部Buffer中，这个buffer必须能存储下至少一个消息最大的大小，但为了性能，需要这个buffer容量可变。

    - 拷贝扩容（Resize by Copy)

      先分配4KB的空间，如果大于4KB，就再分配 8KB空间，并将4KB的内容拷贝到8KB空间中去。

    - 追加扩容（Resize by Append)
      让buffer包含几个数组。当需要扩容的时候只需要再开辟一个新的字节数组，然后把内容写到里面去。一种是开辟单独的字节数组，然后用一个列表把这些独立数组关联起来。另一种是开辟一些更大的，相互共享的字节数组切片，然后用列表把这些切片和buffer关联起来。

- TLV编码消息（TLV Encoded Messages）

  TLV格式（Type，Length，Value）。消息的完整大小存储在消息的头部。我们可以立即为消息分配相应内存。缺点是少量链接慢，数据大，会占用较多内存。

  - 解决办法是使用一种内部含多个TLV的消息格式。这样为每个TLV段分配内存而不是整个消息。但消息片段很大时，仍然会出现占用内存问题。
  - 另一种方法是设置超时，长时间未收到消息则断开连接。
  - HTTP2.0利用TLV编码来传输数据帧。

- 写入不完整的消息（Writing Partial Messages）

  - 当有消息需要写时，将需要被写入的Channel注册到Selector
  - 当服务器空闭时，检查Selector是否有Channel等待被写入，然后将数据写入Channel，如果写入完成，将Channel从Selector上解绑。

- 一个非阻塞的服务器需要时刻检查当前是否有消息被完整送到。

  - 一个非阻塞的顺需要时刻检查是否有数据需要被写。如果有，服务器需要检查相应的连接是否准备好被写。
  - 总之，一个非阻塞的顺需要以下三个管道操作，并且经常执行：
    - 读管道，查询打开的连接是否有新的数据发送进来。
    - 处理管道，处理完整消息。
    - 写管道，查询消息是否能被写入相应的连接。

- 服务器线程模型（Server Thread Model）

  - 包含2条线程。
  - 第一条线程负责ServerSocketChannel接收到达的连接。
  - 第二条线程负责处理这些连接，包括读消息，处理消息 ，把响应写回连接。​