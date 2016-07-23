:version: 1.0.0-rc1

使用跨域请求 (CORS)
=====================================

By `Mike Wasson`_

浏览器安全阻止一个web页面通过ajax访问另外一个域. 这个限制叫做 *同源策略*, 它阻止一个恶意站点读取另外一个网站的敏感数据. 然而, 有时候你想让别的网站跨域访问你的web程序.

`跨域资源共享 <http://www.w3.org/TR/cors/>`_ (CORS) 是一个W3C 标准, 它允许服务器放宽同源策略. 通过使用 CORS, 服务器可以明确的允许一些跨域请求,也可以拒绝其他请求. CORS 比过去的一些技术更安全,更灵活. 例如 `JSONP <http://en.wikipedia.org/wiki/JSONP>`_. 这个主题将说明如何在你的ASP.NET Core程序中使用CORS.

.. contents:: Sections:
  :local:
  :depth: 1

什么是 "同源"?
----------------------

具有相同的协议,域名,端口的2个Url即为同源. (`RFC 6454 <http://tools.ietf.org/html/rfc6454>`_)

这2个Url属于同源:

- \http://example.com/foo.html
- \http://example.com/bar.html

相对于前面2个Url,这些Url都不是同源:

- \http://example.net - 不同域名
- \http://example.com:9000/foo.html - 不同端口
- \https://example.com/foo.html - 不同的协议
- \http://www.example.com/foo.html - 不同的子域名

注意

IE浏览器判断同源时不比较端口.

启用 CORS
---------------

使用 ``Microsoft.AspNetCore.Cors`` 包来为你的程序启用CORS功能. 在 project.json 文件, 添加以下描述:

.. literalinclude:: cors/sample/src/CorsExample1/project.json
  :language: none
  :lines: 5,6,9
  :emphasize-lines: 2
  
在Startup.cs中添加CORS服务:

.. literalinclude:: cors/sample/src/CorsExample1/Startup.cs
  :language: csharp
  :lines: 9-12
  :dedent: 8

通过中间件的方式启用CORS
-----------------------------

为了使你的整个程序启用CORS,需要使用``UseCors``扩展方法来添加CORS中间件到请求管道中.注意,
To enable CORS for your entire application add the CORS middleware to your request pipeline using the ``UseCors`` extension method. Note that the CORS middleware must proceed any defined endpoints in your app that you want to support cross-origin requests (ex. before any call to ``UseMvc``).

添加CORS中间件时,你可以通过使用``CorsPolicyBuilder``类来指定一个跨域策略.这里有2个方法.第一个方法通过lambda表达式调用UseCors:

.. literalinclude:: cors/sample/src/CorsExample1/Startup.cs
  :language: csharp
  :lines: 15-18, 24
  :dedent: 8

lambda表达式获取一个CorsPolicyBuilder对象. 在这个主题结束后,将描述所有的配置选项.在这个例子中, 策略允许 "\http://example.com" 的跨域请求,其他跨域请求会被拒绝.

注意 CorsPolicyBuilder有一个fluent API, 因此你可以进行链式方法调用:

.. literalinclude:: cors/sample/src/CorsExample3/Startup.cs
  :language: csharp
  :lines: 21-24
  :dedent: 12
  :emphasize-lines: 3

第二个方法,你可以通过命名的方式定义一个或者多个策略,然后在运行的时候通过名字来指定使用的策略.

.. literalinclude:: cors/sample/src/CorsExample2/Startup.cs
  :language: csharp
  :lines: 9-17,19-26,27
  :dedent: 8

这个例子添加了一个名字叫 "AllowSpecificOrigin" CORS策略. 要选择这个策略, 需要把策略的名字传入 UseCors.

.. _cors-policy-options:

在MVC中启用CORS
--------------------

在MVC中,你可以选择性的为每个action,每个controller或者通过全局配置为所有controller来启用CORS. When using MVC to enable CORS the same CORS services are used, but the CORS middleware is not.

针对每个action
^^^^^^^^^^

通过在action添加 ``[EnableCors]`` 属性来为action添加CORS策略. 要指定策略名字.

.. literalinclude:: cors/sample/src/CorsMvc/Controllers/HomeController.cs
    :language: csharp
    :lines: 7-13
    :dedent: 4

针对每个controller
^^^^^^^^^^^^^^

通过在controller类添加 ``[EnableCors]`` 属性来为controller添加CORS策略. 要指定策略名字.

