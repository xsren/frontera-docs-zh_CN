========
Settings
========

Frontera settings 允许你定制所有的组件，包括
:class:`FrontierManager <frontera.core.manager.FrontierManager>`,
:class:`Middleware <frontera.core.components.Middleware>` and
:class:`Backend <frontera.core.components.Backend>` themselves.

settings 提供了键值映射的全局命名空间可以用来获取配置值。可以通过不同的机制填充设置，如下所述。

内置的配置请查看: :ref:`Built-in settings reference <frontier-built-in-frontier-settings>`。

设计 settings
========================

当你使用 Frontera 的时候，你需要告诉它你在用什么配置。因为 :class:`FrontierManager <frontera.core.manager.FrontierManager>` 是 Frontier 使用的入口，你可以使用 :ref:`Loading from settings <frontier-loading-from-settings>` 来进行配置。

当使用一个字符串路径指向配置文件的时候，我们建议下面这种文件结构::

    my_project/
        frontier/
            __init__.py
            settings.py
            middlewares.py
            backends.py
        ...

这些都是基本的:

- ``frontier/settings.py``: frontier settings 文件。
- ``frontier/middlewares.py``: frontier 使用的中间件。
- ``frontier/backends.py``: frontier 使用的后端。


如何访问 settings
======================
:class:`Settings <frontera.settings.Settings>` 可以通过
:attr:`FrontierManager.settings <frontera.core.manager.FrontierManager.settings>` 属性访问, 它传给了
:attr:`Middleware.from_manager <frontera.core.components.Middleware.from_manager>` 和
:attr:`Backend.from_manager <frontera.core.components.Backend.from_manager>` 类方法::

    class MyMiddleware(Component):

        @classmethod
        def from_manager(cls, manager):
            manager = crawler.settings
            if settings.TEST_MODE:
                print "test mode is enabled!"

换句话说， settings 可以通过 :class:`Settings <frontera.settings.Settings>` 访问.

Settings 类
==============

.. class:: frontera.settings.Settings

.. _frontier-built-in-frontier-settings:

内置 frontier settings
==========================

下面是所以可用的 Frontera settings，以字母顺序排序，以及它们的默认值和适用范围。

.. setting:: AUTO_START

AUTO_START
----------

默认: ``True``

是否允许 frontier 自动启动。参考 :ref:`Starting/Stopping the frontier <frontier-start-stop>`

.. setting:: BACKEND

BACKEND
-------

默认: ``'frontera.contrib.backends.memory.FIFO'``

:class:`Backend <frontera.core.components.Backend>` 使用的后端。详情参考
:ref:`Activating a backend <frontier-activating-backend>`。


.. setting:: BC_MIN_REQUESTS

BC_MIN_REQUESTS
---------------

默认: ``64``

广泛的抓取队列的获取操作将继续重试，直到收集到指定的请求数。最大重试次数硬编码为3。

.. setting:: BC_MIN_HOSTS

BC_MIN_HOSTS
------------

默认: ``24``

持续从队列中获取请求任务，直到请求的 host 种类到达最小值。最大重试次数硬编码为3。

.. setting:: BC_MAX_REQUESTS_PER_HOST

BC_MAX_REQUESTS_PER_HOST
------------------------

默认:: ``128``

如果某个 host 的请求数已经到达单 host 请求的上限，将不包括这些请求。这是针对广泛抓取获取任务数量的建议。

.. setting:: CANONICAL_SOLVER

CANONICAL_SOLVER
----------------

默认: ``frontera.contrib.canonicalsolvers.Basic``

:class:`CanonicalSolver <frontera.core.components.CanonicalSolver>` 被用来解析规划网址。详情参考 :ref:`Canonical URL Solver <canonical-url-solver>`.

.. setting:: SPIDER_LOG_CONSUMER_BATCH_SIZE

SPIDER_LOG_CONSUMER_BATCH_SIZE
------------------------------

默认: ``512``

