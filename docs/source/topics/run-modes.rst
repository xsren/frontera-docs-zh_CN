=========
运行模式
=========

下图展示了运行模式的架构图：

.. image:: _images/high-level-arc.png


====================  =========================================================================  ======================================================  =====================
模式                  父类                                                               所需组件                                       可用的后端
====================  =========================================================================  ======================================================  =====================
单进程        :class:`Backend <frontera.core.components.Backend>`                        单进程运行爬虫                      内存, SQLAlchemy
分布式爬虫  :class:`Backend <frontera.core.components.Backend>`                        多个爬虫和单个 :term:`db worker`                    内存, SQLAlchemy
分布式后端  :class:`DistributedBackend <frontera.core.components.DistributedBackend>`  多个爬虫, 多个 :term:`strategy worker` (s) 和多个 db worker(s).  SQLAlchemy, HBase
====================  =========================================================================  ======================================================  =====================


单进程
==============

Frontera 与 fetcher 在相同的过程中实例化（例如在 Scrapy 中）。要实现这个，需要设置 :setting:`BACKEND` 为 :class:`Backend <frontera.core.components.Backend>` 的子类。这种模式适合那种少量文档并且时间要求不紧的应用。

分布式爬虫
===================

爬虫是分布式的，但后端不是。后端运行在 :term:`db worker` 中，并通过 :term:`message bus` 与爬虫通信。

1. 将爬虫进程中的 :setting:`BACKEND` 设置为 :class:`MessageBusBackend <frontera.contrib.backends.remote.messgebus.MessageBusBackend>`
2. 在 DB worker 中 :setting:`BACKEND` 应该指向 :class:`Backend <frontera.core.components.Backend>` 的子类。
3. 每个爬虫进程应该有它自己的 :setting:`SPIDER_PARTITION_ID`，值为从0到 :setting:`SPIDER_FEED_PARTITIONS`。
4. 爬虫和 DB worker 都应该将 :setting:`MESSAGE_BUS` 设置为你选择的消息总线类或者其他你自定义的实现。

此模式适用于需要快速获取文档，同时文档的数量相对较小的应用。


分布式爬虫和后端
===============================

爬虫和后端都是分布式的。后端分成了两部分： :term:`strategy worker` 和 :term:`db worker`。strategy worker 实例被分配给他们自己的 :term:`spider log` 部分。

1. 将爬虫进程中的 :setting:`BACKEND` 设置为 :class:`MessageBusBackend <frontera.contrib.backends.remote.messgebus.MessageBusBackend>`
2. DB workers 和 SW workers 的 :setting:`BACKEND` 应该指向 :class:`DistributedBackend <frontera.core.components.DistributedBackend>` 的子类。同时还需要配置您选择的后端。
3. 每个爬虫进程应该有它自己的 :setting:`SPIDER_PARTITION_ID`，值为从0到 :setting:`SPIDER_FEED_PARTITIONS`。最后一个必须可以被所有 DB worker 实例访问。
4. 每个 SW worker 应该有自己的 :setting:`SCORING_PARTITION_ID`，值为从0到 :setting:`SPIDER_LOG_PARTITIONS`。最后一个必须可以被所有 SW worker 实例访问。
5. 爬虫和所有的 worker 都应该将 :setting:`MESSAGE_BUS` 设置为你选择的消息总线类或者其他你自定义的实现。

在这种模式下，只有 Kafka 消息总线、SqlAlchemy 和 Habse 后端是默认支持的。

此模式适用于广度优先抓取和网页数量巨大的情况。
