.. _canonical-url-solver:

====================
Canonical URL Solver
规范 URL 解算器
====================

规范 URL 解算器 是一种特殊的 :ref:`middleware <frontier-middlewares>` 对象，用来识别网页的规范 URL，并根据这个修改 request 或者 response 的元数据。通常规范 URL 解算器是在调用后端方法之前最后一个执行的 middleware。

此组件的主要目的是防止元数据记录重复和混淆与其相关联的抓取器行为。原因是： 
- 不同的重定向链将指向相同的文档。 
- 同一份文件可以通过多个不同的URL访问。

精心设计的系统具有自己的稳定算法，为每个文档选择正确的 URL。另见 `Canonical link element`_ 。

规范 URL 解算器在Frontera Manager初始化期间使用 :setting:`CANONICAL_SOLVER` 设置中的类来实例化。


内置规范 URL 解算器参考
========================================

基本的
-----
用作默认值。

.. autoclass:: frontera.contrib.canonicalsolvers.basic.BasicCanonicalSolver


.. _Canonical link element: https://en.wikipedia.org/wiki/Canonical_link_element#Purpose