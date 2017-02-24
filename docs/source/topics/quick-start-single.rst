==========================
快速启动单进程模式
==========================

1. 创建你的爬虫
=====================

按照通常的方式创建 Scrapy 项目。输入您要存储代码的目录，然后运行::

    scrapy startproject tutorial

这会创建一个 tutorial 目录，包含以下内容::


    tutorial/
        scrapy.cfg
        tutorial/
            __init__.py
            items.py
            pipelines.py
            settings.py
            spiders/
                __init__.py
                ...

These are basically:
这些是最基本的：

- **scrapy.cfg**: 项目的配置文件
- **tutorial/**: 项目的 python 模块，后续你将从这里引用你的代码。
- **tutorial/items.py**: 项目的 items 文件。
- **tutorial/pipelines.py**: 项目的 pipelines 文件。
- **tutorial/settings.py**: 项目的 settings 文件。
- **tutorial/spiders/**: 放爬虫的目录。

2. 安装 Frontera
===================

请看 :doc:`installation`.

3. 集成您的爬虫和 Frontera
==========================================

这篇文章 :doc:`integration with Scrapy <scrapy-integration>` 详细介绍了这一步。


4. 选择您的后端
======================

为 Frontera 设置内置的后端，比如内存中BFS（广度优先）::

    BACKEND = 'frontera.contrib.backends.memory.BFS'

5. 运行爬虫
=================

像往常一样从命令行启动 Scrapy 爬虫::

    scrapy crawl myspider

就是这样! 您成功将您的爬虫与 Frontera 集成了。

还有什么？
==========

您已经看到了一个使用 Frontera 集成 Scrapy 的例子，但是这个仅仅是最基本的功能。Frontera 还提供了许多让抓取更加简单、有效率的强大功能，比如：

* 内置 :ref:`database storage <frontier-backends-sqlalchemy>` 支持存储抓取数据。

* 通过 :doc:`API <frontier-api>` 可以方便的 :doc:`与 Scrapy 集成 <scrapy-integration>` 或者与其他爬虫集成。

* 通过使用 ZeroMq 或 Kafka 和分布式的后端，实现 :ref:`两种分布式抓取模式 <use-cases>`

* 创建不同抓取策略或者逻辑 :doc:`自定义你的后端 <frontier-backends>`.

* 使用 :doc:`middlewares <frontier-middlewares>` 插入你自己的 request/response 修改策略。

* 使用 :doc:`Graph Manager <graph-manager>` 创建假的网站地图，并可以不用爬虫重现抓取过程。

* :doc:`记录你的 Scrapy 抓取结果 <scrapy-recorder>` 后续用它测试 frontier。

* 您可以用 hook 的方式使用日志工具，捕捉错误和调试您的 frontiers。




