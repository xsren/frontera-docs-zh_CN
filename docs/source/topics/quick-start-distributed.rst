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


Get a spider example code
=========================

First checkout a GitHub Frontera repository:
::

    $ git clone https://github.com/scrapinghub/frontera.git

There is a general spider example in ``examples/general-spider`` folder.

This is a general spider, it does almost nothing except extracting links from downloaded content. It also contains some
settings files, please consult :doc:`settings reference <frontera-settings>` to get more information.

.. _running_zeromq_broker:

Start cluster
=============

First, let's start ZeroMQ broker. ::

    $ python -m frontera.contrib.messagebus.zeromq.broker

You should see a log output of broker with statistics on messages transmitted.

All further commands have to be made from ``general-spider`` root directory.

Second, let's start DB worker. ::

    $ python -m frontera.worker.db --config frontier.workersettings


You should notice that DB is writing messages to the output. It's ok if nothing is written in ZeroMQ sockets, because
of absence of seed URLs in the system.

There are Spanish (.es zone) internet URLs from DMOZ directory in general spider repository, let's use them as
seeds to bootstrap crawling.
Starting the spiders: ::

    $ scrapy crawl general -L INFO -s FRONTERA_SETTINGS=frontier.spider_settings -s SEEDS_SOURCE=seeds_es_smp.txt -s SPIDER_PARTITION_ID=0
    $ scrapy crawl general -L INFO -s FRONTERA_SETTINGS=frontier.spider_settings -s SPIDER_PARTITION_ID=1


You should end up with 2 spider processes running. Each should read it's own Frontera config, and first one is using
``SEEDS_SOURCE`` option to read seeds to bootstrap Frontera cluster.

After some time seeds will pass the streams and will be scheduled for downloading by workers. At this moment crawler
is bootstrapped. Now you can periodically check DB worker output or ``metadata`` table contents to see that there is
actual activity.
