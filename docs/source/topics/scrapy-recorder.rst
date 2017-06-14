========================
记录 Scrapy 抓取过程
========================

Scrapy Recorder 是一套 `Scrapy middlewares`_ ，可以让您记录scrapy抓取过程并将其存储到 :doc:`Graph Manager <graph-manager>` 中。

这可以用于执行 frontier 测试，而不必再次爬取整个站点，甚至不用使用 Scrapy。


激活 recorder
=======================

recorder 使用了两个中间件： ``CrawlRecorderSpiderMiddleware`` 和 ``CrawlRecorderDownloaderMiddleware`` 。

要在 Scrapy 项目中激活 recorder，只需要将他们加入 `SPIDER_MIDDLEWARES`_  和 `DOWNLOADER_MIDDLEWARES`_ 配置中::

    SPIDER_MIDDLEWARES.update({
        'frontera.contrib.scrapy.middlewares.recording.CrawlRecorderSpiderMiddleware': 1000,
    })

    DOWNLOADER_MIDDLEWARES.update({
        'frontera.contrib.scrapy.middlewares.recording.CrawlRecorderDownloaderMiddleware': 1000,
    })


选择你的存储引擎
============================

因为 recorder 内部使用 :doc:`Graph Manager <graph-manager>` 存储抓取的网页，所以你可以选择存储引擎，参照 :ref:`different storage engines <graph-manager-database>` 。

我们使用 :setting:`RECORDER_STORAGE_ENGINE <RECORDER_STORAGE_ENGINE>` 配置存储引擎::

    RECORDER_STORAGE_ENGINE = 'sqlite:///my_record.db'

您还可以选择重置数据库表或仅重置数据::

    RECORDER_STORAGE_DROP_ALL_TABLES = True
    RECORDER_STORAGE_CLEAR_CONTENT = True

运行爬虫
=================

和之前一样从命令行运行爬虫::

    scrapy crawl myspider

一旦完成，抓取过程会被记录。

如果你需要取消记录，可以设置 :setting:`RECORDER_ENABLED <RECORDER_ENABLED>` ::

    scrapy crawl myspider -s RECORDER_ENABLED=False

Recorder 设置
=================

以下是所有可用Scrapy Recorder设置的列表，按字母顺序排列，以及默认值及其应用范围。

.. setting:: RECORDER_ENABLED

RECORDER_ENABLED
----------------

默认： ``True``

激活或停用中间件。

.. setting:: RECORDER_STORAGE_CLEAR_CONTENT

RECORDER_STORAGE_CLEAR_CONTENT
------------------------------

默认： ``True``

删除 :ref:`storage database <graph-manager-database>` 中的数据。

.. setting:: RECORDER_STORAGE_DROP_ALL_TABLES

RECORDER_STORAGE_DROP_ALL_TABLES
--------------------------------

默认： ``True``

删除 :ref:`storage database <graph-manager-database>` 中的表。

.. setting:: RECORDER_STORAGE_ENGINE

RECORDER_STORAGE_ENGINE
-----------------------

默认： ``None``

设置 :ref:`Graph Manager storage engine <graph-manager-database>` 来存储记录。

.. _Scrapy middlewares: http://doc.scrapy.org/en/latest/topics/downloader-middleware.html
.. _DOWNLOADER_MIDDLEWARES: http://doc.scrapy.org/en/latest/topics/settings.html#std:setting-DOWNLOADER_MIDDLEWARES
.. _SPIDER_MIDDLEWARES: http://doc.scrapy.org/en/latest/topics/settings.html#std:setting-SPIDER_MIDDLEWARES
