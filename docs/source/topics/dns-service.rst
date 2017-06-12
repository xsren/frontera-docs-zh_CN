===========
DNS 服务
===========

除了提到的 :ref:`basic_requirements` 你可能还需要一个专用的 DNS 服务。特别是在你的爬虫会产生大量 DNS 请求的情况下。在广度优先抓取或者其他短时间内访问大量网站的情况下，使用专用的 DNS 服务都是正确的。

由于负载巨大，DNS 服务最终可能会被您的网络提供商阻止。

DNS策略有两种选择：

* 递归DNS解析，
* 利用上游服务器（大规模的DNS缓存像OpenDNS或Verizon）。

第二个仍然容易阻塞。

NLnet 实验室发布的 DNS 服务软件很好用 https://www.unbound.net/ 。它允许选择上述策略之一，并维护本地DNS缓存。

请查看 Scrapy 选项 ``REACTOR_THREADPOOL_MAXSIZE`` 和 ``DNS_TIMEOUT`` 。