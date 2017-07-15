---
title: librdkafka源码分析
date: 2017-04-16 18:29:20
categories: [笔记]
tags: [CPP, 源码分析]
---

# 介绍
[librdkafka](https://github.com/edenhill/librdkafka)是Apache Kafka的一个C/C++客户端。里面包含了一个C的客户端和一个封装C的C++客户端。两者总的代码行数超过3万行。算起来也是不小。

我准备从以下几个方面入手，简单分析下C客户端代码的整体结构与线程模型。

- 线程模型
- 几种重要的队列（queue）
- 主要分析Consumer的执行流

未涉及的包括：

- Producer的执行流
- 数据交互的格式以及打包与解包
- Client group

大部分代码都是以C编写的，从我阅读后来看，整体的代码编写风格很随意，没有固定的代码风格。这可能也会在阅读过程中增加一点难度，第二是C风格的代码免不了宏，因此在分析的时候需要逐一分析，这样代码的上下文很容易丢失。

阅读的代码为windows平台。


# 线程模型
有以下几个线程：

## 主处理线程
当调用`rd_kafka_new`时，会传入创建`rd_kafka_t`对象的类型，也就是`RD_KAFKA_CONSUMER`或者`RD_KAFKA_PRODUCER`。之后会调用`rd_kafka_cgrp_new`创建`rd_kafka_cgrp_t`对象，接着就是创建主处理线程，线程的入口为`rd_kafka_thread_main`。

`rd_kafka_thread_main`中会从`rd_kafka_t`对象中的操作队列`rk_ops`中逐一取出进行操作。操作的数据结构类型为`rd_kafka_op_t`。操作类型有很多种，参考`rd_kafka_op_type_t`。操作执行完后调用`rd_kafka_op_handle`回调。

这是`rd_kafka_thread_main`线程的主要工作，其中有一些细节这里未涉及。

## Broker线程
上述`rd_kafka_thread_main`线程创建完后，调用`rd_kafka_broker_add`创建internal broker线程。broker线程的类型有三种，分别是：

- RD_KAFKA_CONFIGURED
  
  根据用户配置，生成的broker线程
- RD_KAFKA_LEARNED
  
  内部使用的broker线程，主要针对Client Group使用
- RD_KAFKA_INTERNAL
  
  内部使用的broker线程

Broker线程主要执行的是针对Broker的当前内部状态，比如INIT、DOWN、CONNECT、UP等，在这些状态下，执行针对的操作。比如在`RD_KAFKA_BROKER_STATE_UP`状态下，根据`rk_type`（类型为`rd_kafka_type_t`）来执行`rd_kafka_broker_producer_serve`或者`rd_kafka_broker_consumer_serve`。

Broker线程可能会有多个。


# 队列
队列在librdkafka的数据流转中起到了关键性的作用。主要有：
```cpp
rd_kafka_q_t *rktp_ops;
rd_kafka_q_t *rktp_fetchq;
rd_kafka_q_t *rkcg_q;
rd_kafka_q_t *rk_rep;
rd_kafka_q_t *rkcg_q;
```

buf队列有以下几个：
```cpp
rd_kafka_bufq_t rkb_outbufs;
rd_kafka_bufq_t rkb_waitresps;
rd_kafka_bufq_t rkb_retrybufs;
```

在创建`rd_kafka_cgrp_t`对象时，将`rkcg_q`设置为`rk_rep`的转发队列（forward queue）
```cpp
rd_kafka_q_fwd_set(rk->rk_rep, rkcg->rkcg_q);
```

在线程`rd_kafka_broker_thread_main`里，调用`rd_kafka_toppar_op_fetch_start`设置`rktp_fetchq`的转发queue为`rkcg_q`。

在Broker线程中，rd_kafka_fetch_reply_handle会创建一个临时的queue，然后创建一个rko，将rko压入到临时队列的rkq_q对象中。
```cpp
rd_kafka_q_enq(rkq, rko)
```

最后将这个临时queue压入到`rd_kafka_toppar_s::rktp_fetchq`的转发queue中，也就是rkcg_q。

Broker 线程会调用`rd_kafka_broker_consumer_serve`，然后到`rd_kafka_broker_fetch_toppars`构建一个`rd_kafka_buf_t *`对象，同时将此对象的`rkbuf_cb`设置为`rd_kafka_broker_fetch_reply`。然后将buf压入到broker的`rkb_outbufs`队列中。

在`rd_kafka_send`会从`rkb_outbufs`获取buf发送之后，会将此buf压入到`rkb_waitresps`队列中。

收到回应后，会调用`rkbuf_cb`会被调用`rd_kafka_buf_callback`的request，就是rkbuf。`rkb->rkb_waitresps`的`rkbq_bufs`中，通过corrid查找相应的`rd_kafka_buf_t`对象找到后更新状态。

```cpp
rd_kafka_bufq_deq(&rkb->rkb_waitresps, rkbuf);
```

# Consumer执行流
消费者主动去poll消息

```cpp
rd_kafka_consume0(rk, rkcg->rkcg_q, timeout_ms);
```
`rkcg_q`就是`rk_rep`的转发queue。从`rkcg_q`取出一个`rd_kafka_op_t`对象，再从`rd_kafka_op_t`对象中取得消息（`rd_kafka_message_t`）。


# 总结
这个库的代码结构相对来说还是比较复杂的。上面的分析也很粗糙，有很多细节没有说明。但是大方向有了，其他的细节相分析起来，就会游刃有余，而不是毫无头绪。
这个库有一点不好的是队列的转发操作，说白了就是互相保存对象地址，然后在其他地方操作，所以不是很直观。在没弄清楚的情况会让人摸不着头脑。
一般我自己在阅读源码时，会遵循以下几个步骤：

- 有一个可运行的示例
- 理解线程模型
- 大致阅读相关数据结构
- 将示例的执行流串起来
- 抓包查看如何通讯

然后与以往看过的源码进行一个对比。从代码风格，代码格式化，代码的数据结构，API等方面进行比较。