这是 strategy worker 和 db worker 消费 spider log 流时的批量大小。增加它将使 worker 在每个任务上花更多的时间，但是每个任务处理更多的 item，因此在一段时间内给其它任务留下了更少的时间。减少它将导致在同一时间段内运行多个任务，但总体效率较低。 当你的消费者太快或者太慢的时候使用这个选项。

.. setting:: SCORING_LOG_CONSUMER_BATCH_SIZE

SCORING_LOG_CONSUMER_BATCH_SIZE
-------------------------------

默认: ``512``

这是 db worker 消费 scoring log 流时的批量大小。当你需要调整 scoring log 消费速度的时候使用这个选项。

.. setting:: CRAWLING_STRATEGY

CRAWLING_STRATEGY
-----------------

默认: ``None``

抓取策略类的路径，在 :term:`strategy worker` 中实例化并使用，用来在分布式模式中设置抓取优先级和停止抓取。

.. setting:: DELAY_ON_EMPTY

DELAY_ON_EMPTY
--------------

默认: ``5.0``

当队列大小小于 ``CONCURRENT_REQUESTS`` 时，向调度器发送请求时的延迟时间。当后端没有请求的时候，这个延迟帮助消耗掉剩余的缓存而无需每次请求都去请求后端。如果对后端的调用花费的时间过长，则增加它，如果你需要更快从种子启动爬虫，则减少它。

.. setting:: KAFKA_GET_TIMEOUT

KAFKA_GET_TIMEOUT
-----------------

默认: ``5.0``

从 Kafka 中获取数据的超时时间。

.. setting:: LOGGING_CONFIG

LOGGING_CONFIG
--------------

默认: ``logging.conf``

logging 模块的路径。参考 https://docs.python.org/2/library/logging.config.html#logging-config-fileformat 如果文件不存在，日志将使用 ``logging.basicConfig()`` 实例化，将在 CONSOLE 输出日志。这个选项只在 :term:`db worker` 和 :term:`strategy worker` 中使用。
The path to a file with logging module configuration.

.. setting:: MAX_NEXT_REQUESTS

MAX_NEXT_REQUESTS
-----------------

默认: ``64``

:attr:`get_next_requests <frontera.core.manager.FrontierManager.get_next_requests>` API 返回的最大请求数量。在分布式模式中，它应该是 :term:`db worker` 为每个爬虫生成的请求数或者是每次从消息总线中获取的请求数。在单进程模式下，它应该是每次调用 ``get_next_requests`` 获取的请求数量。

.. setting:: MAX_REQUESTS

MAX_REQUESTS
------------

默认: ``0``

Frontera完成之后返回的最大请求数总量。如果是0（默认值），frontier 将一直运行。详情参考： :ref:`Finishing the frontier <frontier-finish>`.

.. setting:: MESSAGE_BUS

MESSAGE_BUS
-----------

默认: ``frontera.contrib.messagebus.zeromq.MessageBus``

指向 :term:`message bus` 的实现。默认是 ZeroMQ。

.. setting:: MESSAGE_BUS_CODEC

MESSAGE_BUS_CODEC
-----------------

默认: ``frontera.contrib.backends.remote.codecs.msgpack``

指向 :term:`message bus` 编码实现。参考 :ref:`codec interface description <message_bus_protocol>`。
默认是 MsgPack。

.. setting:: MIDDLEWARES

MIDDLEWARES
-----------

包含在 frontier 中启用的中间件的列表。详情参考 :ref:`Activating a middleware <frontier-activating-middleware>`

默认::

    [
        'frontera.contrib.middlewares.fingerprint.UrlFingerprintMiddleware',
    ]

.. setting:: NEW_BATCH_DELAY

NEW_BATCH_DELAY
---------------

默认: ``30.0``

在 DB worker 中使用，它是为所有分区产生任务集合的时间间隔。如果分区很忙，它将被忽略掉。

.. setting:: OVERUSED_SLOT_FACTOR

