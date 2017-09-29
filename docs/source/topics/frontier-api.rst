============
Frontera API
============

本节介绍了 Frontera 核心API，适用于中间件和后端的开发人员。

Frontera API / Manager
======================

Frontera API的主要入口点是 :class:`FrontierManager <frontera.core.manager.FrontierManager>` 对象，通过from_manager类方法传递给中间件和后端。该对象提供对所有Frontera核心组件的访问，并且是中间件和后端访问它们并将其功能挂接到Frontera中的唯一方法。

:class:`FrontierManager <frontera.core.manager.FrontierManager>`  负责加载安装的中间件和后端，以及用于管理整个 frontier 的数据流。

.. _frontier-loading-from-settings:

从settings加载
=====================

尽管 :class:`FrontierManager <frontera.core.manager.FrontierManager>` 可以通过参数初始化，但最常用的初始化方法还是使用 :doc:`Frontera Settings <frontera-settings>` 。

这个可以通过 :attr:`from_settings <frontera.core.manager.FrontierManager.from_settings>` 类方法实现，使用字符串路径::

    >>> from frontera import FrontierManager
    >>> frontier = FrontierManager.from_settings('my_project.frontier.settings')

或者一个 :class:`BaseSettings <frontera.settings.BaseSettings>` 对象::

    >>> from frontera import FrontierManager, Settings
    >>> settings = Settings()
    >>> settings.MAX_PAGES = 0
    >>> frontier = FrontierManager.from_settings(settings)

也可以无参数初始化，这种情况下 frontier 会使用 :ref:`默认配置 <frontier-default-settings>` ::

    >>> from frontera import FrontierManager, Settings
    >>> frontier = FrontierManager.from_settings()


Frontier Manager
================


.. autoclass:: frontera.core.manager.FrontierManager

    **Attributes**

    .. autoattribute:: frontera.core.manager.FrontierManager.request_model
    .. autoattribute:: frontera.core.manager.FrontierManager.response_model
    .. autoattribute:: frontera.core.manager.FrontierManager.backend
    .. autoattribute:: frontera.core.manager.FrontierManager.middlewares
    .. autoattribute:: frontera.core.manager.FrontierManager.test_mode
    .. autoattribute:: frontera.core.manager.FrontierManager.max_requests
    .. autoattribute:: frontera.core.manager.FrontierManager.max_next_requests
    .. autoattribute:: frontera.core.manager.FrontierManager.auto_start
    .. autoattribute:: frontera.core.manager.FrontierManager.settings
    .. autoattribute:: frontera.core.manager.FrontierManager.iteration
    .. autoattribute:: frontera.core.manager.FrontierManager.n_requests
    .. autoattribute:: frontera.core.manager.FrontierManager.finished

    **API Methods**

    .. automethod:: frontera.core.manager.FrontierManager.start
    .. automethod:: frontera.core.manager.FrontierManager.stop
    .. automethod:: frontera.core.manager.FrontierManager.add_seeds
    .. automethod:: frontera.core.manager.FrontierManager.get_next_requests
    .. automethod:: frontera.core.manager.FrontierManager.page_crawled
    .. automethod:: frontera.core.manager.FrontierManager.request_error

    **Class Methods**

    .. automethod:: frontera.core.manager.FrontierManager.from_settings


.. _frontier-start-stop:

启动/停止 frontier
==============================

有时，frontier 组件需要执行初始化和最终化操作。frontier 通过 :attr:`start() <frontera.core.manager.FrontierManager.start>` 和 :attr:`stop() <frontera.core.manager.FrontierManager.stop>` 方法去通知不同组件启动或者停止。

默认 :attr:`auto_start <frontera.core.manager.FrontierManager.auto_start>` 值是激活的，这意味着在创建 :class:`FrontierManager <frontera.core.manager.FrontierManager>` 对象后，组件将被通知。如果您需要对初始化不同组件时进行更精细的控制，请停用 :attr:`auto_start <frontera.core.manager.FrontierManager.auto_start>` 并手动调用frontier API :attr:`start() <frontera.core.manager.FrontierManager.start>` 和 :attr:`stop() <frontera.core.manager.FrontierManager.stop>` 方法。

.. note::
当 :attr:`auto_start <frontera.core.manager.FrontierManager.auto_start>` 处于激活状态时，Frontier :attr:`stop() <frontera.core.manager.FrontierManager.stop>` 方法不会自动调用（因为frontier 不知道抓取状态）。如果您需要通知 frontier 组件，您应该手动调用该方法。


.. _frontier-iterations:

Frontier 迭代
===================

一旦 frontier 运行，通常的过程就是 :ref:`data flow <frontier-data-flow>` 部分所描述的过程。

爬虫调用 :attr:`get_next_requests() <frontera.core.manager.FrontierManager.get_next_requests>` 方法请求接下来要抓取的页面。每次 frontier 返回一个非空列表（可用数据），就是我们所说的前沿迭代。当前 frontier 迭代可以通过 :attr:`iteration <frontera.core.manager.FrontierManager.iteration>` 属性访问。


.. _frontier-finish:

结束 frontier
======================

抓取过程可以被爬虫程序或者 Frontera 停止。当返回最大页数时，Frontera 将结束。此限制由 :attr:`max_requests <frontera.core.manager.FrontierManager.max_requests>` 属性控制（ :setting:`MAX_REQUESTS` 设置）。

如果  :attr:`max_requests <frontera.core.manager.FrontierManager.max_requests>`  设置为0，那么 frontier 会无限抓取下去。

一旦 frontier 完成，:attr:`get_next_requests <frontera.core.manager.FrontierManager.get_next_requests>` 方法将不再返回任何页面，并且 :attr:`finished <frontera.core.manager.FrontierManager.finished>`  属性将为True。

.. _frontier-test-mode:

组件对象
=================

.. autoclass:: frontera.core.components.Component

    **Attributes**

    .. autoattribute:: frontera.core.components.Component.name

    **Abstract methods**

    .. automethod:: frontera.core.components.Component.frontier_start
    .. automethod:: frontera.core.components.Component.frontier_stop
    .. automethod:: frontera.core.components.Component.add_seeds
    .. automethod:: frontera.core.components.Component.page_crawled
    .. automethod:: frontera.core.components.Component.request_error

    **Class Methods**

    .. automethod:: frontera.core.components.Component.from_manager


测试模式
=========

在某些情况下，在测试中，frontier 组件需要采用与通常不同的方式（例如，在测试模式下解析域URL时，:ref:`domain middleware <frontier-domain-middleware>` 会接受诸如 ``'A1'`` 或者 ``'B1'`` 之类的非有效URL）。

组件可以通过 :attr:`test_mode <frontera.core.manager.FrontierManager.test_mode>` 属性知道 frontier 是否处于测试模式。


.. _frontier-another-ways:

使用 frontier 的其他方法
==================================

与 frontier 通信也可以通过HTTP API或队列系统等其他机制完成。这些功能暂时不可用，但希望包含在将来的版本中。