.. literalinclude:: cors/sample/src/CorsMvc/Controllers/HomeController.cs
    :language: csharp
    :lines: 6-8
    :dedent: 4

全局配置
^^^^^^^^

为所有的controller启用CORS，你需要添加 ``CorsAuthorizationFilterFactory`` 过滤器到全局过滤器集合里:

.. literalinclude:: cors/sample/src/CorsMvc/Startup.cs
    :language: csharp
    :lines: 13-15,26-30
    :dedent: 8

The precedence order is: Action, controller, global. Action-level policies take precedence over controller-level policies, and controller-level policies take precedence over global policies.

Disable CORS
^^^^^^^^^^^^

To disable CORS for a controller or action, use the ``[DisableCors]`` attribute.

.. literalinclude:: cors/sample/src/CorsMvc/Controllers/HomeController.cs
    :language: csharp
    :lines: 15-19
    :dedent: 4

CORS policy options
-------------------

This section describes the various options that you can set in a CORS policy.

- `Set the allowed origins`_
- `Set the allowed HTTP methods`_
- `Set the allowed request headers`_
- `Set the exposed response headers`_
- `Credentials in cross-origin requests`_
- `Set the preflight expiration time`_

For some options it may be helpful to read `How CORS works`_ first.

Set the allowed origins
^^^^^^^^^^^^^^^^^^^^^^^

To allow one or more specific origins:

.. literalinclude:: cors/sample/src/CorsExample4/Startup.cs
  :language: csharp
  :start-after: BEGIN01
  :end-before: END01
  :dedent: 16

To allow all origins:

.. literalinclude:: cors/sample/src/CorsExample4/Startup.cs
  :language: csharp
  :start-after: BEGIN02
  :end-before: END02
  :dedent: 16

Consider carefully before allowing requests from any origin. It means that literally any website can make AJAX calls to your app.

Set the allowed HTTP methods
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To specify which HTTP methods are allowed to access the resource.

.. literalinclude:: cors/sample/src/CorsExample4/Startup.cs
  :language: csharp
  :start-after: BEGIN03
  :end-before: END03
  :dedent: 16

To allow all HTTP methods:

.. literalinclude:: cors/sample/src/CorsExample4/Startup.cs
  :language: csharp
  :start-after: BEGIN04
  :end-before: END04
  :dedent: 16

This affects pre-flight requests and Access-Control-Allow-Methods header.

Set the allowed request headers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A CORS preflight request might include an Access-Control-Request-Headers header, listing the HTTP headers set by the application (the so-called "author request headers").

To whitelist specific headers:

.. literalinclude:: cors/sample/src/CorsExample4/Startup.cs
  :language: csharp
  :start-after: BEGIN05
  :end-before: END05
  :dedent: 16

To allow all author request headers:

.. literalinclude:: cors/sample/src/CorsExample4/Startup.cs
  :language: csharp
  :start-after: BEGIN06
  :end-before: END06
  :dedent: 16

Browsers are not entirely consistent in how they set Access-Control-Request-Headers. If you set headers to anything other than "*", you should include at least "accept", "content-type", and "origin", plus any custom headers that you want to support.

Set the exposed response headers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