OVERUSED_SLOT_FACTOR
--------------------

默认: ``5.0``

（某个 slot 中进行中+队列中的请求）/ （每个slot允许的最大并发送）称作 overused。它只影响 Scrapy scheduler。

.. setting:: REQUEST_MODEL

REQUEST_MODEL
-------------

默认: ``'frontera.core.models.Request'``

frontier 使用的 :class:`Request <frontera.core.models.Request>` 模型。

.. setting:: RESPONSE_MODEL

RESPONSE_MODEL
--------------

默认: ``'frontera.core.models.Response'``

frontier 使用的 :class:`Response <frontera.core.models.Response>` 模型。

.. setting:: SCORING_PARTITION_ID

SCORING_PARTITION_ID
--------------------

默认: ``0``

被 strategy worker 使用，代表 strategy worker 被分配的分区。

.. setting:: SPIDER_LOG_PARTITIONS

SPIDER_LOG_PARTITIONS
---------------------

默认: ``1``

:term:`spider log` 分区的数量。这个参数影响 :term:`strategy worker` (s) 的数量，每个 strategy worker 被分配到自己的分区。

.. setting:: SPIDER_FEED_PARTITIONS

SPIDER_FEED_PARTITIONS
----------------------

默认: ``1``

:term:`spider feed` 分区的数量。这个参数影响爬虫进程数量。每个爬虫被分配到了自己的分区。

.. setting:: SPIDER_PARTITION_ID

SPIDER_PARTITION_ID
-------------------

默认: ``0``

每个爬虫的配置，将爬虫指向分配给自己的分区。

.. setting:: STATE_CACHE_SIZE

STATE_CACHE_SIZE
----------------

默认: ``1000000``

在被清除之前状态缓存的最大数量。

.. setting:: STORE_CONTENT

STORE_CONTENT
-------------

默认: ``False``

Determines if content should be sent over the message bus and stored in the backend: a serious performance killer.

.. setting:: TEST_MODE

TEST_MODE
---------

默认: ``False``

是否开启 frontier 的测试模式。 参考 :ref:`Frontier test mode <frontier-test-mode>`



内置指纹中间件设置
========================================

设置被 :ref:`UrlFingerprintMiddleware <frontier-url-fingerprint-middleware>` 和 :ref:`DomainFingerprintMiddleware <frontier-domain-fingerprint-middleware>` 使用。

.. _frontier-default-settings:

.. setting:: URL_FINGERPRINT_FUNCTION

URL_FINGERPRINT_FUNCTION
------------------------

默认: ``frontera.utils.fingerprint.sha1``

用来计算 ``url`` 指纹的函数。


.. setting:: DOMAIN_FINGERPRINT_FUNCTION

DOMAIN_FINGERPRINT_FUNCTION
---------------------------

默认: ``frontera.utils.fingerprint.sha1``

用来计算 ``domain`` 指纹的函数。


.. setting:: TLDEXTRACT_DOMAIN_INFO

TLDEXTRACT_DOMAIN_INFO
----------------------

默认: ``False``

如果设置为 ``True`` ，将使用 `tldextract`_ 附加额外的域信息（二级，顶级和子域名）到 meta 字段（参考 :ref:`frontier-objects-additional-data` ）。

.. _tldextract: https://pypi.python.org/pypi/tldextract


内置后端配置
==========================

SQLAlchemy
----------

.. setting:: SQLALCHEMYBACKEND_CACHE_SIZE

SQLALCHEMYBACKEND_CACHE_SIZE
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

默认: ``10000``

SQLAlchemy 元数据的 LRU 缓存大小。它用来缓存对象，这些对象从数据库获得，还缓存抓取的文档。它主要节约了数据库的吞吐量，如果你面临从数据库获得太多数据的问题增加它，如果你想节约内存就减少它。

.. setting:: SQLALCHEMYBACKEND_CLEAR_CONTENT

