==============================
使用 Frontier 和 Scrapy
==============================


Scrapy 中使用 Frontera 非常简单，它包含一组 `Scrapy middlewares`_ 和 Scrapy 调度程序，封装了 Frontera 的功能 ，可以使用 `Scrapy settings`_ 轻松配置。

激活 frontier
=======================

Frontera 使用两种不同的中间件：``SchedulerSpiderMiddleware`` and ``SchedulerDownloaderMiddleware`` 和自己的调度程序 ``FronteraScheduler``。

要在你的 Scrapy 项目中激活 Frontera，只要把它们加入到 `SPIDER_MIDDLEWARES`_,
`DOWNLOADER_MIDDLEWARES`_ 和 `SCHEDULER`_ settings::

    SPIDER_MIDDLEWARES.update({
        'frontera.contrib.scrapy.middlewares.schedulers.SchedulerSpiderMiddleware': 1000,
    })

    DOWNLOADER_MIDDLEWARES.update({
        'frontera.contrib.scrapy.middlewares.schedulers.SchedulerDownloaderMiddleware': 1000,
    })

    SCHEDULER = 'frontera.contrib.scrapy.schedulers.frontier.FronteraScheduler'

创建一个 Frontera ``settings.py`` 并把它加入到你的 Scrapy 设置中::

    FRONTERA_SETTINGS = 'tutorial.frontera.settings'

另一种选择是将这些设置放在 Scrapy 设置模块中。



组织文件
================

当使用 Scrapy 和 frontier 时，我们有以下目录结构::

    my_scrapy_project/
        my_scrapy_project/
            frontera/
                __init__.py
                settings.py
                middlewares.py
                backends.py
            spiders/
                ...
            __init__.py
            settings.py
         scrapy.cfg

这些都是基本的：

- ``my_scrapy_project/frontera/settings.py``:  Frontera settings 文件。
- ``my_scrapy_project/frontera/middlewares.py``: Frontera 的中间件。
- ``my_scrapy_project/frontera/backends.py``: Frontera 使用的后端。
- ``my_scrapy_project/spiders``: Scrapy spiders 文件夹
- ``my_scrapy_project/settings.py``: Scrapy settings 文件
- ``scrapy.cfg``: Scrapy 配置文件

运行爬虫
=================

只要按照常规从命令行运行你的 Scrapy 爬虫::

    scrapy crawl myspider


Frontier Scrapy settings
========================
你可以使用两种方式配置 frontier :

.. setting:: FRONTERA_SETTINGS

- 使用 ``FRONTERA_SETTINGS``，可以在 Scrapy 配置文件中指明 Frontera 配置文件的路径，默认是 ``None``

- 将 frontier 设置定义到 Scrapy 设置文件中。

通过 Scrapy 配置 frontier
----------------------------------------------

:ref:`Frontier settings <frontier-built-in-frontier-settings>` 也可以通过 Scrapy 配置。在这种情况下，配置优先级顺序如下：

1. :setting:`FRONTERA_SETTINGS` 指向的文件中的配置（更高的优先级）
2. Scrapy 配置文件中的配置
3. 默认 frontier 配置


.. _Scrapy middlewares: http://doc.scrapy.org/en/latest/topics/downloader-middleware.html
.. _Scrapy settings: http://doc.scrapy.org/en/latest/topics/settings.html
.. _DOWNLOADER_MIDDLEWARES: http://doc.scrapy.org/en/latest/topics/settings.html#std:setting-DOWNLOADER_MIDDLEWARES
.. _SPIDER_MIDDLEWARES: http://doc.scrapy.org/en/latest/topics/settings.html#std:setting-SPIDER_MIDDLEWARES
.. _SCHEDULER: http://doc.scrapy.org/en/latest/topics/settings.html#std:setting-SCHEDULER


写 Scrapy 爬虫
=====================

爬虫逻辑
------------
创建基本的 Scrapy 爬虫在 `Quick start single process`_ 中做了描述。

