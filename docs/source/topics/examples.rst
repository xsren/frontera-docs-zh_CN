========
例子
========

这个项目包含了 ``examples`` 文件夹，其中有使用 Frontera 的代码 ::

    examples/
        requests/
        general-spider/
        scrapy_recording/
        scripts/


- **requests**: Example script with `Requests`_ library.
- **general-spider**: Scrapy 整合示例项目。
- **scrapy_recording**: Scrapy 记录示例项目。
- **scripts**: 一些简单的脚本。

.. note::

    **这个例子可能需要安装额外的库才能工作**.

    你可以使用 pip 来安装它们::


        pip install -r requirements/examples.txt


requests
========

一个使用 `Requests`_ 库，抓取一个网站所有链接的脚本。

运行::

    python links_follower.py


general-spider
==============

一个简单的 Scrapy 爬虫，执行所有的种子任务。包含单进程，分布式爬虫和后端运行模式的配置文件。

查看 :doc:`quick-start-distributed` 如何运行。

cluster
=======

是一个大型可扩展爬虫程序，用于抓取每个域有限制的大量网站。它在HBase中保留每个域的状态，并在安排新的下载请求时使用它。设计用于使用 HBase 在分布式后端运行模式下运行。

scrapy_recording
================

一个带有爬虫的简单脚本，可以跟踪站点的所有链接，记录抓取结果。

运行::

    scrapy crawl recorder


scripts
=======

一些关于如何使用不同 frontier 组件的示例脚本。


.. _Requests: http://docs.python-requests.org/en/latest/