SQLALCHEMYBACKEND_CLEAR_CONTENT
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

默认: ``True``

如果你想禁止每次后端初始化的时候清理表数据则将之设为 ``False`` （例如每次启动 Scrapy 爬虫的时候）。


.. setting:: SQLALCHEMYBACKEND_DROP_ALL_TABLES

SQLALCHEMYBACKEND_DROP_ALL_TABLES
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

默认: ``True``

如果你想禁止每次后端初始化的时候删除数据库表则将之设为 ``False`` （例如每次启动 Scrapy 爬虫的时候）。

.. setting:: SQLALCHEMYBACKEND_ENGINE

SQLALCHEMYBACKEND_ENGINE
^^^^^^^^^^^^^^^^^^^^^^^^

默认:: ``sqlite:///:memory:``

SQLAlchemy 数据库 URL。 默认使用内存。

.. setting:: SQLALCHEMYBACKEND_ENGINE_ECHO

SQLALCHEMYBACKEND_ENGINE_ECHO
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

默认: ``False``

打开/关闭 SQLAlchemy 的冗余输出。调试 SQL 语句的时候有用。

.. setting:: SQLALCHEMYBACKEND_MODELS

SQLALCHEMYBACKEND_MODELS
^^^^^^^^^^^^^^^^^^^^^^^^

默认::

    {
        'MetadataModel': 'frontera.contrib.backends.sqlalchemy.models.MetadataModel',
        'StateModel': 'frontera.contrib.backends.sqlalchemy.models.StateModel',
        'QueueModel': 'frontera.contrib.backends.sqlalchemy.models.QueueModel'
    }

用来设置后端使用的 SQLAlchemy 模型。主要用来定制化。


重新抓取后端
------------------

.. setting:: SQLALCHEMYBACKEND_REVISIT_INTERVAL

SQLALCHEMYBACKEND_REVISIT_INTERVAL
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

默认: ``timedelta(days=1)``

