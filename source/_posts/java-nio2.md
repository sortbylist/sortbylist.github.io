---
title: Java NIO.2 回顾
date: 2017-02-09 10:47:58
tags: [Java,NIO.2]
---

### JAVA NIO.2

> 参考资料：[NIO.2 入门，第 1 部分: 异步通道 API](https://www.ibm.com/developerworks/cn/java/j-nio2-1/)、[NIO.2 入门，第 2 部分: 文件系统 API](http://www.ibm.com/developerworks/cn/java/j-nio2-2/index.html)

- Files
- Paths
- AsynchronousSocketChannel
- AsynchronousServerSocketChannel
- AsynchronousDatagramChannel
- AsynchronousFileChannel

#### Files、Paths

- Java 7中三个新的文件系统包

  - java.nio.file
  - java.nio.file.attribute
  - java.nio.file.spi

- 最有用的类

  - `java.nio.file.Files`、`java.nio.file.FileVisitor`在特定目录按深度优先遍历算法来查询文件或目录，并可对每个查询结果执行用户实现的回调方法。
  - `java.nio.file.Path`、`java.nio.file.WatchService`允许“注册”来监视特定目录。如果在目录中发生了文件创建、修改或者删除操作，监视目录的应用程序将收到通知。
  - `java.nio.attribute.*AttributeView`允许查询此前对于Java用户隐藏的文件和目录属性，这些属性包括文件所所有 者及组权限，访问控制列表（ACL），以及扩展文件属性。

    ```java
    FileVisitor<Path> fileVisitor = new SimpleFileVisitor<Path>() {
      @Override
      public FileVisitResult preVisitDirectory(Path dir) {
        System.out.println("visit dir[" + dir + "]");
        return FileVisitResult.CONTINUE;
      }
      @Override
      public FileVisitResult visitFile(Path file, BasicFileAttributes attribs) {
        System.out.println("visi file[" + file + "]'s size=" + attribs.size());
        return FileVisitResult.CONTINUE;
      }
    }
    Files.walkFileTree(path, fileVisitor);
    ```

- 目录监视

  WatchService API。

  ```java
  WatchService watchService = FileSystems.getDefault().newWatchService();
  File watchDirFile = new File("watchDir");
  Path watchDirPath = watchDirFile.toPath(); //FilesSystems.getDefault().getPath(watchDirFile.getPath())
  //Path watchDirPath = Paths.get("watchDir");
  WatchKey watchKey = watchDirPath.register(watchService, StandardWatchEventKind.ENTRY_CREATE, StandardWatchEventKind.ENTRY_MODIFY);
  //get watchkey
  WatchKey watchKey = watchService.take(); //blocking //watchService.poll()/watchService.poll(10,TimeUtil.SECONDS);
  for(WatchEvent<?> event : watchKey.pollEvents()) {
    System.out.println(event.kind() +" with " + event.context());
  }
  ```

- 文件属性

  使用`java.nio.file.attribute`包中的类获取并设置文件属性。

  该包中有7个属性视图（AtrributeView)，其中一些特定于操作系统。

  - `AclFileAttributeView`与`AclEntry`

    允许设置和获取特定文件的ACL（Access Control List）及文件所有者属性。这些属性视图仅可用于Windows系统。

    ```&gt;java
    Path path = Paths.get("c:", "users", "download");
    UserPrincipal joe = path.getFileSystem().getUserPrincipalLookupService().lookupPrincipalByName("joe");
    //get view
    AclFileAttributeView view = Files.getFileAttributeView(path, AclFileAttributeView.class);
    //create ACE to give "joe" read access
    AclEntry entry = AclEntry.newBuilder().setType(AclEntryType.ALLOW)
      .setPrincipal(joe)
      .setPermissions(AclEntryPermission.READ_DATA, AclEntryPermission.READ_ATTRIBUTES)
      .build();
    //read ACL, insert ACE, re-write ACL
    List<AclEntry> acl = view.getAcl();
    acl.add(0, entry); //insert before any DENY entries
    view.setAcl(acl);
    ```

  - `BasicFileAttributeView`与`BasicFileAttributes`

    允许获取一系列“平常的”基本文件属性。其中`readAttributes()`方法返回一 个`BasicFileAttributes`实例，该实例包含`最后修改时间`、`最后访问时间`、`创建时间`、`大小`、以及`文件属性`等细节（常规文件、目录、符号链接、或者其他）。这一属性视图在所有平台上均可用。

    ```java
    Path path = Paths.get("attributeFile");
    BasicFileAttributeView basicView = Files.getFileAttributeView(path, BasicFileAttributeView.class);
    BasicFileAttribute basicAttribute = basiceView.readAttributes();
    FileTime newModTime = FileTime.fromMillis(basicAttribute.lastModifiedTime().toMillis() + 60000);
    basicView.setTimes(newModTime, null, null); // only update modifyTime
    ```

  - `DosFileAttributeView`与`DosFileAttributes`

    这一视图类允许您获取指定给 DOS 的属性。（您可能会猜想，这一视图仅用于 Windows 系统。）其 `readAttributes()` 方法返回一个 `DosFileAttributes` 实例，该实例包含有问题的文件是否为只读、隐藏、系统文件、以及存档文件等细节信息。这一视图还包含针对每个属性的 `set*(boolean)` 方法。

  - `FileOwnerAttributeView`与`UserPrincipal`

    这一视图类允许您获取并设置特定文件的所有者。其`getOwner()`方法返回一个`UserPrincipal`，其getName()方法，返回所有者名字。该视图还提供`setOwner(UserPrincipal)`方法用于变更文件所有者。这一属性视图在所有平台上均可用。

  - `PosixFileAttributeView `与`PosixFileAttributes`

    允许获取并设置指定给POSIX（Portable Operationg System Interface）的属性。有关特定文件存储的信息。其`readAttributes()`方法返回一个包含有关这一文件的所有者、组所有者、以及这一文件许可（这些细节通常用UNIX　chmod命令设置）的`PosixFileAttributes`实例。这一视图还提供`setOwner(UserPrincipal)`、`setGroup(GroupPrincipal)`、以及`setPermissions(Set<PosixFilePermission>)`来修改这些属性。这一属性视图仅在UNIX系统上可用。

  - `FileStoreAttributeView`与`FileStore`

    允许您获取有关特定文件存储的信息。其 `readAttributes()` 方法返回一个包含文件存储的整个空间、未分配空间、以及已使用空间细节的 `FileStoreSpaceAttributes` 实例。这一视图在所有平台上都可用。

    ```java
    Path path = Paths.get("c:");
    FileStore fileStore = Files.getFileStore(path);
    fileStore.getTotalSpace(); 
    fileStore.getUnallocatedSpace();
    fileStore.getUsableSpace();
    fileStore.isReadOnly();
    fileStore.name();
    FileStoreAttributeView view = fileStore.getFileStoreAttributeView(FileStoreAttributeView.class);
    ```

  - `UserDefinedFileAttributeView`

    允许您获取并设置文件的**扩展属性**。这些属性跟其他的不同，它们只是名称值对，并可按需对其进行设置。如果想向文件增加一些隐藏的元数据，而不必修改文件内容，这就很有用了。通过`list()`方法，来返回相关扩展属性的名字列表。然后通过`size(String name)`返回属性值的大小，通过`read(String name, ByteBuffer dst)`来将属性值读取到ByteBuffer中，通过`write(String name, ByteBuffer source)`方法来创建或者修改属性，通过`delete(String name)`来移除现有属性。

    ```java
    Path path = Paths.get("c:");
    UserDefinedFileAttributeView userView = Files.getFileAttributeView(path, UserDefinedFileAttributeView.class);
    List<String> attribList = userView.list();
    ByteBuffer attribValue = ByteBuffer.allocate(userView.size(attribName));
    userView.read(attribName, attribValue); //读取
    CharsetDecoder decoder = Charset.forName("utf-8").newDecoder();
    CharBuffer buffer = decoder.decode(attribValue);
    userView.write(attribName, attribValue);
    ```


