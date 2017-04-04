.. _frontier-middlewares:

===========
Middlewares（中间件）
===========

Frontier :class:`Middleware <frontera.core.components.Middleware>` 位于
:class:`FrontierManager <frontera.core.manager.FrontierManager>` 和
:class:`Backend <frontera.core.components.Backend>` objects 之间, 根据 :ref:`frontier data flow <frontier-data-flow>` 的流程，处理 :class:`Request <frontera.core.models.Request>` 和 :class:`Response <frontera.core.models.Response>`。

Middlewares 是一个轻量级、低层次的系统，可以用来过滤和更改 Frontier 的 requests 和 responses。

.. _frontier-activating-middleware:

激活一个 middleware
=======================

要激活 :class:`Middleware <frontera.core.components.Middleware>` component, 需要添加它到
:setting:`MIDDLEWARES` setting（这是一个列表，包含类的路径或者一个 :class:`Middleware <frontera.core.components.Middleware>` 对象）。

这是一个例子::

    MIDDLEWARES = [
        'frontera.contrib.middlewares.domain.DomainMiddleware',
    ]

Middlewares按照它们在列表中定义的相同顺序进行调用，根据你自己的需要安排顺序。 该顺序很重要，因为每个中间件执行不同的操作，并且您的中间件可能依赖于一些先前（或后续的）执行的中间件。

最后，记住一些 middlewares 需要通过特殊的 setting。详细请参考 :ref:`each middleware documentation <frontier-built-in-middleware>` 。

.. _frontier-writing-middleware:

写你自己的 middleware
===========================


写自己的 Frontera middleware 是很简单的。每个 :class:`Middleware <frontera.core.components.Middleware>` 是一个继承 :class:`Component <frontera.core.components.Component>` 的 Python 类。


:class:`FrontierManager <frontera.core.manager.FrontierManager>` 会通过下面的方法和所有激活的 middlewares 通信。


.. class:: frontera.core.components.Middleware

    **Methods**

    .. automethod:: frontera.core.components.Middleware.frontier_start
    .. automethod:: frontera.core.components.Middleware.frontier_stop
    .. automethod:: frontera.core.components.Middleware.add_seeds

        :return: :class:`Request <frontera.core.models.Request>` object list or ``None``

        应该返回 ``None`` 或者 :class:`Request <frontera.core.models.Request>` 的列表。

        如果返回 ``None`` ， :class:`FrontierManager <frontera.core.manager.FrontierManager>` 将不会处理任何中间件，并且种子也不会到达 :class:`Backend <frontera.core.components.Backend>` 。

        如果返回 :class:`Request <frontera.core.models.Request>` 列表，该列表将会传给下个中间件。这个过程会在每个激活的中间件重复，直到它到达 :class:`Backend <frontera.core.components.Backend>`。

        如果要过滤任何种子，请不要将其包含在返回的对象列表中。

    .. automethod:: frontera.core.components.Middleware.page_crawled

        :return: :class:`Response <frontera.core.models.Response>` or ``None``

        应该返回 ``None`` 或者一个 :class:`Response <frontera.core.models.Response>` 对象。

        如果返回 ``None`` ，:class:`FrontierManager <frontera.core.manager.FrontierManager>` 将不会处理任何中间件，并且 :class:`Backend <frontera.core.components.Backend>` 不会被通知。

        如果返回 :class:`Response <frontera.core.models.Response>`，它将会被传给下个中间件。这个过程会在每个激活的中间件重复，直到它到达 :class:`Backend <frontera.core.components.Backend>`。

        如果要过滤页面，只需返回 None。

    .. automethod:: frontera.core.components.Middleware.request_error


        :return: :class:`Request <frontera.core.models.Request>` or ``None``

        应该返回 ``None`` 或者一个 :class:`Request <frontera.core.models.Request>` 对象。

        如果返回 ``None``，:class:`FrontierManager <frontera.core.manager.FrontierManager>` 将不会和其他任何中间件通信，并且 :class:`Backend <frontera.core.components.Backend>` 不会被通知。

        如果返回一个 :class:`Response <frontera.core.models.Request>` 对象，它将会被传给下个中间件。这个过程会在每个激活的中间件重复，直到它到达 :class:`Backend <frontera.core.components.Backend>`。

        如果要过滤页面错误，只需返回 None。

    **Class Methods**

    .. automethod:: frontera.core.components.Middleware.from_manager



.. _frontier-built-in-middleware:

内置 middleware 参考
=============================

这篇文章描述了 Frontera 所有的 :class:`Middleware <frontera.core.components.Middleware>` 组件。如何使用和写自己的 middleware，请参考 :ref:`middleware usage guide. <frontier-writing-middleware>`。

有关默认启用的组件列表（及其顺序），请参阅 MIDDLEWARES 设置。


.. _frontier-domain-middleware:

DomainMiddleware
----------------

.. autoclass:: frontera.contrib.middlewares.domain.DomainMiddleware()


.. _frontier-url-fingerprint-middleware:

UrlFingerprintMiddleware
------------------------

.. autoclass:: frontera.contrib.middlewares.fingerprint.UrlFingerprintMiddleware()
.. autofunction:: frontera.utils.fingerprint.hostname_local_fingerprint



.. _frontier-domain-fingerprint-middleware:

DomainFingerprintMiddleware
---------------------------

.. autoclass:: frontera.contrib.middlewares.fingerprint.DomainFingerprintMiddleware()
