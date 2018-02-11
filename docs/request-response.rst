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

		
		当某个站点返回cookie（在响应中）时，这些cookie将存储在该域的cookie中，并将在未来的请求中再次发送。 这是任何常规Web浏览器的典型行为。 但是，如果出于某种原因想要避免与现有Cookie合并，可以通过在:attr:`Request.meta` ``dont_merge_cookies``项设置为True来指示Scrapy执行此操作。

        不合并Cookie的请求示例::

            request_with_cookies = Request(url="http://www.example.com",
                                           cookies={'currency': 'USD', 'country': 'UY'},
                                           meta={'dont_merge_cookies': True})

        欲了解更多信息，请参阅 :ref:`cookies-mw`.
    :type cookies: dict 或 list

    :param encoding: 此请求的编码（默认为``'utf-8'`` ）。 此编码将用于百分比编码URL并将主体转换为``str`` （如果作为``unicode`` ）。
	   
    :type encoding: string

    :param priority: 此请求的优先级（默认为``0`` ）。 调度程序使用优先级来定义用于处理请求的顺序。 具有较高优先级值的请求将在较早时间执行。 为了表示相对低的优先级，允许负值。
    :type priority: int

    :param dont_filter: 表示这个请求不应该被调度器过滤。 当您想多次执行相同的请求时使用此选项，以忽略重复的过滤器。 小心使用它，否则你将进入爬行循环。 默认为``False`` 。
    :type dont_filter: boolean

    :param errback: 如果在处理请求时引发任何异常，将会调用该函数。 这包括404 HTTP错误等失败的页面。 它收到`Twisted Failure`_实例作为第一个参数。 有关更多信息，请参阅下面的:ref:`topics-request-response-ref-errbacks` 。
    :type errback: callable

    :param flags:  发送到请求的标志可用于日志记录或类似目的。
    :type flags: list

    .. attribute:: Request.url

		
		包含此请求的URL的字符串。 请记住，此属性包含转义的URL，因此它可能与构造函数中传递的URL不同。

		该属性是只读的。 要更改请求的URL，请使用:meth:`replace`。

    .. attribute:: Request.method

		表示请求中的HTTP方法的字符串。 这保证是大写的。 例如： ``"GET"``, ``"POST"``, ``"PUT"``等

    .. attribute:: Request.headers

        一个包含请求头文件的类似字典的对象。

    .. attribute:: Request.body

        包含请求主体的str。

		该属性是只读的。 要更改请求的主体，请使用:meth:`replace`。

    .. attribute:: Request.meta

	
	包含此请求的任意元数据的字典。 这个词典对于新的请求是空的，并且通常由不同的Scrapy组件（扩展，中间件等）填充。 因此，此字典中包含的数据取决于您启用的扩展。

有关由Scrapy识别的特殊元键列表，请参阅:ref:`topics-request-meta` 。

当使用``copy()``或``replace()``方法克隆请求时，此词典被`浅拷贝`_ ，并且也可以在你的蜘蛛中从``response.meta``属性访问。
.. _浅拷贝: https://docs.python.org/2/library/copy.html

    .. method:: Request.copy()

		返回一个新请求，它是此请求的副本。 另请参阅：
       :ref:`topics-request-response-ref-request-callback-arguments`.

    .. method:: Request.replace([url, method, headers, body, cookies, meta, encoding, dont_filter, callback, errback])

		使用相同的成员返回Request对象，但通过指定任何关键字参数给予新值的成员除外。 :attr:`Request.meta`属性默认被复制（除非在``meta``参数中给出一个新值）。 另请参见
       :ref:`topics-request-response-ref-request-callback-arguments`。

.. _topics-request-response-ref-request-callback-arguments:

将附加数据传递给回调函数
---------------------------------------------

请求的回调函数将在下载该请求的响应时调用。 回调函数将以下载的:class:`Response`对象作为第一个参数来调用。

例::

    def parse_page1(self, response):
        return scrapy.Request("http://www.example.com/some_page.html",
                              callback=self.parse_page2)

    def parse_page2(self, response):
        # this would log http://www.example.com/some_page.html
        self.logger.info("Visited %s", response.url)

在某些情况下，您可能有兴趣将参数传递给这些回调函数，以便稍后在第二个回调函数中接收参数。 您可以使用:attr:`Request.meta`属性。

