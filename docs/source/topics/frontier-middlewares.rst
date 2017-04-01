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


.. autoclass:: frontera.core.components.Middleware

    **Methods**

    .. automethod:: frontera.core.components.Middleware.frontier_start
    .. automethod:: frontera.core.components.Middleware.frontier_stop
    .. automethod:: frontera.core.components.Middleware.add_seeds

        :return: :class:`Request <frontera.core.models.Request>` object list or ``None``

        Should either return ``None`` or a list of :class:`Request <frontera.core.models.Request>` objects.

        If it returns ``None``, :class:`FrontierManager <frontera.core.manager.FrontierManager>` won't continue
        processing any other middleware and seed will never reach the
        :class:`Backend <frontera.core.components.Backend>`.

        If it returns a list of :class:`Request <frontera.core.models.Request>` objects, this will be passed to
        next middleware. This process will repeat for all active middlewares until result is finally passed to the
        :class:`Backend <frontera.core.components.Backend>`.

        If you want to filter any seed, just don't include it in the returned object list.

    .. automethod:: frontera.core.components.Middleware.page_crawled

        :return: :class:`Response <frontera.core.models.Response>` or ``None``

        Should either return ``None`` or a :class:`Response <frontera.core.models.Response>` object.

        If it returns ``None``, :class:`FrontierManager <frontera.core.manager.FrontierManager>` won't continue
        processing any other middleware and :class:`Backend <frontera.core.components.Backend>` will never be
        notified.

        If it returns a :class:`Response <frontera.core.models.Response>` object, this will be passed to
        next middleware. This process will repeat for all active middlewares until result is finally passed to the
        :class:`Backend <frontera.core.components.Backend>`.

        If you want to filter a page, just return None.

    .. automethod:: frontera.core.components.Middleware.request_error


        :return: :class:`Request <frontera.core.models.Request>` or ``None``

        Should either return ``None`` or a :class:`Request <frontera.core.models.Request>` object.

        If it returns ``None``, :class:`FrontierManager <frontera.core.manager.FrontierManager>` won't continue
        processing any other middleware and :class:`Backend <frontera.core.components.Backend>` will never be
        notified.

        If it returns a :class:`Response <frontera.core.models.Request>` object, this will be passed to
        next middleware. This process will repeat for all active middlewares until result is finally passed to the
        :class:`Backend <frontera.core.components.Backend>`.

        If you want to filter a page error, just return None.

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
