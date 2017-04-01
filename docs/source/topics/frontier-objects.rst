================
Frontier 对象
================

Frontier 使用两种对象类型: :class:`Request <frontera.core.models.Request>`
and :class:`Response <frontera.core.models.Response>`. 他们各自代表 HTTP 请求和 HTTP 返回.

这两个类会被大多数的 Frontera API 方法调用，根据方法不同可能作为参数也可能作为返回值。

Frontera 同样也会使用这两种对象在内部组件之间传递数据（比如 middlewares 和 backend）。

Request 对象
===============

.. class:: frontera.core.models.Request
    :members:


Response 对象
================

.. class:: frontera.core.models.Response
    :members:

Request objects
===============

.. class:: frontera.core.models.Request
    :members:


Response objects
================

.. class:: frontera.core.models.Response
    :members:


``domain`` 和 ``fingerprint`` 字段被 :ref:`内置 middlewares <frontier-built-in-middleware>` 添加。

对象唯一识别标志
==========================

因为 Frontera 对象会在爬虫和服务器之间传递，所以需要一些机制来唯一标示一个对象。这个识别机制会基于 Frontera 逻辑不同而有所变化（大多数情况是根据后端的逻辑）。

默认 Frontera 会激活 :ref:`fingerprint middleware <frontier-url-fingerprint-middleware>` ，根据 :attr:`Request.url <frontera.core.models.Request.url>`
和 :attr:`Response.url <frontera.core.models.Response.url>` 分别生成一个唯一标示，并分别赋值给 :attr:`Request.meta <frontera.core.models.Request.meta>` and
:attr:`Response.meta <frontera.core.models.Response.meta>`。你可以使用这个中间件或者自己定义。

一个为 :class:`Request <frontera.core.models.Request>` 生成指纹的例子::

    >>> request.url
    'http://thehackernews.com'

    >>> request.meta['fingerprint']
    '198d99a8b2284701d6c147174cd69a37a7dea90f'


.. _frontier-objects-additional-data:


为对象添加其他值
=================================

大多数情况下 Frontera 存储了系统运行所需要的参数。

同样的，其他信息也可以存入 :attr:`Request.meta <frontera.core.models.Request.meta>` 和 :attr:`Response.meta <frontera.core.models.Response.meta>`

例如，激活 :ref:`domain middleware <frontier-domain-middleware>` 会为每个 :attr:`Request.meta <frontera.core.models.Request.meta>` 和
:attr:`Response.meta <frontera.core.models.Response.meta>` 添加 ``domain`` 字段::

    >>> request.url
    'http://www.scrapinghub.com'

    >>> request.meta['domain']
    {
        "name": "scrapinghub.com",
        "netloc": "www.scrapinghub.com",
        "scheme": "http",
        "sld": "scrapinghub",
        "subdomain": "www",
        "tld": "com"
    }
