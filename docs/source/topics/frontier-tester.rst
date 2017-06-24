==================
测试一个 Frontier
==================

Frontier Tester是一个帮助类，用于方便 frontier 测试。

基本上它基于 Frontier 运行一个模拟的爬取过程，爬虫信息是使用Graph Manager实例伪造的。

创建一个 Frontier Tester
==========================

FrontierTester 需要一个 :doc:`Graph Manager <graph-manager>` 和一个
:class:`FrontierManager <frontera.core.manager.FrontierManager>` 实例::

    >>> from frontera import FrontierManager, FrontierTester
    >>> from frontera.utils import graphs
    >>> graph = graphs.Manager('sqlite:///graph.db')  # Crawl fake data loading
    >>> frontier = FrontierManager.from_settings()  # Create frontier from default settings
    >>> tester = FrontierTester(frontier, graph)

运行一个 Test
==============

tester 已经被实例化，现在只需调用 `run` 函数运行::

    >>> tester.run()

当 run 方法被调用 tester 将：

    1. 从图中添加所有的种子。
    2. 向 frontier 询问新的任务。
    3. 模拟页面响应，并通知 frontier 关于页面抓取及其链接。

重复步骤1和2，直到抓取或 frontier 结束。

测试完成后，抓取页面 ``sequence`` 可作为 frontier :class:`Request <frontera.core.models.Request>` 对象列表使用。

测试参数
===============

在某些测试用例中，您可能需要将所有页面添加为种子，这可以通过参数 ``add_all_pages`` 来完成::

    >>> tester.run(add_all_pages=True)

每个 :attr:`get_next_requests <frontera.core.manager.FrontierManager.get_next_requests>` 调用的最大返回页数可以使用 frontier settings进行设置，也可创建 FrontierTester 时使用 ``max_next_pages`` 参数进行修改::

    >>> tester = FrontierTester(frontier, graph, max_next_pages=10)


使用的例子
=================

一个使用 graph 测试数据和 :ref:`basic backends <frontier-backends-basic-algorithms>` 的例子 ::

    from frontera import FrontierManager, Settings, FrontierTester, graphs


    def test_backend(backend):
        # Graph
        graph = graphs.Manager()
        graph.add_site_list(graphs.data.SITE_LIST_02)

        # Frontier
        settings = Settings()
        settings.BACKEND = backend
        settings.TEST_MODE = True
        frontier = FrontierManager.from_settings(settings)

        # Tester
        tester = FrontierTester(frontier, graph)
        tester.run(add_all_pages=True)

        # Show crawling sequence
        print '-'*40
        print frontier.backend.name
        print '-'*40
        for page in tester.sequence:
            print page.url

    if __name__ == '__main__':
        test_backend('frontera.contrib.backends.memory.heapq.FIFO')
        test_backend('frontera.contrib.backends.memory.heapq.LIFO')
        test_backend('frontera.contrib.backends.memory.heapq.BFS')
        test_backend('frontera.contrib.backends.memory.heapq.DFS')
