================================
Frontier + Requests
================================

为了结合 frontier 和 `Requests`_ ，提供了 ``RequestsFrontierManager`` 类。

这个类是一个简单的 :class:`FrontierManager <frontera.core.manager.FrontierManager>` 封装，它使用 `Requests`_ 对象 (``Request``/``Response``)，将他们和 frontier 相互转换。


和 :class:`FrontierManager <frontera.core.manager.FrontierManager>` 一样使用，使用你的 settings 初始化它。 ``get_next_requests`` 将返回 `Requests`_ ``Request`` 对象。

一个例子::

    import re

    import requests

    from urlparse import urljoin

    from frontera.contrib.requests.manager import RequestsFrontierManager
    from frontera import Settings

    SETTINGS = Settings()
    SETTINGS.BACKEND = 'frontera.contrib.backends.memory.FIFO'
    SETTINGS.LOGGING_MANAGER_ENABLED = True
    SETTINGS.LOGGING_BACKEND_ENABLED = True
    SETTINGS.MAX_REQUESTS = 100
    SETTINGS.MAX_NEXT_REQUESTS = 10

    SEEDS = [
        'http://www.imdb.com',
    ]

    LINK_RE = re.compile(r'href="(.*?)"')


    def extract_page_links(response):
        return [urljoin(response.url, link) for link in LINK_RE.findall(response.text)]

    if __name__ == '__main__':

        frontier = RequestsFrontierManager(SETTINGS)
        frontier.add_seeds([requests.Request(url=url) for url in SEEDS])
        while True:
            next_requests = frontier.get_next_requests()
            if not next_requests:
                break
            for request in next_requests:
                    try:
                        response = requests.get(request.url)
                        links = [requests.Request(url=url) for url in extract_page_links(response)]
                        frontier.page_crawled(response=response)
                        frontier.links_extracted(request=request, links=links)
                    except requests.RequestException, e:
                        error_code = type(e).__name__
                        frontier.request_error(request, error_code)


.. _Requests: http://docs.python-requests.org/en/latest/
