========
术语表
========


.. glossary::
    spider log
        来自爬虫的编码消息流。每个消息都是从文档中提取，通常是链接，分数，分类结果。

    scoring log
        从 strategy worker 到 db worker，包含更新事件和调度标志（如果链接需要调度下载）的分数。

    spider feed
        从 :term:`db worker` 到爬虫，包含新的一批需要抓取的文档。

    strategy worker
        一种特殊的 worker，运行抓取策略代码：为链接评分，决定链接是否需要被调度（查询 :term:`state cache` ）和合适停止抓取。这种 worker 是分片的。

    db worker
        负责和存储数据库通信，主要存储元数据和内容，同时拉取新的任务去下载。

    state cache
        内存数据结构，包含文档是否被抓取的状态信息。
        定期与持久存储同步。

    message bus
        传输层抽象机制。提供传输层抽象和实现的接口。

    spider
        从 Web 检索和提取内容，使用 :term:`spider feed` 作为输入队列，将结果存储到 :term:`spider log` 。在这个文档中，提取器被用作同义词。