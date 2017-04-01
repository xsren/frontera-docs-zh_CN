.. _topics-index:

================================
Frontera |version| 文档
================================

`Frontera`_ 是一个爬虫工具箱，它可以让你构建任何规模和任意目的的爬虫。

`Frontera`_ 提供 :ref:`crawl frontier <crawl-frontier>` 框架，这个框架可以帮助解决*何时抓取下一个URL*、*下个抓取的URL是什么*和检查*抓取结果*等问题。

Frontera 还为所有的爬虫组件提供了复制、分片、隔离的特性，这可以方便的扩展爬虫规模和将爬虫做成分布式。

Fronteta 包含完全支持 `Scrapy`_ 的组件，可以使用Scrapy的所有功能创建爬虫。尽管它最初是为Scrapy设计的，但是它也可以完美契合其他任何的框架或系统，因为它可以作为一个框架提供通用的工具箱。

介绍
============

这一章的目的是介绍 Frontera 的概念，通过阅读本章，你可以知道 Frontera 的设计理念和确定它能不能满足你的需求。


.. toctree::
   :hidden:

   topics/overview
   topics/run-modes
   topics/quick-start-single
   topics/quick-start-distributed
   topics/cluster-setup

:doc:`topics/overview`
    明白什么是 Frontera ？它能为你做什么？

:doc:`topics/run-modes`
    Frontera的高层体系结构和运行模式。

:doc:`topics/quick-start-single`
    使用 Scrapy 作为容器来运行 Frontera。

:doc:`topics/quick-start-distributed`
    引入 SQLite 和 ZeroMQ。

:doc:`topics/cluster-setup`
    Setting up clustered version of Frontera on multiple machines with HBase and Kafka.
    使用 HBase 和 Kafka 在多台机器上部署 Frontera 集群。
    

使用 Frontera
==============

.. toctree::
   :hidden:

   topics/installation
   topics/frontier-objects
   topics/frontier-middlewares
   topics/frontier-canonicalsolvers
   topics/frontier-backends
   topics/message_bus
   topics/own_crawling_strategy
   topics/scrapy-integration
   topics/frontera-settings

:doc:`topics/installation`
    安装方法和依赖的选项。

:doc:`topics/frontier-objects`
    理解用来代表网络请求和网络响应的类。

:doc:`topics/frontier-middlewares`
    过滤或者更改链接和网页的信息。

:doc:`topics/frontier-canonicalsolvers`
    确认和使用网页的规范url。

:doc:`topics/frontier-backends`
    自定义抓取规则和存储方式。

:doc:`topics/message_bus`
    内置消息总线参考。

:doc:`topics/own_crawling_strategy`
    为分布式后端实现自己的抓取策略。

:doc:`topics/scrapy-integration`
    学习如何使用 Frontera + Scrapy 。

:doc:`topics/frontera-settings`
    设置参考。


高级用法
==============

.. toctree::
   :hidden:

   topics/what-is-cf
   topics/graph-manager
   topics/scrapy-recorder
   topics/fine-tuning
   topics/dns-service

:doc:`topics/what-is-cf`
    学习 Crawl Frontier 理论。

:doc:`topics/graph-manager`
    定义假的抓取规则来测试你的 frontier 。

:doc:`topics/scrapy-recorder`
    创建 Scrapy 抓取记录，并在之后重现他们。

:doc:`topics/fine-tuning`
    机器部署和微调信息。

:doc:`topics/dns-service`
    DNS 服务搭建简介。

开发者文档
=======================

.. toctree::
   :hidden:

   topics/architecture
   topics/frontier-api
   topics/requests-integration
   topics/examples
   topics/tests
   topics/loggers
   topics/frontier-tester
   topics/faq
   topics/contributing
   topics/glossary




:doc:`topics/architecture`
    了解 Frontera 如何工作和它的不同组件。

:doc:`topics/frontier-api`
    学习如何使用 frontier 。

:doc:`topics/requests-integration`
    学习如何使用 Frontera + Requests 。 

:doc:`topics/examples`
    一些使用 Frontera 的示例工程和示例脚本。

:doc:`topics/tests`
    如果运行和写 Frontera 的测试用例。

:doc:`topics/loggers`
    使用 python 原生日志系统创建的一些 loggers 。

:doc:`topics/frontier-tester`
    使用一个简单的方法测试你的 frontier。

:doc:`topics/faq`
    常见问题。

:doc:`topics/contributing`
    如何贡献。


:doc:`topics/glossary`
    术语表。


.. _Crawling System: http://en.wikipedia.org/wiki/Web_crawler
.. _Scrapy: http://scrapy.org/
.. _`Frontera`: http://github.com/scrapinghub/frontera