这也是一个防止爬虫因为队列中请求不足而关闭的好方法::

    @classmethod
    def from_crawler(cls, crawler, *args, **kwargs):
        spider = cls(*args, **kwargs)
        spider._set_crawler(crawler)
        spider.crawler.signals.connect(spider.spider_idle, signal=signals.spider_idle)
        return spider

    def spider_idle(self):
        self.log("Spider idle signal caught.")
        raise DontCloseSpider


配置准则
------------------------

您可以进行多种调整，以实现高效的广泛抓取。

添加一个种子加载器，用于启动爬虫进程::

    SPIDER_MIDDLEWARES.update({
        'frontera.contrib.scrapy.middlewares.seeds.file.FileSeedLoader': 1,
    })

适合广泛抓取的各种设置::

    HTTPCACHE_ENABLED = False   # 关闭磁盘缓存，它在大量抓取中具有较低的命中率
    REDIRECT_ENABLED = True
    COOKIES_ENABLED = False
    DOWNLOAD_TIMEOUT = 120
    RETRY_ENABLED = False   # 重试可以由 Frontera 本身处理，具体取决于爬网策略
    DOWNLOAD_MAXSIZE = 10 * 1024 * 1024  # 最大文档大小，如果未设置，会导致OOM
    LOGSTATS_INTERVAL = 10  # 每10秒钟向控制台打印统计

自动限流和并发设置，以方便有礼貌和负责任的抓取::

    # auto throttling
    AUTOTHROTTLE_ENABLED = True
    AUTOTHROTTLE_DEBUG = False
    AUTOTHROTTLE_MAX_DELAY = 3.0
    AUTOTHROTTLE_START_DELAY = 0.25     # 任何足够小的值，它将在平均运行期间通过瓶颈响应延迟进行调整。
    RANDOMIZE_DOWNLOAD_DELAY = False

    # concurrency
    CONCURRENT_REQUESTS = 256           # 取决于许多因素，应通过实验确定
    CONCURRENT_REQUESTS_PER_DOMAIN = 10
    DOWNLOAD_DELAY = 0.0

具体参照 `Scrapy broad crawling`_.


.. _`Quick start single process`: http://frontera.readthedocs.org/en/latest/topics/quick-start-single.html
.. _`Scrapy broad crawling`: http://doc.scrapy.org/en/master/topics/broad-crawls.html


Scrapy 种子加载器
===================

Frontera 有一些内置的 Scrapy 中间件用于种子装载。

种子装载使用 ``process_start_requests`` 方法从源中生成请求，这些请求后续会被加入 :class:`FrontierManager <frontera.core.manager.FrontierManager>` 。

激活一个种子加载器
------------------------

只需将种子加载器中间件加入 ``SPIDER_MIDDLEWARES`` 中::

    SPIDER_MIDDLEWARES.update({
        'frontera.contrib.scrapy.middlewares.seeds.FileSeedLoader': 650
    })


.. _seed_loader_file:

FileSeedLoader
--------------

从文件中导入种子。该文件必须是每行一个 URL 的格式::

    http://www.asite.com
    http://www.anothersite.com
    ...

你可以使用 ``#`` 注释掉某一行::

    ...
    #http://www.acommentedsite.com
    ...

**Settings**:

- ``SEEDS_SOURCE``: 种子文件路径


.. _seed_loader_s3:

S3SeedLoader
------------

从存储在 Amazon S3 中的文件导入种子
Load seeds from a file stored in an Amazon S3 bucket

文件格式应该和 :ref:`FileSeedLoader <seed_loader_file>` 中的一样。

Settings:

- ``SEEDS_SOURCE``: S3 bucket 文件路径。 例如: ``s3://some-project/seed-urls/``

- ``SEEDS_AWS_ACCESS_KEY``: S3 credentials Access Key

- ``SEEDS_AWS_SECRET_ACCESS_KEY``: S3 credentials Secret Access Key


.. _`Scrapy Middleware doc`: http://doc.scrapy.org/en/latest/topics/spider-middleware.html
