=============
Graph Manager
=============

Graph Manager 是一种将网站地图表示为图形的工具。

它可以很容易地用于测试 frontier。我们可以向 graph manager 查询页面来伪造爬虫的请求/响应，并且不使用爬虫就可以知道每个页面的提取链接。你可以使用你的伪造测试或者  :doc:`Frontier Tester tool <frontier-tester>` 。

你可以用它来定义你自己的网站，用来测试或者使用 :doc:`Scrapy Recorder <scrapy-recorder>` 来记录抓取过程并可以在之后重现。


定义一个网站图
=====================

网站的页面及其链接可以轻松定义为有向图，其中每个节点表示页面边缘之间的链接。

我们使用一个非常简单的站点表示，一个起始页面 `A` 里面有一个链接到 `B, C, D` 里面的链接。
我们可以用这个图表来表示网站：

.. image:: _images/site_01.png
   :width: 200px
   :height: 179px

我们使用列表来表示不同的网站页面和元组来定义页面及其链接，上面的例子 ::

    site = [
        ('A', ['B', 'C', 'D']),
    ]

请注意，我们不需要定义没有链接的页面，但是我们也可以将其用作有效的表示 ::

    site = [
        ('A', ['B', 'C', 'D']),
        ('B', []),
        ('C', []),
        ('D', []),
    ]

一个更复杂的网站:

.. image:: _images/site_02.png
   :width: 220px
   :height: 272px

可以表示为::

    site = [
        ('A', ['B', 'C', 'D']),
        ('D', ['A', 'D', 'E', 'F']),
    ]

注意 `D` 链接到自己和它的父节点 `A` 。

同样的，一个页面可以有多个父节点:

.. image:: _images/site_03.png
   :width: 160px
   :height: 310px

::

    site = [
        ('A', ['B', 'C', 'D']),
        ('B', ['C']),
        ('D', ['C']),
    ]

为了简化示例，我们不使用网址表示，但当然url是可以用于网站图:

.. image:: _images/site_04.png
   :width: 400px
   :height: 120px

::

    site = [
        ('http://example.com', ['http://example.com/anotherpage', 'http://othersite.com']),
    ]


使用 Graph Manager
=======================

一旦我们将我们的站点定义为一个图表，我们就可以开始使用 Graph Manager 了。

我们必须先创建我们的图表管理器 ::

    >>> from frontera.utils import graphs
    >>> g = graphs.Manager()

使用 `add_site` 方法添加网站 ::

    >>> site = [('A', ['B', 'C', 'D'])]
    >>> g.add_site(site)

这个管理器现在被初始化并准备好使用。

我们可以得到图表中的所有页面 ::

    >>> g.pages
    [<1:A*>, <2:B>, <3:C>, <4:D>]

星号表示页面是种子，如果我们想要获取站点图形的种子 ::

    >>> g.seeds
    [<1:A*>]

我们可以用 `get_page` 获取一个单独页面, 如果页面不存在就返回 None

    >>> g.get_page('A')
    <1:A*>

    >>> g.get_page('F')
    None

CrawlPage 对象
=================
页面使用 :class:`CrawlPage` 对象表示：


.. class:: CrawlPage()

   :class:`CrawlPage` 对象表示一个 Graph Manager 页面，且该页面通常在 Graph Manager 生成。

    .. attribute:: id

            自动页面 id。

    .. attribute:: url

             页面 url。

    .. attribute:: status

            代表页面状态码。

    .. attribute:: is_seed

            布尔值表示页面是不是种子页面

    .. attribute:: links

            当前页面链接到的页面列表。

    .. attribute:: referers

            链接到当前页面的页面列表。





例子::

    >>> p = g.get_page('A')
    >>> p.id
    1

    >>> p.url
    u'A'

    >>> p.status  # defaults to 200
    u'200'

    >>> p.is_seed
    True

    >>> p.links
    [<2:B>, <3:C>, <4:D>]

    >>> p.referers  # No referers for A
    []

    >>> g.get_page('B').referers  # referers for B
    [<1:A*>]


添加页面和链接
======================
网站图也可以定义为单独添加页面和链接，我们可以用这种方式定义相同的图形::

    >>> g = graphs.Manager()
    >>> a = g.add_page(url='A', is_seed=True)
    >>> b = g.add_link(page=a, url='B')
    >>> c = g.add_link(page=a, url='C')
    >>> d = g.add_link(page=a, url='D')

