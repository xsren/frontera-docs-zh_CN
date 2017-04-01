===================
集群安装指南
===================

这个指南的目标是教你如何初始化爬虫集群，在实践过程中一些步骤可能需要微调。这篇指南假设你使用 Kafka作为消息总线（官方推荐），当然用 Zero MQ 也是可以的，但是可靠性会差点。

需要决定的事情

================
* 你抓取的速度，
* 爬虫进程的数量（假设单个爬虫的最大速度是1200个网页/分钟），
* DB worker 和 Strategy worker 的数量。

启动之前需要安装的
================================
* Kafka,
* HBase (推荐 1.0.x 或更高的版本),
* :doc:`DNS Service <dns-service>` (推荐但并不是必须的).

启动之前需要实现的
====================================
* :doc:`Crawling strategy <own_crawling_strategy>`
* 爬虫代码

配置 Kafka
=================
Create all topics needed for Kafka message bus
为 Kafka 消息总线创建所有需要的 topic
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* :term:`spider log` (`frontier-done` (see :setting:`SPIDER_LOG_TOPIC`)), 设置 topic 分区数与 Strategy worker 实例数相同,
* :term:`spider feed` (`frontier-todo` (see :setting:`SPIDER_FEED_TOPIC`)), 设置 topic 分区数与爬虫数相同,
* :term:`scoring log` (`frontier-score` (see :setting:`SCORING_LOG_TOPIC`))


配置 HBase
=================
* 创建一个 namespace ``crawler`` (请参照 :setting:`HBASE_NAMESPACE`),
* 确保原生支持 Snappy 压缩。


配置 Frontera
====================
每个 Frontera 组件需要自己的配置模块，但是一些配置项是共享的，所以我们推荐创建一个公有的配置模块，并在自有的配置中引入这个公有模块。

1. 创建一个公有模块并添加如下信息: ::

    from __future__ import absolute_import
    from frontera.settings.default_settings import MIDDLEWARES
    MAX_NEXT_REQUESTS = 512
    SPIDER_FEED_PARTITIONS = 2 # number of spider processes
    SPIDER_LOG_PARTITIONS = 2 # worker instances
    MIDDLEWARES.extend([
        'frontera.contrib.middlewares.domain.DomainMiddleware',
        'frontera.contrib.middlewares.fingerprint.DomainFingerprintMiddleware'
    ])

    QUEUE_HOSTNAME_PARTITIONING = True
    KAFKA_LOCATION = 'localhost:9092' # your Kafka broker host:port
    SCORING_TOPIC = 'frontier-scoring'
    URL_FINGERPRINT_FUNCTION='frontera.utils.fingerprint.hostname_local_fingerprint'

2. 创建 workers 的公有模块: ::

    from __future__ import absolute_import
    from .common import *

    BACKEND = 'frontera.contrib.backends.hbase.HBaseBackend'

    MAX_NEXT_REQUESTS = 2048
    NEW_BATCH_DELAY = 3.0

    HBASE_THRIFT_HOST = 'localhost' # HBase Thrift server host and port
    HBASE_THRIFT_PORT = 9090

3. 创建 DB worker 配置模块: ::

    from __future__ import absolute_import
    from .worker import *

    LOGGING_CONFIG='logging-db.conf' # if needed

4. 创建 Strategy worker 配置模块: ::

    from __future__ import absolute_import
    from .worker import *

    CRAWLING_STRATEGY = '' # path to the crawling strategy class
    LOGGING_CONFIG='logging-sw.conf' # if needed

logging 配置可参考 https://docs.python.org/2/library/logging.config.html 请看
:doc:`list of loggers <loggers>`.

5. 设置爬虫配置模块: ::

    from __future__ import absolute_import
    from .common import *

    BACKEND = 'frontera.contrib.backends.remote.messagebus.MessageBusBackend'
    KAFKA_GET_TIMEOUT = 0.5


6. 配置 Scrapy settings 模块. 这个模块在 Scrapy 项目文件夹中，并被 scrapy.cfg 引用 。 添加如下::

    FRONTERA_SETTINGS = ''  # module path to your Frontera spider config module

    SCHEDULER = 'frontera.contrib.scrapy.schedulers.frontier.FronteraScheduler'

    SPIDER_MIDDLEWARES = {
        'frontera.contrib.scrapy.middlewares.schedulers.SchedulerSpiderMiddleware': 999,
        'frontera.contrib.scrapy.middlewares.seeds.file.FileSeedLoader': 1,
    }
    DOWNLOADER_MIDDLEWARES = {
        'frontera.contrib.scrapy.middlewares.schedulers.SchedulerDownloaderMiddleware': 999,
    }


启动集群
====================

首先，启动 DB worker: ::

    # start DB worker only for batch generation
    $ python -m frontera.worker.db --config [db worker config module] --no-incoming
    ...
    # Then start next one dedicated to spider log processing
    $ python -m frontera.worker.db --no-batches --config [db worker config module]


之后，启动strategy workers，每个 spider log topic 的分区需要对应一个 strategy workers 的实例: ::

    $ python -m frontera.worker.strategy --config [strategy worker config] --partition-id 0
    $ python -m frontera.worker.strategy --config [strategy worker config] --partition-id 1
    ...
    $ python -m frontera.worker.strategy --config [strategy worker config] --partition-id N

你应该注意到所有的进程会向 log 中写信息。如果没有数据传递相关的 log 信息也是正常的，因为现在系统中还没有种子 URLS。


让我们在文件中每行放一个 URL 作为种子，来启动爬虫。每个爬虫进程对应一个 spider feed topic 的分区: ::

    $ scrapy crawl [spider] -L INFO -s SEEDS_SOURCE = 'seeds.txt' -s SPIDER_PARTITION_ID=0
    ...
    $ scrapy crawl [spider] -L INFO -s SPIDER_PARTITION_ID=1
    $ scrapy crawl [spider] -L INFO -s SPIDER_PARTITION_ID=2
    ...
    $ scrapy crawl [spider] -L INFO -s SPIDER_PARTITION_ID=N

最后你应该启动 N 个爬虫进程。通常一个爬虫实例从 ``SEEDS_SOURCE`` 中读取种子发送给 Frontera 集群就足够了。只有爬虫的任务队列为空时才会读取种子。也可以从配置文件中读取 :setting:`SPIDER_PARTITION_ID` 。

一段时间以后，种子会被准备好，以供爬虫抓取。爬虫真正启动了。
