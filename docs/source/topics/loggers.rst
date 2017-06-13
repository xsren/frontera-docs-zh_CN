=======
Logging
=======

Frontera 使用 Python 原生日志系统。这允许用户通过配置文件（参考 :setting:`LOGGING_CONFIG` ）或者在运行时配置 logger。

Logger 配置语法在这里
https://docs.python.org/2/library/logging.config.html

使用的Loggers
============

* kafka
* hbase.backend
* hbase.states
* hbase.queue
* sqlalchemy.revisiting.queue
* sqlalchemy.metadata
* sqlalchemy.states
* sqlalchemy.queue
* offset-fetcher
* messagebus-backend
* cf-server
* db-worker
* strategy-worker
* messagebus.kafka
* memory.queue
* memory.dequequeue
* memory.states
* manager.components
* manager
* frontera.contrib.scrapy.schedulers.FronteraScheduler