`add_page` 和 `add_link` 可以随时和 `add_site` 配合使用::


    >>> site = [('A', ['B', 'C', 'D'])]
    >>> g = graphs.Manager()
    >>> g.add_site(site)
    >>> d = g.get_page('D')
    >>> g.add_link(d, 'E')

添加多个网站
=====================

多个网站可以加入管理器::

    >>> site1 = [('A1', ['B1', 'C1', 'D1'])]
    >>> site2 = [('A2', ['B2', 'C2', 'D2'])]

    >>> g = graphs.Manager()
    >>> g.add_site(site1)
    >>> g.add_site(site2)

    >>> g.pages
    [<1:A1*>, <2:B1>, <3:C1>, <4:D1>, <5:A2*>, <6:B2>, <7:C2>, <8:D2>]

    >>> g.seeds
    [<1:A1*>, <5:A2*>]

或者使用 `add_site_list` 方法 ::

    >>> site_list = [
        [('A1', ['B1', 'C1', 'D1'])],
        [('A2', ['B2', 'C2', 'D2'])],
    ]
    >>> g = graphs.Manager()
    >>> g.add_site_list(site_list)


.. _graph-manager-database:

Graphs 数据库
===============

Graph Manager 使用 `SQLAlchemy`_ 来存储和重现图。

默认使用 SQLite 内存模式当成数据库引擎，但是 `any databases supported by SQLAlchemy`_ 中提到的都可以支持。

使用 SQLite 的例子 ::

    >>> g = graphs.Manager(engine='sqlite:///graph.db')

默认情况下，每个新添加都会进行更改，稍后可以加载图 ::

    >>> graph = graphs.Manager(engine='sqlite:///graph.db')
    >>> graph.add_site(('A', []))

    >>> another_graph = graphs.Manager(engine='sqlite:///graph.db')
    >>> another_graph.pages
    [<1:A1*>]

可以使用 `clear_content` 参数重置数据库 ::

    >>> g = graphs.Manager(engine='sqlite:///graph.db', clear_content=True)

使用状态码
==============================

为了使用图重新创建/模拟爬虫，可以为每个页面定义HTTP响应代码。

一个404错误的例子 ::

    >>> g = graphs.Manager()
    >>> g.add_page(url='A', status=404)

可以使用元组列表以下列方式为站点定义状态代码::

    >>> site_with_status_codes = [
        ((200, "A"), ["B", "C"]),
        ((404, "B"), ["D", "E"]),
        ((500, "C"), ["F", "G"]),
    ]
    >>> g = graphs.Manager()
    >>> g.add_site(site_with_status_codes)


新页面的默认状态码值为200。


一个简单的伪造爬虫例子
=============================

使用 :doc:`Frontier Tester tool <frontier-tester>` 能更好的完成 Frontier 的测试，但这里就是一个例子，说明如何伪造 frontier::

    from frontera import FrontierManager, Request, Response
    from frontera.utils import graphs

    if __name__ == '__main__':
        # Load graph from existing database
        graph = graphs.Manager('sqlite:///graph.db')

        # Create frontier from default settings
        frontier = FrontierManager.from_settings()

        # Create and add seeds
        seeds = [Request(seed.url) for seed in graph.seeds]
        frontier.add_seeds(seeds)

        # Get next requests
        next_requets = frontier.get_next_requests()

        # Crawl pages
        while (next_requests):
            for request in next_requests:

                # Fake page crawling
                crawled_page = graph.get_page(request.url)

                # Create response
                response = Response(url=crawled_page.url, status_code=crawled_page.status)

                # Update Page
                page = frontier.page_crawled(response=response
                                             links=[link.url for link in crawled_page.links])
                # Get next requests
                next_requets = frontier.get_next_requests()




渲染图
================

图可以被渲染成 png 文件::

    >>> g.render(filename='graph.png', label='A simple Graph')

渲染图使用的是 `pydot`_,  `Graphviz`_'s 点语言的 python 接口。

如何使用
=============

Graph Manager 可以结合 :doc:`Frontier Tester <frontier-tester>` 测试 frontiers，也可以结合 :doc:`Scrapy Recordings <scrapy-recorder>` 。

.. _SQLAlchemy: http://www.sqlalchemy.org/
.. _any databases supported by SQLAlchemy: http://docs.sqlalchemy.org/en/rel_0_9/dialects/index.html
.. _pydot: https://code.google.com/p/pydot/
.. _Graphviz: http://www.graphviz.org/