#### 异步通道

通道API提供2种对已启动异步操作的监测与控制机制。第一种是通过返回`java.util.concurrent.Future`对象来实现，它将会建模一个挂起操作，并可用于查询其状态以及获取结果。第二种通过传递给操作一个新类的对象`java.nio.channels.CompletionHandler`来完成，它会定义在操作完毕后所需要执行的内容。

- 异步套接字通道及特性

  ```java
  AsynchronousServerSocketChannel server = AsynchronousServerSocketChannel.open();
  server.bind(new InetSocketServer(5555));

  //Future 
  Future<AsynchronousSocketChannel> acceptFuture = server.accept(); //立即返回
  //if(!future.isDone()) {
  //  future.cancel(true);
  //}
  while(!future.isDone()); //waiting
  AsynchronousSocketChannel worker = future.get();
  //AsynchronousSocketChannel worker = future.get(10, TimeUnit.SECONDS); //等待10秒
  // read a message from the client
  worker.read(readBuffer).get(10, TimeUnit.SECONDS);
  System.out.println("Message: " + new String(readBuffer.array()));

  //CompletionHandler，第一个参数为accept返回值
  server.accept(null, new CompletionHandler(AsynchronousSocketChannel, Void) {
    public void completed(AsynchronousSocketChannel ch, Void att) {
     server.accept(null, this); //accept the next connection
      //handle this connection
      handle(ch);
    }
    public void failed(Throwable exc, Void att) {
      //...
    }
  });
  //client
  AsynchronousSocketChannel client = AsynchronousSocketChannel.open();
  client.connect(server.getLocalAddress());
  ByteBuffer message = ByteBuffer.wrap("ping".getBytes());
  int writeLength = client.write(message).get();
  ```