重新访问网页的时间，使用 ``datetime.timedelta` 表示。它会影响网页的定期抓取。之前抓取的网页还是使用旧的时间间隔。


.. _hbase-settings:

HBase 后端
-------------

.. setting:: HBASE_BATCH_SIZE

HBASE_BATCH_SIZE
^^^^^^^^^^^^^^^^

默认: ``9216``

在发送到HBase之前累计的PUT操作的数量。

.. setting:: HBASE_DROP_ALL_TABLES

HBASE_DROP_ALL_TABLES
^^^^^^^^^^^^^^^^^^^^^

默认: ``False``

允许在 worker 启动的时候删除并重建 Hbase 表格。

.. setting:: HBASE_METADATA_TABLE

HBASE_METADATA_TABLE
^^^^^^^^^^^^^^^^^^^^

默认: ``metadata``

网页元数据表格的名字。

.. setting:: HBASE_NAMESPACE

HBASE_NAMESPACE
^^^^^^^^^^^^^^^

默认: ``crawler``

所有爬虫程序相关表在 HBase 命名空间的名称。

.. setting:: HBASE_QUEUE_TABLE

HBASE_QUEUE_TABLE
^^^^^^^^^^^^^^^^^

默认: ``queue``

Hbase 中队列优先级表。

.. setting:: HBASE_STATE_CACHE_SIZE_LIMIT

HBASE_STATE_CACHE_SIZE_LIMIT
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

默认: ``3000000``

在写入 HBase 和清除之前 :term:`strategy worker` 中 :term:`state cache` 的最多数量。

.. setting:: HBASE_THRIFT_HOST

HBASE_THRIFT_HOST
^^^^^^^^^^^^^^^^^

默认: ``localhost``

HBase Thrift server 主机

.. setting:: HBASE_THRIFT_PORT

HBASE_THRIFT_PORT
^^^^^^^^^^^^^^^^^

默认: ``9090``

HBase Thrift server 端口

.. setting:: HBASE_USE_FRAMED_COMPACT

HBASE_USE_FRAMED_COMPACT
^^^^^^^^^^^^^^^^^^^^^^^^

默认: ``False``

启用此选项可大大降低传输开销，但是服务器需要正确配置才能使用节俭框架运输和紧凑协议。

.. setting:: HBASE_USE_SNAPPY

HBASE_USE_SNAPPY
^^^^^^^^^^^^^^^^

默认: ``False``

是否在 Hbase 中使用 snappy 压缩内容和元数据。在HBase中减少磁盘和网络IO的数量，降低响应时间。 HBase必须正确配置以支持Snappy压缩。

.. _zeromq-settings:

ZeroMQ 消息总线设置
===========================

消息总线类是 ``distributed_frontera.messagebus.zeromq.MessageBus``

.. setting:: ZMQ_ADDRESS

ZMQ_ADDRESS
-----------

默认: ``127.0.0.1``

定义 ZeroMQ 套接字应该绑定或连接的位置。可以是主机名或IP地址。现在，ZMQ 只有通过 IPv4进行了测试。IPv6在不久的将来会增加支持。

.. setting:: ZMQ_BASE_PORT

ZMQ_BASE_PORT
-------------

默认: ``5550``

所有 ZeroMQ 的基本端口。 它使用6个 sockets ，且6个端口是顺序的.确保[base：base + 5]所表示的端口都是可用的。


.. _kafka-settings:

Kafka 消息总线配置
==========================

这个消息总线的类是 ``frontera.contrib.messagebus.kafkabus.MessageBus``

.. setting:: KAFKA_LOCATION

KAFKA_LOCATION
--------------

kafka 代理的主机名和端口号，以 :分割。可以是主机名:端口的字符串对，用逗号（,）分隔。

.. setting:: KAFKA_CODEC

KAFKA_CODEC
-----------

默认: ``None``

Kafka-python 1.0.x 版本使用的压缩方式, 它应该是 None 或者 ``snappy``, ``gzip`` 和
``lz4``中的一个。

.. setting:: SPIDER_LOG_DBW_GROUP

SPIDER_LOG_DBW_GROUP
--------------------

默认: ``dbw-spider-log``

Kafka 消费者群组名，被 :term:`db worker` 的 :term:`spider log` 使用。

.. setting:: SPIDER_LOG_SW_GROUP

SPIDER_LOG_SW_GROUP
-------------------

默认: ``sw-spider-log``

Kafka 消费者群组名，被 :term:`strategy worker` 的 :term:`spider log` 使用。

.. setting:: SCORING_LOG_DBW_GROUP

SCORING_LOG_DBW_GROUP
---------------------

默认: ``dbw-scoring-log``

Kafka 消费者群组名，被 :term:`db worker` 的 :term:`scoring log` 使用。

.. setting:: SPIDER_FEED_GROUP

SPIDER_FEED_GROUP
-----------------

默认: ``fetchers-spider-feed``

Kafka 消费者群组名，被 :term:`spider` 的 :term:`spider feed` 使用。

.. setting:: SPIDER_LOG_TOPIC

SPIDER_LOG_TOPIC
----------------

默认: ``frontier-done``

:term:`spider log` 数据流的 topic 名字。


.. setting:: SPIDER_FEED_TOPIC

SPIDER_FEED_TOPIC
-----------------

默认: ``frontier-todo``

:term:`spider feed` 数据流的 topic 名字。

.. setting:: SCORING_LOG_TOPIC

SCORING_LOG_TOPIC
-----------------

:term:`scoring log` 数据流的 topic 名字。


Default settings
================

如果没有指定设置，frontier 将使用内置的默认设置。有关默认值的完整列表，请参见 :ref:`Built-in settings reference <frontier-built-in-frontier-settings>` 。所有默认设置都可以被覆盖。


.. _`kafka-python documentation`: http://kafka-python.readthedocs.io/en/1.1.1/apidoc/KafkaProducer.html
