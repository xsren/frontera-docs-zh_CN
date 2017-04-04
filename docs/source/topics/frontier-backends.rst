========
后端
========

Frontier :class:`Backend <frontera.core.components.Backend>` 是抓取逻辑/策略所在的地方，本质上是你的爬虫的大脑。 :class:`Queue <frontera.core.components.Queue>`, :class:`Metadata <frontera.core.components.Metadata>` 和 :class:`States <frontera.core.components.States>`是为了放置低级代码的类，相反，后端类运行更高级的代码。Frontera 内置了内存或数据库方式实现的 Queue, Metadata 和 States，它们可以在你自定义的后端类中使用或者实例化 :class:`FrontierManager <frontera.core.manager.FrontierManager>` 和后端独立使用。

后端方法在 :class:`Middleware <frontera.core.components.Middleware>` 之后被 FrontierManager 调用，根据 :ref:`frontier data flow <frontier-data-flow>` 使用 hooks 处理 class:`Request <frontera.core.models.Request>` 和 :class:`Response <frontera.core.models.Response>` 。

与中间件可以激活许多不同的实例不同，每个 Frontera 只能使用一种后端。

.. _frontier-activating-backend:

激活一个后端
====================

要激活 Frontera 后端组件，请通过  :setting:`BACKEND` 设置进行设置。

这是一个例子 ::

    BACKEND = 'frontera.contrib.backends.memory.FIFO'

请记住，某些后端可能需要额外配置其他 settings。 更多信息，请参阅 :ref:`backends documentation <frontier-built-in-backend>` 。

.. _frontier-writing-backend:

写你自己的后端
========================

每个后端组件是一个 Python 类，它继承自 :class:`Backend <frontera.core.components.Backend>` 或
:class:`DistributedBackend <frontera.core.components.DistributedBackend>` ，且使用 :class:`Queue`, :class:`Metadata` 和 :class:`States` 中的一个或多个。

:class:`FrontierManager` 将通过下列方法与激活的后端通信。

.. class:: frontera.core.components.Backend

    **Methods**

    .. automethod:: frontera.core.components.Backend.frontier_start

        :return: None.

    .. automethod:: frontera.core.components.Backend.frontier_stop

        :return: None.

    .. automethod:: frontera.core.components.Backend.finished

    .. automethod:: frontera.core.components.Backend.add_seeds

        :return: None.

    .. automethod:: frontera.core.components.Backend.page_crawled

        :return: None.

    .. automethod:: frontera.core.components.Backend.request_error

        :return: None.

    .. automethod:: frontera.core.components.Backend.get_next_requests

    **Class Methods**

    .. automethod:: frontera.core.components.Backend.from_manager

    **Properties**

    .. attribute:: frontera.core.components.Backend.queue

    .. attribute:: frontera.core.components.Backend.states

    .. attribute:: frontera.core.components.Backend.metadata


.. class:: frontera.core.components.DistributedBackend

Inherits all methods of Backend, and has two more class methods, which are called during strategy and db worker
instantiation.

    .. automethod:: frontera.core.components.DistributedBackend.strategy_worker
    .. automethod:: frontera.core.components.DistributedBackend.db_worker

Backend should communicate with low-level storage by means of these classes:

Metadata
^^^^^^^^

.. class:: frontera.core.components.Metadata

    **Methods**

    .. automethod:: frontera.core.components.Metadata.add_seeds

    .. automethod:: frontera.core.components.Metadata.request_error

    .. automethod:: frontera.core.components.Metadata.page_crawled


Known implementations are: :class:`MemoryMetadata` and :class:`sqlalchemy.components.Metadata`.

Queue
^^^^^

.. class:: frontera.core.components.Queue

    **Methods**

    .. automethod:: frontera.core.components.Queue.get_next_requests

    .. automethod:: frontera.core.components.Queue.schedule

    .. automethod:: frontera.core.components.Queue.count

Known implementations are: :class:`MemoryQueue` and :class:`sqlalchemy.components.Queue`.

States
^^^^^^

.. class:: frontera.core.components.States

    **Methods**

    .. automethod:: frontera.core.components.States.update_cache

    .. automethod:: frontera.core.components.States.set_states

    .. automethod:: frontera.core.components.States.flush

    .. automethod:: frontera.core.components.States.fetch


Known implementations are: :class:`MemoryStates` and :class:`sqlalchemy.components.States`.


.. _frontier-built-in-backend:

Built-in backend reference
==========================

This article describes all backend components that come bundled with Frontera.

To know the default activated :class:`Backend <frontera.core.components.Backend>` check the
:setting:`BACKEND` setting.


.. _frontier-backends-basic-algorithms:

Basic algorithms
^^^^^^^^^^^^^^^^
Some of the built-in :class:`Backend <frontera.core.components.Backend>` objects implement basic algorithms as
as `FIFO`_/`LIFO`_ or `DFS`_/`BFS`_ for page visit ordering.

Differences between them will be on storage engine used. For instance,
:class:`memory.FIFO <frontera.contrib.backends.memory.FIFO>` and
:class:`sqlalchemy.FIFO <frontera.contrib.backends.sqlalchemy.FIFO>` will use the same logic but with different
storage engines.

All these backend variations are using the same :class:`CommonBackend <frontera.contrib.backends.CommonBackend>` class
implementing one-time visit crawling policy with priority queue.

.. class:: frontera.contrib.backends.CommonBackend


