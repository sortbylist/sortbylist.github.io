---
title: Rabbitmq入门
date: 2018-09-15 13:42:15
tags: [Rabbitmq]
---

#### AMQP

- AMQP ( Advance Message Queuing Protocal ) ，高级消息队列协议。是应用层协议的一个开放标准，为面向消息的中间件设计。消息中间件主要用于组件之间的解耦。
- AMQP的主要特征是面向消息、队列、路由（ 包括点对点和发布/订阅 ）、可靠性、安全。
- RabbitMQ是一个开源的 AMQP 实现。服务端用 Erlang 语言编写，支持多种客户端，如`Python`、`Java`、`C`等。用于在分布式系统中存储转发消息。

### RabbitMQ

- 原理图

  ![rabbitmq](/images/rabbitmq.png)


- 概念

  - server ( broker ) 服务器端

    接受客户端连接，实现 AMQP 消息队列和路由功能的进程

  - producer 生产者

    用来创建消息，然后发消息到代理服务器 ( RabbitMQ )。消息包括有效载荷和标签。

  - consumer 消费者

  - Virtual Host

    这是一个虚拟概念，类似于权限控制组，一个Vitual Host里面可以有若干个Exchange和Queue，但是权限控制的最小粒度是Vitual Host。

  - Exchange 交换机/交易所

    所有消息都是发送给它的.由它调度给相应队列。

    接收生产者发送的消息，并根据Binding规则将消息路由给服务器中的队列。ExchangeType决定了Exchange路由消息的行为。在RabbitMQ中，ExchangeType有`direct`、`Fanout`和`Topic`三种。

  - Queue 队列

    RabbitMQ内部的消息缓冲器，用于存储还未被消费者消费的消息。

  - Message 消息

    由`Header`和`Body`组成，`Header`是由生产者添加的各种属性的集合，包括Message是否被持久化、由哪个Message Queue接受、优先级是多少等。`Body`是真正传输的数据。

  - Binding 绑定规则

    将Exchange和Queue绑定起来，通过Routing Key (路由键)。

  - Routing Key 路由关键字

    exchange根据这个关键字进行消息投递。

  - Connection 连接

    一个连接可以创建多个通道.多个通道共享该连接的通道线程池。

  - Channel 通道

    一个通道通常对应一个生产者或一个消费者，否则可能出现bug,通道可以创建若干消费者，所有被创建的消费者共享connection的消费者线程池。

  - 所以，对于消息的过滤，我们首先是指定交易所，然后指定队列和路由。如果需要进一步细分，就可以使用主题。

- 使用过程

  - **消费端**
      - 消费端连接到消息队列服务器，此时建立一个Connection (连接)
      - 打开一个Channel
      - 消费端声明一个Exchange，并设置相关属性
      - 消费端声明一个Queue，并设置相关属性
      - 消费端使用Routing Key，在Exchange和Queue之间建立绑定关系
  - **生产端**
      - 生产端连接到消息队列服务器，此时建立一个Connection(连接)或使用消费端的Connection
      - 打开一个Channel
      - 生产端声明一个Exchange，并设置相关属性
      - 生产端使用Routing Key投递消息到Exchange
      - Exchange接收到消息后，根据消息的Routing key和已经设置的binding，进行消息路由（ExchangeType起作用），将消息投递到一个或多个队列中