以下是如何使用此机制传递项目以从不同页面填充不同字段的示例::


    def parse_page1(self, response):
        item = MyItem()
        item['main_url'] = response.url
        request = scrapy.Request("http://www.example.com/some_page.html",
                                 callback=self.parse_page2)
        request.meta['item'] = item
        yield request

    def parse_page2(self, response):
        item = response.meta['item']
        item['other_url'] = response.url
        yield item


.. _topics-request-response-ref-errbacks:

使用errbacks在请求处理中捕获异常
--------------------------------------------------------

请求的错误是一个函数，当处理异常时会调用它。

它接收`Twisted Failure`_实例作为第一个参数，可用于跟踪连接建立超时，DNS错误等。

以下是一个蜘蛛日志记录所有错误并在需要时捕获一些特定错误的示例::

    import scrapy

    from scrapy.spidermiddlewares.httperror import HttpError
    from twisted.internet.error import DNSLookupError
    from twisted.internet.error import TimeoutError, TCPTimedOutError

    class ErrbackSpider(scrapy.Spider):
        name = "errback_example"
        start_urls = [
            "http://www.httpbin.org/",              # HTTP 200 expected
            "http://www.httpbin.org/status/404",    # Not found error
            "http://www.httpbin.org/status/500",    # server issue
            "http://www.httpbin.org:12345/",        # non-responding host, timeout expected
            "http://www.httphttpbinbin.org/",       # DNS error expected
        ]

        def start_requests(self):
            for u in self.start_urls:
                yield scrapy.Request(u, callback=self.parse_httpbin,
                                        errback=self.errback_httpbin,
                                        dont_filter=True)

        def parse_httpbin(self, response):
            self.logger.info('Got successful response from {}'.format(response.url))
            # do something useful here...

        def errback_httpbin(self, failure):
            # log all failures
            self.logger.error(repr(failure))

            # in case you want to do something special for some errors,
            # you may need the failure's type:

            if failure.check(HttpError):
                # these exceptions come from HttpError spider middleware
                # you can get the non-200 response
                response = failure.value.response
                self.logger.error('HttpError on %s', response.url)

            elif failure.check(DNSLookupError):
                # this is the original request
                request = failure.request
                self.logger.error('DNSLookupError on %s', request.url)

            elif failure.check(TimeoutError, TCPTimedOutError):
                request = failure.request
                self.logger.error('TimeoutError on %s', request.url)

.. _topics-request-meta:

Request.meta 特殊键
=========================

:attr:`Request.meta`属性可以包含任何任意数据，但Scrapy和它的内置扩展可以识别一些特殊的键。

那些是：

* :reqmeta:`dont_redirect`
* :reqmeta:`dont_retry`
* :reqmeta:`handle_httpstatus_list`
* :reqmeta:`handle_httpstatus_all`
* ``dont_merge_cookies`` (请参阅``cookies`` 参数 :class:`Request` 构造函数的)
* :reqmeta:`cookiejar`
* :reqmeta:`dont_cache`
* :reqmeta:`redirect_urls`
* :reqmeta:`bindaddress`
* :reqmeta:`dont_obey_robotstxt`
* :reqmeta:`download_timeout`
* :reqmeta:`download_maxsize`
* :reqmeta:`download_latency`
* :reqmeta:`download_fail_on_dataloss`
* :reqmeta:`proxy`
* ``ftp_user`` (请参阅 :setting:`FTP_USER` 了解更多信息)
* ``ftp_password`` (请参阅 :setting:`FTP_PASSWORD` 了解更多信息)
* :reqmeta:`referrer_policy`
* :reqmeta:`max_retry_times`

.. reqmeta:: bindaddress

bindaddress
-----------

用于执行请求的传出IP地址的IP。

.. reqmeta:: download_timeout

download_timeout
----------------

下载器在超时之前等待的时间（以秒为单位）。 另请参阅: :setting:`DOWNLOAD_TIMEOUT`.

.. reqmeta:: download_latency

download_latency
----------------

从请求开始以来（即通过网络发送的HTTP消息）获取响应所花费的时间量。 这个元键只有在响应被下载后才可用。 虽然大多数其他元键用于控制Scrapy行为，但它应该是只读的。

.. reqmeta:: download_fail_on_dataloss

download_fail_on_dataloss
-------------------------

是否对失败的反应失败。 请参阅:
:setting:`DOWNLOAD_FAIL_ON_DATALOSS`.

.. reqmeta:: max_retry_times

