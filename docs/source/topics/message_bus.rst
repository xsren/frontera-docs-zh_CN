===========
消息总线
===========

消息总线是传输层抽象机制。Frontera 提供了接口和几个实现。同一时间只能使用一种类型的消息总线，并通过 :setting:`MESSAGE_BUS` 设置。

爬虫进程可以使用

.. autoclass:: frontera.contrib.backends.remote.messagebus.MessageBusBackend

和消息总线进行通信。


内置消息总线参考
==============================

ZeroMQ
------
这是默认选项，使用轻量级的 `ZeroMQ`_ 库实现

.. autoclass:: frontera.contrib.messagebus.zeromq.MessageBus

可以使用 :ref:`zeromq-settings` 配置。

ZeroMQ 需要按照 ZeroMQ 库，并且启动broker进程，请参考 :ref:`running_zeromq_broker` 。

总的来说，使用 ZeroMQ 消息总线是为了用最少的部署实现 PoC （Patch Output Converter 成批输出转换程序）。因为它很容易
在组件的数据流未正确调整或启动过程中发生消息丢失，所以请参照下面的顺序启动组件：

#. :term:`db worker`
#. :term:`strategy worker`
#. :term:`spiders`

不幸的是，停止执行未完成抓取的爬虫时，无法避免消息丢失。如果你的爬虫程序对少量的信息丢失敏感的话，我建议你使用 Kafka。

.. pull-quote::
    警告！ZeroMQ消息总线不支持多个 SW worker 和 DB worker， 每种 woker 只能有一个实例。

Kafka
-----
使用这个类

.. autoclass:: frontera.contrib.messagebus.kafkabus.MessageBus

使用 :ref:`kafka-settings` 配置。

需要运行 `Kafka`_ 服务，这个服务更适合大规模采集。

.. _Kafka: http://kafka.apache.org/
.. _ZeroMQ: http://zeromq.org/


.. _message_bus_protocol:

协议
========

根据数据流，Frontera 使用几种消息类型来编码它的消息。每种消息是用 `msgpack`_ 或 JSON 序列化的 python 对象。可以使用 :setting:`MESSAGE_BUS_CODEC` 选择编解码器模块，并且需要导出编码器和解码器类。

以下是子类实现自己的编解码器所需的类:

.. autoclass:: frontera.core.codec.BaseEncoder

    .. automethod:: frontera.core.codec.BaseEncoder.encode_add_seeds
    .. automethod:: frontera.core.codec.BaseEncoder.encode_page_crawled
    .. automethod:: frontera.core.codec.BaseEncoder.encode_request_error
    .. automethod:: frontera.core.codec.BaseEncoder.encode_request
    .. automethod:: frontera.core.codec.BaseEncoder.encode_update_score
    .. automethod:: frontera.core.codec.BaseEncoder.encode_new_job_id
    .. automethod:: frontera.core.codec.BaseEncoder.encode_offset

.. autoclass:: frontera.core.codec.BaseDecoder

    .. automethod:: frontera.core.codec.BaseDecoder.decode
    .. automethod:: frontera.core.codec.BaseDecoder.decode_request


可用的编解码器
================

MsgPack
-------
.. automodule:: frontera.contrib.backends.remote.codecs.msgpack

Module: frontera.contrib.backends.remote.codecs.msgpack

JSON
----
.. automodule:: frontera.contrib.backends.remote.codecs.json

Module: frontera.contrib.backends.remote.codecs.json


.. _msgpack: http://msgpack.org/index.html