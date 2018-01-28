.. _topics-请求-响应:

======================
请求和响应
======================

.. 模块:: scrapy.http
   :概要: Request 和 Response 类

Scrapy 使用 :class:`Request` 和 :class:`Response` 对象来抓取网站。

通常情况下，:class:`Request`对象是在爬虫中生成，并由程序传到Downloader，执行请求并返回一个:class:`Response`对象，回传到发出请求的爬虫。

:class:`Request`和:class:`Response`类都有子类，它们添加了基类中不需要的功能。 这些在:ref:`topics-request-response-ref-request-subclasses`和:ref:`topics-request-response-ref-response-subclasses`中进行了介绍。


Request 对象
===============

.. class:: Request(url[, callback, method='GET', headers, body, cookies, meta, encoding='utf-8', priority=0, dont_filter=False, errback, flags])

    :class:`Request` 对象代表一个 HTTP 请求, 通常在Spider中生成并由Downloader执行，从而生成 :class:`Response`.

    :param url: 这个请求的 URL 
    :type url: string

    :param callback: 这个函数将会被这个请求的响应（一旦下载完成）调用为第一个参数。 有关更多信息，请参阅下面的:ref:`topics-request-response-ref-request-callback-arguments` 。 如果请求没有指定回调，则将使用spider的:meth:`~scrapy.spiders.Spider.parse`方法。 请注意，如果在处理期间引发异常，则会调用errback。
	   
    :type callback: callable

    :param method:  这个请求的HTTP方法。 默认为 ``'GET'``.
    :type method: string

    :param meta:  :attr:`Request.meta` 属性的初始值。 如果给出，在这个参数传递的字典将被浅拷贝。
    :type meta: dict

    :param body: 
	  请求正文。 如果一个``unicode``被传递，那么它使用传递的`encoding` （默认为``utf-8`` ）编码为``str``。 如果没有给出``body`` ，则存储空字符串。 不管这个参数的类型如何，存储的最终值将是一个``str`` （从不``unicode``或``None`` ）。
    :type body: str 或 unicode

    :param headers: 
	    这个请求的标题。 字典值可以是字符串（对于单值标题）或列表（对于多值标题）。 如果``None``作为值传递，则HTTP头将不会被发送。
    :type headers: dict

    :param cookies: the request cookies. These can be sent in two forms.

        1. 使用字典::

            request_with_cookies = Request(url="http://www.example.com",
                                           cookies={'currency': 'USD', 'country': 'UY'})

        2. 使用一系列的字典::

            request_with_cookies = Request(url="http://www.example.com",
                                           cookies=[{'name': 'currency',
                                                    'value': 'USD',
                                                    'domain': 'example.com',
                                                    'path': '/currency'}])

		后一种形式允许自定义cookie的``domain``和``path``属性。 这只有在cookie被保存用于以后的请求时才有用。

        When some site returns cookies (in a response) those are stored in the
        cookies for that domain and will be sent again in future requests. That's
        the typical behaviour of any regular web browser. However, if, for some
        reason, you want to avoid merging with existing cookies you can instruct
        Scrapy to do so by setting the ``dont_merge_cookies`` key to True in the
        :attr:`Request.meta`.
		
		当某个站点返回cookie（在响应中）时，这些cookie将存储在该域的cookie中，并将在未来的请求中再次发送。 这是任何常规Web浏览器的典型行为。 但是，如果出于某种原因想要避免与现有Cookie合并，可以通过在:attr:`Request.meta` ``dont_merge_cookies``项设置为True来指示Scrapy执行此操作。

        不合并Cookie的请求示例::

            request_with_cookies = Request(url="http://www.example.com",
                                           cookies={'currency': 'USD', 'country': 'UY'},
                                           meta={'dont_merge_cookies': True})

        欲了解更多信息，请参阅 :ref:`cookies-mw`.
    :type cookies: dict 或 list