max_retry_times
---------------

元密钥用于设置每个请求的重试次数。 初始化时，
:reqmeta:`max_retry_times` 元键优先于
:setting:`RETRY_TIMES` 设置。

.. _topics-request-response-ref-request-subclasses:

请求子类
==================


这是内置的:class:`Request`子类的列表。 您也可以将其子类化以实现您自己的自定义功能。

FormRequest 对象
-------------------

FormRequest类使用用于处理HTML表单的功能来扩展基本:class:`Request` 。 它使用`lxml.html forms`_预先填充来自:class:`Response`对象的表单数据的表单字段。

.. _lxml.html forms: http://lxml.de/lxmlhtml.html#forms

.. class:: FormRequest(url, [formdata, ...])

	:class:`FormRequest`类为构造函数添加一个新参数。 其余参数与:class:`Request`类相同，这里没有记录。

    :param formdata: is a dictionary (or iterable of (key, value) tuples)
    是包含HTML表单数据的字典（或（键，值）元组的迭代器），这些数据将被URL编码并分配给请求的主体。
    :type formdata: dict or iterable of tuples

	除了标准的:class:`Request`方法外， :class:`FormRequest`对象还支持以下类方法：

    .. classmethod:: FormRequest.from_response(response, [formname=None, formid=None, formnumber=0, formdata=None, formxpath=None, formcss=None, clickdata=None, dont_click=False, ...])

	   返回一个新的:class:`FormRequest`对象，其表单字段值预先填充到给定响应中包含的HTML:class:`FormRequest`元素中。 有关示例，请参阅:ref:`topics-request-response-ref-request-userlogin` 。

	   该策略默认情况下会自动模拟任何可点击的窗体控件上的点击，如``<input type="submit">``。 尽管这很方便，而且通常是所需的行为，但有时它可能会导致难以调试的问题。 例如，在处理使用javascript填充和/或提交的表单时，默认的:meth:`from_response`行为可能不是最合适的。 要禁用此行为，可以将``dont_click``参数设置为``True`` 。 另外，如果你想改变点击的控件（而不是禁用它），你也可以使用``clickdata``参数。

       .. caution:: 
		  由于lxml中存在一个`bug in lxml`_，在选项值中使用具有前导空白或尾随空白的select元素使用此方法将不起作用，这应该在lxml 3.8及更高版本中修复。

       :param response: 包含将用于预填充表单域的HTML表单的响应
       :type response: :class:`Response` 对象

       :param formname: 如果给定，将使用名称属性设置为此值的表单。
       :type formname: string

       :param formid: 如果给定，将使用id属性设置为该值的表单。
       :type formid: string

       :param formxpath: 如果给定，将使用匹配xpath的第一个表单。
       :type formxpath: string

       :param formcss: 如果给出，将使用匹配CSS选择器的第一个表单。
       :type formcss: string

       :param formnumber: 响应包含多个表单时要使用的表单数。 第一个（也是默认值）是 ``0``。
       :type formnumber: integer

       :param formdata: 要在表单数据中覆盖的字段。 如果某个字段已经存在于响应``<form>``元素中，则其值会被此参数中传递的值覆盖。 如果在此参数中传递的值为``None`` ，则该字段将不会包含在请求中，即使它存在于响应``<form>``元素中。
       :type formdata: dict

       :param clickdata:用于查找点击的控件的属性。 如果没有给出，则会提交表单数据模拟点击第一个可点击元素。 除了html属性之外，控件还可以通过其相对于表单内其他可提交输入的从零开始的索引，通过``nr``属性进行标识。
       :type clickdata: dict

       :param dont_click:  如果为True，表单数据将被提交而不需要单击任何元素。
       :type dont_click: boolean

	   这个类方法的其他参数直接传递给:class:`FormRequest`构造函数。

       .. versionadded:: 0.10.3
           ``formname`` 参数.

       .. versionadded:: 0.17
           ``formxpath`` 参数.

       .. versionadded:: 1.1.0
           ``formcss`` 参数.

       .. versionadded:: 1.1.0
           ``formid`` 参数.

请求使用示例
----------------------

使用FormRequest通过HTTP POST发送数据
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


如果你想在你的蜘蛛模拟一个HTML表单POST并发送一些键值字段，你可以像这样返回一个:class:`FormRequest`对象（来自你的蜘蛛）::



   return [FormRequest(url="http://www.example.com/post/action",
                       formdata={'name': 'John Doe', 'age': '27'},
                       callback=self.after_post)]

