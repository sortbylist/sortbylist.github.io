---
title: WebService基础回顾
date: 2017-02-09 11:23:08
tags: [Java,WebService]
---

# WebService

### 定义

> W3C 组织对其的定义如下，它是一个软件系统，为了支持跨网络的机器间相互操作交互而设计。Web Service 服务通常被定义为一组模块化的 API，它们可以通过网络进行调用，来执行远程系统的请求服务。

Web Service = SOAP + HTTP + WSDL。

其中SOAP （ Simple Object Access Protocol ）协议是 Web Service 主体，它通过 HTTP 或者 SMTP 等应用层协议进行通讯，自身使用 XML  文件来描述程序的函数方法和参数信息，从而完成不同主机的异构系统间的计算服务处理。这里的 WSDL （ Web Service Description Language ）web服务描述语言也是一个 XML 文档，它通过 HTTP 向公众发布，公告客户端程序关于某个具体的 Web Service 服务的 URL 信息、方法的命名、 参数、 返回值等。

### SOAP

> SOAP 指简单对象访问协议，它是一种基于 XML 的消息通讯格式，用于网络上，不同平台，不同语言的应用程序间的通讯。可自定义，易于扩展。一条 SOAP 消息就是一个普通的 XML 文档，包含下列元素：
>
> - Envelope 元素，标识 XML 文档一条 SOAP 消息
> - Header 元素，包含头部信息的 XML 标签
> - Body 元素，包含所有的调用和响应的主体信息的标签
> - Fault 元素，错误信息标签。
>
> 以上的元素都在 SOAP 的命名空间 [http://www.w3.org/2001/12/soap-envelope](http://www.w3.org/2001/12/soap-envelope) 中声明。

- SOAP 的语法规则
  - SOAP 消息必须用 XML 来编码
  - SOAP 消息必须使用 SOAP Envelope 命名空间
  - SOAP 消息必须使用 SOAP Encoding 命名空间
  - SOAP 消息不能包含 DTD 引用
  - SOAP 消息不能包含 XML 处理指令

- SOAP 消息的基本结构
  ```xml
  <? xml version="1.0"?>
  <soap:Envelope
  xmlns:soap="http://www.w3.org/2001/12/soap-envelope"
  soap:encodingStyle="http://www.w3.org/2001/12/soap-encoding">
  <soap:Header>
  ...
  ...
  </soap:Header>
  <soap:Body>
  ...
  ...
  <soap:Fault>
    ...
    ...
  </soap:Fault>
  </soap:Body>
  </soap:Envelope>
  ```
- SOAP Envelope元素
  Envelope元素是 SOAP 消息的根元素。它指明 XML 文档是一个 SOAP 消息。它的属性 xmlns:soap 的值必须是 [http://www.w3.org/2001/12/soap-envelope](http://www.w3.org/2001/12/soap-envelope)。
  encodingStyle 属性，语法：soap:encodingStyle="URI"。
  encodingStyle 属性用于定义文档中使用的数据类型。此属性可出现在任何 SOAP 元素中，并会被应用到元素的内容及元素的所有子元素上。
- SOAP Header 元素
  actor 属性，语法 soap:actor="URI"
  通过沿着消息路径经过不同的端点， SOAP 消息可从某个发送者传播到某个接收者。并非 SOAP 消息的所有部分均打算传送到 SOAP 消息的最终端点。不过，另一个方面，也许打算传送给消息路径上的一个或多个端点。 SOAP 的 actor 属性可被用于将会Header 元素寻址到一个特定的端点。
  mustUnderstand 属性，语法 soap:mustUnderstand="0|1"
  SOAP 的 mustUnderstand 属性可用标识标题项对于要对其进行处理的接收者来说是强制的还是可选的。假如您向Header 元素的某个子元素添加了“mustUnderstand=1“，则要求处理头部的接收者必须认可此元素。
- SOAP Body 元素
  必需的 SOAP Body 元素可包含打算传送到消息最终端点的实际 SOAP 消息。Body 元素中既可以包含 SOAP 定义的命名空间中的元素，如 Fault，也可以是用户的应用程序自定义的元素。
- SOAP Fault 元素
  Fault 元素表示 SOAP 的错误消息。它必须是 Body 元素的子元素，且在一条 SOAP 消息中， Fault 元素只能出现一次。Fault 元素拥有下列子元素：
  - `<faultcode>`供识别故障的代码
  - `<faultstring>`可供人阅读的有关故障的说明
  - `<faultactor>`有关是谁引发故障的信息
  - `<detail>`存留涉及 Body 元素的应用程序专用错误信息
    常用的 SOAP Fault Codes
  - VersionMismatch SOAP Envelope 元素的无效命名空间
  - MustUnderstand Header 元素的一个直接子元素（带有设置为”1“的mustUnderstand 属性）无法被理解
  - Client 消息被不正确的构成，或包含了不正确的信息
  - Server 服务器有问题，因此无法处理进行下去
- HTTP 协议中的 SOAP 实例
  - SOAP 请求：(注意 HTTP 的 Head 属性)
    ```xml
    POST /InStock HTTP/1.1
    Host: www.jsoso.net
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: XXX

    <?xml version="1.0"?>
    <soap:Envelope
    xmlns:soap="http://www.w3.org/2001/12/soap-envelope"
    soap:encodingStyle="http://www.w3.org/2001/12/soap-encoding">
      <soap:Body xmlns:m="http://www.jsoso.net/stock">
        <m:GetStockPrice>
          <m:StockName>IBM</m:StockName>
        </m:GetStockPrice>
      </soap:Body>  
    </soap:Envelope>
    ```

  - SOAP 响应：(注意 HTTP 的 Head 属性)
    ```xml
    HTTP/1.1 200 OK
    Content-Type: application/soap+xml; charset=utf-8
    Content-Length: XXX

    <? xml version="1.0"?>
    <soap:Envelope
    xmlns:soap="http://www.w3.org/2001/12/soap-envelope"
    soap:encodingStyle="http://www.w3.org/2001/12/soap-encoding">
      <soap:Body xmlns:m="http://www.jsoso.net/stock">
        <m:GetStockPriceResponse>
          <m:Price>34.5</m:Price>
        </m:GetStockPriceResponse>
      </soap:Body>  
    </soap:Envelope>
    ```

### WSDL
主要元素
- `<types>` web service 使用的数据类型，它是独立与机器和语言的类型定义，这些数据类型定义被`<message>`标签所使用。
- `<message>` web service 使用的消息，它定义了Web Service函数的参数。在WSDL中，输入参数与输出参数要分开定义，使用不同的`<message>`标签体标识。`<message>`标签定义的输出输入参数被`<portType>`标签使用。
- `<portType>` web service 执行的操作。该标签引用`<message>`标签的定义来描述函数签名（操作名、输入参数、输出参数）。
- `<binding>` web service 使用的通信协议。`<portType>`标签中定义的每一操作在此绑定实现。
- `<service>` 该标签确定每一`<binding>`标签的端口地址。
  WSDL 文档可分为两个部分。
- 顶部分由抽象定义组成。抽象部分以独立于平台和语言的方式定义 SOAP 消息，它们并不包含任何随机器或语言而变的元素。这就定义了一系列服务，截然不同的应用都可以实现。`<types>`、`<message>`、`<portType>`属于抽象定义层。所有的抽象可以是单独存在于别的文件中，也可以从主文档中导入。
- 底部分由具体描述组成。如数据的序列化则可以归入底部分，因为它包含具体的定义。`<binding>`、`<service>`属于具体定义层。