- 异步文件

  ```java
  AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(Paths.get("afile"), StandardOpenOption.READ, StandardOpenOption.WRITE, StandardOpenOption.CREATE, StandardOpenOption.DELETE_ON_CLOSE);

  fileChannel.write(ByteBuffer.wrap(bytes), 0, "Write operation 1", new CompletionHandler<Integer, Object>() {
    @Override
    public void completed(Integer result, Object attachment) {
      System.out.println(attachement + " completed with " + result + " bytes Written");
    }
    
    @Override
    public void failed(Throwable e, Object attachement) {
      System.err.println(attachment +" fialed with:");
      e.printStackTrace();
    }
  });
  ```

- 异步通道组

  每个异步通道都属于一个通道组，它们共享一个Java线程池，该线程池用于完成启动的异步I/O操作。
  默认情况下，具有 `open()` 方法的通道属于一个全局通道组，可利用如下系统变量对其进行配置：

  - `java.nio.channels.DefaultThreadPool.threadFactory`，其不采用默认设置，而是定义一个 `java.util.concurrent.ThreadFactory`
  - `java.nio.channels.DefaultThreadPool.initialSize`，指定线程池的初始规模

  `java.nio.channels.AsynchronousChannelGroup` 中的三个实用方法提供了创建新通道组的方法：

- `withCachedThreadPool()`
- `withFixedThreadPool()`


- `withThreadPool()`

  ```JAVA
    AsynchronousChannelGroup tenThreadGroup = AsynchronousChannelGroup.withFixedThreadPool(10, Executors.defaultThreadFactory());
    AysnchronousChannelGroup tenThreadGroup2 = AsynchronousChannelGroup.withCachedThreadPool(Executors.newCachedThreadPool(), 10);
    //使用 tenThreadGroup 而不是默认通道组来获取线程
    AsynchronousServerSocketChannel channel = 
        AsynchronousServerSocketChannel.open(tenThreadGroup);
  ```

  利用通道组来控制线程关闭

  ```java
    channel.accept(null, completionHandler);
    if(!tenThreadGroup.isShutdown()) {
      tenThreadGroup.shutdown();
    }
    if(!tenThreadGroup.isTerminated()) {
      tenThreadGroup.shutdownNow();
    }
    tenThreadGroup.awaitTermination(10, TimeUnit.SECONDS);
  ```

  AsynchronousFileChannel的open()方法使用ExecutorService而不是AsynchronousChannelGroup来实现定制的线程池。

- 异步数据报通道与多播