.. _topics-request-response-ref-request-userlogin:

使用 FormRequest.from_response() 模拟用户登录
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

网站通常通过``<input
type="hidden">``元素（如会话相关数据或身份验证令牌（用于登录页面））提供预填充表单字段。 在抓取时，您需要自动预填这些字段，并仅覆盖其中的几个字段，例如用户名和密码。 您可以使用:meth:`FormRequest.from_response`方法执行此作业。 这是一个使用它的蜘蛛示例::


    import scrapy

    class LoginSpider(scrapy.Spider):
        name = 'example.com'
        start_urls = ['http://www.example.com/users/login.php']

        def parse(self, response):
            return scrapy.FormRequest.from_response(
                response,
                formdata={'username': 'john', 'password': 'secret'},
                callback=self.after_login
            )

        def after_login(self, response):
            # check login succeed before going on
            if "authentication failed" in response.body:
                self.logger.error("Login failed")
                return

            # continue scraping with authenticated session...


响应对象
================

.. class:: Response(url, [status=200, headers=None, body=b'', flags=None, request=None])

	一个:class:`Response`对象表示一个HTTP响应，它通常被下载（通过Downloader）并且被提供给蜘蛛进行处理。

    :param url: 此响应的URL
    :type url: string

    :param status: 响应的HTTP状态。 默认为 ``200``.
    :type status: integer

    :param headers:  这个响应的标题。 字典值可以是字符串（对于单值标题）或列表（对于多值标题）。
    :type headers: dict

    :param body: 响应主体。 要以str（Python 2中的unicode）的形式访问解码文本，可以使用``response.text``等具有编码意识的:ref:`Response subclass <topics-request-response-ref-response-subclasses>`子类中的:class:`TextResponse` 。
    :type body: bytes

    :param flags: 是一个包含:attr:`Response.flags`属性初始值的列表。 如果给出，列表将被浅拷贝。
    :type flags: list

    :param request: :attr:`Response.request`属性的初始值。 这表示生成此响应的:class:`Request` 。
    :type request: :class:`Request` 对象

    .. attribute:: Response.url

		一个包含响应URL的字符串。

		该属性是只读的。 要更改Response的URL，请使用:meth:`replace` 。	

    .. attribute:: Response.status

		表示响应的HTTP状态的整数。 例如：``200``,``404``。

    .. attribute:: Response.headers

		一个包含响应头文件的类似字典的对象。 可以使用:meth:`get`访问值，以返回具有指定名称的第一个标头值或:meth:`getlist`返回具有指定名称的所有标头值。 例如，这个调用会为您提供标题中的所有Cookie::

            response.headers.getlist('Set-Cookie')

    .. attribute:: Response.body

		这个响应的主体。 请记住，Response.body始终是一个字节对象。 如果您希望unicode版本使用:attr:`TextResponse.text` （仅在:class:`TextResponse`和子类中可用）。

		该属性是只读的。 要更改Response的主体，请使用:meth:`replace` 。

    .. attribute:: Response.request

		生成此响应的:class:`Request`对象。 在响应和请求已经通过所有:ref:`Downloader Middlewares <topics-downloader-middleware>`之后，在Scrapy引擎中分配此属性。 特别是，这意味着：

        - HTTP重定向会将原始请求（重定向前的URL）分配给重定向的响应（重定向后使用最终的URL）。

        - Response.request.url 并不总是等于 Response.url

        - 该属性仅在spider代码中，在:ref:`Spider Middlewares <topics-spider-middleware>`中可用，但在Downloader中间件中不可用（尽管您可以通过其他方式获得请求）以及:signal:`response_downloaded`信号的处理程序。

    .. attribute:: Response.meta

		
		:attr:`Response.request`对象的:attr:`Request.meta`属性（即``self.request.meta``）的快捷方式。

		与:attr:`Response.request`属性不同， :attr:`Response.request`属性沿着重定向和重试传播，因此您将获得从蜘蛛中发送的原始:attr:`Response.meta` 。

        .. seealso:: :attr:`Request.meta` 属性

    .. attribute:: Response.flags

		
		包含此响应标志的列表。 标志是用于标记响应的标签。 例如：`'cached'`, `'redirected`'等。它们显示在引擎用于记录的Response（ `__str__`
        method）的字符串表示中。

    .. method:: Response.copy()

       返回一个新的Response，它是此Response的副本。

    .. method:: Response.replace([url, status, headers, body, request, flags, cls])
	   
	   使用相同的成员返回一个Response对象，除了那些由指定的关键字参数赋予新值的成员。 :attr:`Response.meta` 属性默认被复制。

    .. method:: Response.urljoin(url)

		通过将响应的:attr:`url`与可能的相对URL结合来构造绝对URL。

		这是一个通过`urlparse.urljoin`_的封装，它仅仅是进行此调用的别名::

            urlparse.urljoin(response.url, url)

    .. automethod:: Response.follow


