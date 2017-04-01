============================
分布式模式快速入门
============================

这篇文档教您在本地快速搭建单机、多进程的 Frontera 系统。我们将使用 SQLite 和 ZeroMQ 构建可能最简单的 Frontera 系统。如果要搭建生产环境下的 Frontera 系统，请参考 :doc:`cluster-setup`。

.. _basic_requirements:

前提
=============

Here is what services needs to be installed and configured before running Frontera:
以下是运行 Frontera 之前需要安装和配置的：

- Python 2.7+ 或 3.4+
- Scrapy

安装 Frontera
---------------------
Ubuntu 系统, 在命令行中输入： ::

    $ pip install frontera[distributed,zeromq,sql]


得到一个爬虫例子代码
=========================

首先从 Github 上下载 Frontera:
::

    $ git clone https://github.com/scrapinghub/frontera.git

在 ``examples/general-spider`` 中有一个普通的爬虫例子。

这是一个很普通的爬虫，它仅仅从下载的内容中抽取链接。它同样包含了一些配置文件，请参考 :doc:`settings reference <frontera-settings>` 获取更多的信息。

.. _running_zeromq_broker:

启动集群
=============

首先，让我们启动 ZeroMQ 代理。 ::

    $ python -m frontera.contrib.messagebus.zeromq.broker

你应该看到代理打印出 spider 与 DB worker 之间传递信息的统计信息。

后续所有的命令都可以在``general-spider``的根目录下执行。

第二步，启动 DB worker。::

    $ python -m frontera.worker.db --config frontier.workersettings

你应该注意点 DB worker 正在输出信息。此时没有向 ZeroMQ 发送信息是正常的，因为现在系统中缺乏种子URL。

在目录下有一批西班牙的URL，让我们把它们当做种子来启动爬虫。启动爬虫: ::

    $ scrapy crawl general -L INFO -s FRONTERA_SETTINGS=frontier.spider_settings -s SEEDS_SOURCE=seeds_es_smp.txt -s SPIDER_PARTITION_ID=0
    $ scrapy crawl general -L INFO -s FRONTERA_SETTINGS=frontier.spider_settings -s SPIDER_PARTITION_ID=1

最后应该有两个爬虫进程在运行。每个爬虫应该读取自己的Frontera配置，并且第一个应该使用 ``SEEDS_SOURCE`` 选项来读取种子和启动 Frontera 集群。

一段时间以后，种子会被准备好，以供爬虫抓取。此时爬虫已经被启动了。现在你可以周期性的检查 DB worker 的输出或者 ``metadata`` 表信息来确认爬虫确实在运行。