By default, the browser does not expose all of the response headers to the application. (See http://www.w3.org/TR/cors/#simple-response-header.) The response headers that are available by default are:

- Cache-Control
- Content-Language
- Content-Type
- Expires
- Last-Modified
- Pragma

The CORS spec calls these *simple response headers*. To make other headers available to the application:

.. literalinclude:: cors/sample/src/CorsExample4/Startup.cs
  :language: csharp
  :start-after: BEGIN07
  :end-before: END07
  :dedent: 16

Credentials in cross-origin requests
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Credentials require special handling in a CORS request. By default, the browser does not send any credentials with a cross-origin request. Credentials include cookies as well as HTTP authentication schemes. To send credentials with a cross-origin request, the client must set XMLHttpRequest.withCredentials to true.

Using XMLHttpRequest directly:

.. code-block:: js

    var xhr = new XMLHttpRequest();
    xhr.open('get', 'http://www.example.com/api/test');
    xhr.withCredentials = true;

In jQuery:

.. code-block:: js

    $.ajax({
        type: 'get',
        url: 'http://www.example.com/home',
        xhrFields: {
            withCredentials: true
        }

In addition, the server must allow the credentials. To allow cross-origin credentials:

.. literalinclude:: cors/sample/src/CorsExample4/Startup.cs
  :language: csharp
  :start-after: BEGIN08
  :end-before: END08
  :dedent: 16

Now the HTTP response will include an Access-Control-Allow-Credentials header, which tells the browser that the server allows credentials for a cross-origin request.

If the browser sends credentials, but the response does not include a valid Access-Control-Allow-Credentials header, the browser will not expose the response to the application, and the AJAX request fails.

Be very careful about allowing cross-origin credentials, because it means a website at another domain can send a logged-in user’s credentials to your app on the user’s behalf, without the user being aware. The CORS spec also states that setting origins to "*" (all origins) is invalid if the Access-Control-Allow-Credentials header is present.

Set the preflight expiration time
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Access-Control-Max-Age header specifies how long the response to the preflight request can be cached. To set this header:

.. literalinclude:: cors/sample/src/CorsExample4/Startup.cs
  :language: csharp
  :start-after: BEGIN09
  :end-before: END09
  :dedent: 16

.. _cors-how-cors-works:

How CORS works
--------------

This section describes what happens in a CORS request, at the level of the HTTP messages. It’s important to understand how CORS works, so that you can configure the your CORS policy correctly, and troubleshoot if things don’t work as you expect.

The CORS specification introduces several new HTTP headers that enable cross-origin requests. If a browser supports CORS, it sets these headers automatically for cross-origin requests; you don’t need to do anything special in your JavaScript code.

Here is an example of a cross-origin request. The "Origin" header gives the domain of the site that is making the request::

    GET http://myservice.azurewebsites.net/api/test HTTP/1.1
    Referer: http://myclient.azurewebsites.net/
    Accept: */*
    Accept-Language: en-US
    Origin: http://myclient.azurewebsites.net
    Accept-Encoding: gzip, deflate
    User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; WOW64; Trident/6.0)
    Host: myservice.azurewebsites.net

If the server allows the request, it sets the Access-Control-Allow-Origin header. The value of this header either matches the Origin header, or is the wildcard value "*", meaning that any origin is allowed.::

    HTTP/1.1 200 OK
    Cache-Control: no-cache
    Pragma: no-cache
    Content-Type: text/plain; charset=utf-8
    Access-Control-Allow-Origin: http://myclient.azurewebsites.net
    Date: Wed, 20 May 2015 06:27:30 GMT
    Content-Length: 12

    Test message

If the response does not include the Access-Control-Allow-Origin header, the AJAX request fails. Specifically, the browser disallows the request. Even if the server returns a successful response, the browser does not make the response available to the client application.

Preflight Requests
^^^^^^^^^^^^^^^^^^

For some CORS requests, the browser sends an additional request, called a "preflight request", before it sends the actual request for the resource.
The browser can skip the preflight request if the following conditions are true:

- The request method is GET, HEAD, or POST, and
- The application does not set any request headers other than Accept, Accept-Language, Content-Language, Content-Type, or Last-Event-ID, and
- The Content-Type header (if set) is one of the following:

  - application/x-www-form-urlencoded
  - multipart/form-data
  - text/plain

The rule about request headers applies to headers that the application sets by calling setRequestHeader on the XMLHttpRequest object. (The CORS specification calls these "author request headers".) The rule does not apply to headers the browser can set, such as User-Agent, Host, or Content-Length.

Here is an example of a preflight request::

    OPTIONS http://myservice.azurewebsites.net/api/test HTTP/1.1
    Accept: */*
    Origin: http://myclient.azurewebsites.net
    Access-Control-Request-Method: PUT
    Access-Control-Request-Headers: accept, x-my-custom-header
    Accept-Encoding: gzip, deflate
    User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; WOW64; Trident/6.0)
    Host: myservice.azurewebsites.net
    Content-Length: 0

The pre-flight request uses the HTTP OPTIONS method. It includes two special headers:

- Access-Control-Request-Method: The HTTP method that will be used for the actual request.
- Access-Control-Request-Headers: A list of request headers that the application set on the actual request. (Again, this does not include headers that the browser sets.)

Here is an example response, assuming that the server allows the request::

    HTTP/1.1 200 OK
    Cache-Control: no-cache
    Pragma: no-cache
    Content-Length: 0
    Access-Control-Allow-Origin: http://myclient.azurewebsites.net
    Access-Control-Allow-Headers: x-my-custom-header
    Access-Control-Allow-Methods: PUT
    Date: Wed, 20 May 2015 06:33:22 GMT

The response includes an Access-Control-Allow-Methods header that lists the allowed methods, and optionally an Access-Control-Allow-Headers header, which lists the allowed headers. If the preflight request succeeds, the browser sends the actual request, as described earlier.
