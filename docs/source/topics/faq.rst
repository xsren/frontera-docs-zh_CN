======
F.A.Q.
======

.. _efficient-parallel-downloading:

如何并行有效下载？
----------------------------------------

通常，URL排序的设计意味着从同一个域获取许多URL。 如果抓取过程需要有礼貌，则必须保留一些延迟和请求率。 另一方面，下载器同时可以同时下载许多网址（例如100）。 因此，来自同一域的URL的并发导致下载器连接池资源的低效浪费。

这是一个简短的例子。 想像一下，我们有来自许多不同域的10K网址的队列。 我们的任务是尽快取得它。 在下载期间，我们希望对每个主机有礼貌和并限制RPS。 同时，我们有一个优先级，它倾向于对来自同一个域的URL进行分组。 当抓取工具正在请求批量的URL以获取时，将从同一主机获取数百个URL。 由于RPS限制和延迟，下载器将无法快速获取它们。 因此，从队列中挑选统一域的URL可以让我们浪费时间，因为下载器的连接池浪费了大部分时间。

解决方案是在下载程序中为 Frontera 后端提供主机名/ ip（通常但不是必需的）使用。 我们在方法 :attr:`get_next_requests <frontera.core.components.Backend.get_next_requests>` 中有一个关键字参数，用于传递这些统计信息到 Fronter a后端。 任何类型的信息都可以通过。 这个参数通常设置在 Frontera 之外，然后通过 :class:`FrontierManagerWrapper <frontera.utils.managers.FrontierManagerWrapper>` 子类传递给CF到后端。