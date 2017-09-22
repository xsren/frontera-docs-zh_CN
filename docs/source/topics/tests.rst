=====
Tests
=====

Frontera 测试使用 `pytest`_ 工具实现。

您可以使用pip安装 `pytest`_ 和测试中使用的附加必需库::

    pip install -r requirements/tests.txt


运行 tests
=============

要运行所有测试，请转到源代码的根目录并运行::

    py.test


写 tests
=============

所有功能（包括新功能和错误修复）都必须包含一个测试用例，以检查它是否按预期工作，所以如果你希望他们能够早点上线，请尽快写好相关的测试。


后端 testing
===============

有一个继承 `pytest`_ 的类用来测试 :class:`Backend <frontera.core.components.Backend>`:
:class:`BackendTest <tests.backends.BackendTest>`

.. autoclass:: tests.backends.BackendTest

    .. automethod:: tests.backends.BackendTest.get_settings
    .. automethod:: tests.backends.BackendTest.get_frontier
    .. automethod:: tests.backends.BackendTest.setup_backend
    .. automethod:: tests.backends.BackendTest.teardown_backend


比方说，你想测试你的后端  ``MyBackend`` 并为每个测试方法调用创建一个新的 frontier 实例，你可以定义一个这样的测试类::


    class TestMyBackend(backends.BackendTest):

        backend_class = 'frontera.contrib.backend.abackend.MyBackend'

        def test_one(self):
            frontier = self.get_frontier()
            ...

        def test_two(self):
            frontier = self.get_frontier()
            ...

        ...


如果它使用一个数据库文件，你需要在每次测试之前和之后进行清理::


    class TestMyBackend(backends.BackendTest):

        backend_class = 'frontera.contrib.backend.abackend.MyBackend'

        def setup_backend(self, method):
            self._delete_test_db()

        def teardown_backend(self, method):
            self._delete_test_db()

        def _delete_test_db(self):
            try:
                os.remove('mytestdb.db')
            except OSError:
                pass

        def test_one(self):
            frontier = self.get_frontier()
            ...

        def test_two(self):
            frontier = self.get_frontier()
            ...

        ...


测试后端执行顺序
=========================

为了测试 :class:`Backend <frontera.core.components.Backend>` 抓取顺序你可以使用 :class:`BackendSequenceTest <tests.backends.BackendSequenceTest>` 类。

.. autoclass:: tests.backends.BackendSequenceTest

    .. automethod:: tests.backends.BackendSequenceTest.get_sequence
    .. automethod:: tests.backends.BackendSequenceTest.assert_sequence


:class:`BackendSequenceTest <tests.backends.BackendSequenceTest>` 将依据传过来的网站图进行一遍完整的抓取，并返回后端访问网页的顺序。

比如你想测试一个按照字母顺序抓取网页的后端。你可以这样写测试::


    class TestAlphabeticSortBackend(backends.BackendSequenceTest):

        backend_class = 'frontera.contrib.backend.abackend.AlphabeticSortBackend'

        SITE_LIST = [
            [
                ('C', []),
                ('B', []),
                ('A', []),
            ],
        ]

        def test_one(self):
            # Check sequence is the expected one
            self.assert_sequence(site_list=self.SITE_LIST,
                                 expected_sequence=['A', 'B', 'C'],
                                 max_next_requests=0)

        def test_two(self):
            # Get sequence and work with it
            sequence = self.get_sequence(site_list=SITE_LIST,
                                max_next_requests=0)
            assert len(sequence) > 2

        ...


测试基本算法
========================

如果你的后端使用 :ref:`基本算法逻辑 <frontier-backends-basic-algorithms>` 中的一个，你可以继承对应的测试类，之后顺序会被自动测试::

    from tests import backends


    class TestMyBackendFIFO(backends.FIFOBackendTest):
        backend_class = 'frontera.contrib.backends.abackend.MyBackendFIFO'


    class TestMyBackendLIFO(backends.LIFOBackendTest):
        backend_class = 'frontera.contrib.backends.abackend.MyBackendLIFO'


    class TestMyBackendDFS(backends.DFSBackendTest):
        backend_class = 'frontera.contrib.backends.abackend.MyBackendDFS'


    class TestMyBackendBFS(backends.BFSBackendTest):
        backend_class = 'frontera.contrib.backends.abackend.MyBackendBFS'


    class TestMyBackendRANDOM(backends.RANDOMBackendTest):
        backend_class = 'frontera.contrib.backends.abackend.MyBackendRANDOM'



.. _pytest: http://pytest.org/latest/

