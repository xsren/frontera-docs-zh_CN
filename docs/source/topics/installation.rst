==================
安装指南
==================

下面的步骤假定您已经安装了以下必需的软件:

* `Python`_ 2.7+ 或 3.4+

* `pip`_ 和 `setuptools`_ Python 包。 

你可以使用 pip 安装 Frontera.

使用 pip 安装::

   pip install frontera[option1,option2,...optionN]

选项
=======
Each option installs dependencies needed for particular functionality.
每个选项安装所需的特定功能的依赖。

* *sql* - 关系型数据库,
* *graphs* - Graph Manager,
* *logging* - 彩色日志,
* *tldextract* - 可以使用 :setting:`TLDEXTRACT_DOMAIN_INFO`
* *hbase* - HBase 分布式后端,
* *zeromq* - ZeroMQ 消息总线,
* *kafka* - Kafka 消息总线,
* *distributed* - workers 依赖.

.. _Python: http://www.python.org
.. _pip: http://www.pip-installer.org/en/latest/installing.html
.. _setuptools: https://pypi.python.org/pypi/setuptools