.. _frontier-backends-memory:

Memory backends
^^^^^^^^^^^^^^^

This set of :class:`Backend <frontera.core.components.Backend>` objects will use an `heapq`_ module as queue and native
dictionaries as storage for :ref:`basic algorithms <frontier-backends-basic-algorithms>`.


.. class:: frontera.contrib.backends.memory.BASE

    Base class for in-memory :class:`Backend <frontera.core.components.Backend>` objects.

.. class:: frontera.contrib.backends.memory.FIFO

    In-memory :class:`Backend <frontera.core.components.Backend>` implementation of `FIFO`_ algorithm.

.. class:: frontera.contrib.backends.memory.LIFO

    In-memory :class:`Backend <frontera.core.components.Backend>` implementation of `LIFO`_ algorithm.

.. class:: frontera.contrib.backends.memory.BFS

    In-memory :class:`Backend <frontera.core.components.Backend>` implementation of `BFS`_ algorithm.

.. class:: frontera.contrib.backends.memory.DFS

    In-memory :class:`Backend <frontera.core.components.Backend>` implementation of `DFS`_ algorithm.

.. class:: frontera.contrib.backends.memory.RANDOM

    In-memory :class:`Backend <frontera.core.components.Backend>` implementation of a random selection
    algorithm.


.. _frontier-backends-sqlalchemy:

SQLAlchemy backends
^^^^^^^^^^^^^^^^^^^

This set of :class:`Backend <frontera.core.components.Backend>` objects will use `SQLAlchemy`_ as storage for
:ref:`basic algorithms <frontier-backends-basic-algorithms>`.

By default it uses an in-memory SQLite database as a storage engine, but `any databases supported by SQLAlchemy`_ can
be used.


If you need to use your own `declarative sqlalchemy models`_, you can do it by using the
:setting:`SQLALCHEMYBACKEND_MODELS` setting.

This setting uses a dictionary where ``key`` represents the name of the model to define and ``value`` the model to use.

For a complete list of all settings used for SQLAlchemy backends check the :doc:`settings <frontera-settings>` section.

.. class:: frontera.contrib.backends.sqlalchemy.BASE

    Base class for SQLAlchemy :class:`Backend <frontera.core.components.Backend>` objects.

.. class:: frontera.contrib.backends.sqlalchemy.FIFO

    SQLAlchemy :class:`Backend <frontera.core.components.Backend>` implementation of `FIFO`_ algorithm.

.. class:: frontera.contrib.backends.sqlalchemy.LIFO

    SQLAlchemy :class:`Backend <frontera.core.components.Backend>` implementation of `LIFO`_ algorithm.

.. class:: frontera.contrib.backends.sqlalchemy.BFS

    SQLAlchemy :class:`Backend <frontera.core.components.Backend>` implementation of `BFS`_ algorithm.

.. class:: frontera.contrib.backends.sqlalchemy.DFS

    SQLAlchemy :class:`Backend <frontera.core.components.Backend>` implementation of `DFS`_ algorithm.

.. class:: frontera.contrib.backends.sqlalchemy.RANDOM

    SQLAlchemy :class:`Backend <frontera.core.components.Backend>` implementation of a random selection
    algorithm.


Revisiting backend
^^^^^^^^^^^^^^^^^^

Based on custom SQLAlchemy backend, and queue. Crawling starts with seeds. After seeds are crawled, every new
document will be scheduled for immediate crawling. On fetching every new document will be scheduled for recrawling
after fixed interval set by :setting:`SQLALCHEMYBACKEND_REVISIT_INTERVAL`.

Current implementation of revisiting backend has no prioritization. During long term runs spider could go idle, because
there are no documents available for crawling, but there are documents waiting for their scheduled revisit time.


.. class:: frontera.contrib.backends.sqlalchemy.revisiting.Backend

    Base class for SQLAlchemy :class:`Backend <frontera.core.components.Backend>` implementation of revisiting back-end.


HBase backend
^^^^^^^^^^^^^

.. class:: frontera.contrib.backends.hbase.HBaseBackend

Is more suitable for large scale web crawlers. Settings reference can be found here :ref:`hbase-settings`. Consider
tunning a block cache to fit states within one block for average size website. To achieve this it's recommended to use
:attr:`hostname_local_fingerprint <frontera.utils.fingerprint.hostname_local_fingerprint>`

to achieve documents closeness within the same host. This function can be selected with :setting:`URL_FINGERPRINT_FUNCTION`
setting.

..  TODO: document details of block cache tuning,
    BC* settings and queue get operation concept,
    hbase tables schema and data flow
    Queue exploration
    shuffling with MR jobs

.. _FIFO: http://en.wikipedia.org/wiki/FIFO
.. _LIFO: http://en.wikipedia.org/wiki/LIFO_(computing)
.. _DFS: http://en.wikipedia.org/wiki/Depth-first_search
.. _BFS: http://en.wikipedia.org/wiki/Breadth-first_search
.. _OrderedDict: https://docs.python.org/2/library/collections.html#collections.OrderedDict
.. _heapq: https://docs.python.org/2/library/heapq.html
.. _SQLAlchemy: http://www.sqlalchemy.org/
.. _any databases supported by SQLAlchemy: http://docs.sqlalchemy.org/en/latest/dialects/index.html
.. _declarative sqlalchemy models: http://docs.sqlalchemy.org/en/latest/orm/extensions/declarative/index.html
