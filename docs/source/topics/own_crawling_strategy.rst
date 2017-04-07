=================
抓取策略
=================

使用 ``cluster`` 例子和 ``frontera.worker.strategies.bfs`` 模型进行参考。
Use ``cluster`` example and ``frontera.worker.strategies.bfs`` module for reference. 一般来说，你需要写一个
抓取策略子类，参照:

.. autoclass:: frontera.worker.strategies.BaseCrawlingStrategy

    **Methods**

    .. automethod:: frontera.worker.strategies.BaseCrawlingStrategy.from_worker
    .. automethod:: frontera.worker.strategies.BaseCrawlingStrategy.add_seeds
    .. automethod:: frontera.worker.strategies.BaseCrawlingStrategy.page_crawled
    .. automethod:: frontera.worker.strategies.BaseCrawlingStrategy.page_error
    .. automethod:: frontera.worker.strategies.BaseCrawlingStrategy.finished
    .. automethod:: frontera.worker.strategies.BaseCrawlingStrategy.close


该类可以放在任何模块中，并在启动时使用命令行选项或 :setting:`CRAWLING_STRATEGY` 设置传递给 :term:`strategy worker`。

这个策略类在 strategy worker 中实例化，可以使用自己的存储或任何其他类型的资源。所有来着 :term:`spider log` 的都会传给这些方法。返回的分数不一定与方法参数相同。``finished()`` 方法会被周期性的调用来检测抓取目标是否达到了。