.. _urlparse.urljoin: https://docs.python.org/2/library/urlparse.html#urlparse.urljoin

.. _topics-request-response-ref-response-subclasses:

响应子类
===================

以下是可用的内置Response子类的列表。 您也可以继承Response类来实现您自己的功能。

TextResponse 对象
--------------------

.. class:: TextResponse(url, [encoding[, ...]])

	:class:`TextResponse`对象将编码功能添加到基本:class:`Response`类，该类仅用于二进制数据，例如图像，声音或任何媒体文件。

	除了基本的:class:`Response`对象之外， :class:`TextResponse`对象还支持一个新的构造函数参数。 其余功能与:class:`Response`类相同，在此不作介绍。

    :param encoding: 是包含用于此响应的编码的字符串。 如果使用unicode主体创建一个:class:`TextResponse`对象，它将使用此编码进行编码（请记住，body属性始终是一个字符串）。 如果``encoding``为``None`` （默认值），则将在响应标题和正文中查找编码。
    :type encoding: string

	:class:`TextResponse`对象除了标准的:class:`Response` 对象外，还支持以下属性：

    .. attribute:: TextResponse.text

       响应主体，如unicode。

	   与``response.body.decode(response.encoding)``相同，但结果在第一次调用后被缓存，因此您可以多次访问``response.text``而无需额外开销。

       .. note::

			``unicode(response.body)``不是将响应主体转换为unicode的正确方法：您将使用系统默认编码(typically `ascii`)而不是响应编码。


    .. attribute:: TextResponse.encoding

       一个包含此响应编码的字符串。 通过按以下顺序尝试以下机制来解决编码问题：

       1. 在构造函数编码参数中传递的`encoding`参数

       2.在Content-Type HTTP头中声明的编码。 如果这种编码无效（即未知），它将被忽略，并尝试下一个解析机制。

       3. 在响应正文中声明的编码。 TextResponse类没有为此提供任何特殊功能。 但是， :class:`HtmlResponse`和:class:`XmlResponse`类可以。

       4. 通过查看响应主体来推断编码。 这是更脆弱的方法，但也是最后一个尝试。



    .. attribute:: TextResponse.selector

		使用:class:`~scrapy.selector.Selector`作为目标的Selector实例。 第一次访问时，选择器是懒洋洋地实例化的。


	:class:`TextResponse`对象除了标准的:class:`Response`对象外，还支持以下方法：

    .. method:: TextResponse.xpath(query)

        快捷方式 ``TextResponse.selector.xpath(query)``::

            response.xpath('//p')

    .. method:: TextResponse.css(query)

        快捷方式``TextResponse.selector.css(query)``::

            response.css('p')

    .. automethod:: TextResponse.follow

    .. method:: TextResponse.body_as_unicode()

		与:attr:`text`相同，但作为方法可用。 这种方法保持向后兼容; 请优先选择``response.text``。


HtmlResponse 对象
--------------------

.. class:: HtmlResponse(url[, ...])

	:class:`HtmlResponse`类是:class:`TextResponse`的一个子类，它通过查看HTML `meta
    http-equiv`_属性来添加编码自动发现支持。 请参阅:attr:`TextResponse.encoding`。

.. _meta http-equiv: https://www.w3schools.com/TAGS/att_meta_http_equiv.asp

XmlResponse 对象
-------------------

.. class:: XmlResponse(url[, ...])


	:class:`XmlResponse`类是:class:`TextResponse`的一个子类，它通过查看XML声明行来添加编码自动发现支持。 请参阅:attr:`TextResponse.encoding` 。

.. _Twisted Failure: https://twistedmatrix.com/documents/current/api/twisted.python.failure.Failure.html
.. _bug in lxml: https://bugs.launchpad.net/lxml/+bug/1665241
