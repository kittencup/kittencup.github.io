---
layout: post
title: "HTTP Message Meta 文档"
date: 2015-02-06
categories: php
---

该提案的目的是为Http Message 提供一组通用的接口,如[RFC7230]描述（http://tools.ietf.org/html/rfc7230）和
[RFC7231]（http://tools.ietf.org/html/rfc7231），并如所描述的URI[RFC3986]（http://tools.ietf.org/html/rfc3986）（在HTTP消息的上下文中）。

- RFC 7230: http://www.ietf.org/rfc/rfc7230.txt
- RFC 7231: http://www.ietf.org/rfc/rfc7231.txt
- RFC 3986: http://www.ietf.org/rfc/rfc3986.txt

所有的Http Message包括被使用HTTP协议的版本(HTTP protocol version)，头(Header)和消息体(Message Body)。
一个请求(Request)建立的消息包含了http方法(Method)来用于创建这个请求，以及该发出请求的URI。一个响应包含了HTTP状态码和原因短语(Reason Phrase)

在PHP中，Http Message被用于两种情况:

- 要发送一个HTTP请求，通过`ext/curl`扩展，PHP原生流层(Stream layer)等，并处理所接收的HTTP响应，换句话说，
  Http Message应用于当PHP作为HTTP客户端时。
- 从服务器接受传入的HTTP请求，并给发出请求的客户端返回一个HTTP响应，当用于服务器端应用程序为满足HTTP请求时， PHP可以使用Http Message

该提案提出了全面描述PHP中的各种Http Message的所有部分的API。

2. 在PHP中的Http Message
-----------------------

PHP不具有Http Message的内置支持。

### 客户端的HTTP支持

PHP支持通过多种机制发送HTTP请求:

- [PHP streams](http://php.net/streams)
- [cURL extension](http://php.net/curl)
- [ext/http](http://php.net/http) (v2也试图解决服务器端支持)

PHP Streams 是发送HTTP请求的最方便和最普遍的方式，但是也造成了一些局限性需要正确地配置SSL支持，并提供一个繁琐的接口用来设置，如 头(headers)。Curl 提供了一个完整的和扩展的功能集，但是，由于它不是一个默认的扩展，往往是未安装的。 HTTP的扩展也如Curl面临着同样的问题，以及事实上，历来有很少的使用的例子。

大多数现代HTTP客户端库倾向于抽象的实现，确保他们能够在工作环境中的任何它们执行上，并且跨越任何上述的层。

### 服务器端的HTTP支持

PHP使用服务器APIS（SAPI）来解释传入的HTTP请求，封送输入，并执行脚本，原来的SAPI的设计反映了通用网关接口(CGI)，在传入委托的脚本之前，这将封送请求数据，并将其推到环境变量，脚本将从环境变量来处理请求并返回一个响应。

PHP的SAPI设计抽象通用的输入源，如cookies,查询字符串参数(query string arguments)和POST内容等，通过超全局变量（$_COOKIE，$_GET和$_POST..），为Web开发人员提供了方便。

在服务器端Http的响应，PHP最初是作为一个模板语言，并允许混用HTML和PHP，文件的任何HTML部分被立即刷新到输出缓冲区。然后，现在的应用程序和框架，都避开了这种做法，因为这样会导致发出状态行或响应头的问题，他们往往会聚集所有的标题和内容，当其他所有应用程序处理完成后并发出一次。需要特别小心，注意确保错误报告(error reporting)和其他actions内容发送到输出缓冲区不刷新输出缓冲区。

3. Why Bother?
--------------

Http Mssages 被使用在多数的PHP项目中-- 客户端和服务器。在每一种情况下，我们观察到以下一种模式或一种或多种情况：

1. 项目直接使用PHP的超全局变量。
2. 项目将从头开始创建实现。
3. 项目可能需要特定的HTTP client/server库，提供Http Message实现。
4. 项目可能为Http Message实现而创造适配器(adapters)。

为例：

1. 在框架兴起前,很多应用不断发展, 其中包括一些很流行的CMS，论坛，购物车系统，在历史上都使用超级全局变量(superglobals)。
2. 如Syfmony和Zend Framework框架在自己的MVC层构成的基础上都定义了HTTP组件，即使是小而单一用途的库，如oauth2-server-php，他们会提供自己的HTTP请求/响应实现。Guzzle, Buzz和其他HTTP客户端创建自己的HttpMessage的实现也是如此。
3. 项目如Silex,Stack和Drupal 8依赖于Symfony Http Kernel，建立在Guzzle任何SDK都要对Guzzle的Http Message实现一个硬性要求。
4. 项目如Geocoder创建的冗余[公共库的适配器](https://github.com/geocoder-php/Geocoder/tree/6a729c6869f55ad55ae641c74ac9ce7731635e6e/src/Geocoder/HttpAdapter).

直接使用超全局变量有一些担忧。首先，这些都是可变的，使得库和代码改变值，从而改变应用程序状态。此外，超全局变量使单元测试和集成测试困难，导致代码质量下降。

目前的框架生态系统中，框架在实现HTTP消息的抽象，最终结果是，项目不可不操作性或交叉性(cross-pollination)，为了对一个框架使用另一个框架，首先要做的是Http Message的实现之间建一座桥(bridge)，在客户端，如果一个特定的库没有适配器可以使用，如果你想使用另一个库的适配器，你需要桥接Request/Response。

最后，当涉及到服务器端的响应的时候，PHP会以自己的方式：）在调用`header()`前发布任何内容将导致调用变为no-op，取决于错误报告设置，这往往意味着headers 和/或 响应状态没有正确发送。解决这个问题的方法是使用PHP的输出缓冲功能，但输出缓冲器的嵌套可以成为难以调试的问题。框架和应用程序从而倾向于创建来聚合标题和内容，可一次发送的响应抽象 - 而这些抽象概念往往又是不兼容的。

因此，该方案的目标是抽象的客户端和服务器端的请求和响应的接口，以促进项目之间的互操作性，如果项目实现这些接口，当采用不同的库代码是可以假设有一个合理的兼容性

这个方案的目的不是为了淘汰现有的php库的接口. 这个方案是针对描述Http Message为目的的开源软件之间的互操作性。

4. 范围
--------

### 4.1 目标

* 提供描述HTTP消息所需要的接口。
* 注重实际应用和可用性。
* 定义Http Message和URI规范接口模型的所有元素
* 确保API不设任何限制的Http Message,例如，一些Http Message存储在内存中可能太大，所以我们必须考虑到这一点。
* 为处理传入的服务器端应用程序和HTTP客户端发送的请求，提供有用的抽象

### 4.2 非目标

* 这一建议并不指望所有的HTTP客户端库或服务器端框架改变其接口符合。它是严格意味着互操作性。
* 每个人对于细节的感知是不一样的,这个提议不应该强加实现细节，然而，因为RFC的7230，7231，和3986不强迫任何特定的实现，需要在PHP中描述Http Message的接口将会有一定数量的实现。

5. 设计决策
-------------------

### Message 设计

`MessageInterface` 提供了访问不论是对请求或响应共同所有的Http Message的元素。这些要素包括：

- HTTP protocol version (e.g., "1.0", "1.1")
- HTTP headers
- HTTP message body

还有更具体的接口用于描述requests和responses，也更具体的上下文(客户端和服务器端)。 这些分歧部分灵感来自现有的PHP,但也通过其他语言如
Ruby's [Rack](https://rack.github.io),
Python's [WSGI](https://www.python.org/dev/peps/pep-0333/),
Go's [http package](http://golang.org/pkg/net/http/),
Node's [http module](http://nodejs.org/api/http.html), 等.

### 为什么header方法在message里好过在header包(header bag)里？

message本身就是headers的容器(还有其他message属性)，虽然header表示是一种内部执行细节，但统一访问header是mesaage的主要意义。

### 为什么URIs表示为对象？

URIs是通过其值来被定义特性, 因此应该设置为值对象.

除此之外，URIs 包含了在特定的请求中能被多次访问的片段 -- 这些片段值需要解析URI来确定(比如.,通过 paser_url()),URIs作为value objects允许被解析一次，并简化了访问各个URI片段，它也提供了方便在客户端应用程序中允许用户改变URI中片段创建新的URI实例（例如，只更新的URI的路径）。

### 为什么request接口有方法来处理请求目标并组成URI

RFC7230 详细介绍了请求行(request line)包含一个“请求的目标”。在4种形式的请求目标中，只有一个URI符合RFC 3986。最常用的形式是origin-form，它代表了没有scheme 或 authority信息的URI的。

因此`RequestInterface`具有相关request-target的方法，默认情况下，它将使用由URI提出来源请求的目标，并且，在不存在的URI实例，则返回字符串“/”。另外一个方法`withRequestTarget()`，允许通过一个明确的请求目标来获取一个新的实例，允许用户根据另一种有效的request-target的形式来创建requests.

URI在Request中为了各种原因被要求保存成独立片段，对于客户端和服务器,通常要求了解独立的URI。在客户端中，为了TCP连接，需要使用具体的scheme和authority细节,在服务器应用中，完整的URI通常需要为了验证该请求或路由到一个合适的处理程序。

### 为什么使用value objects?

该提案的模型消息和URIs作为[value objects](http://en.wikipedia.org/wiki/Value_object).

Messages的特性是通过聚合其所有的组成部分值来定义的，改变Message任何部分的实际上则是一个新Message，这是一个value objects的定义，通过该变化则获得新的实例的做法被称为[immutability](http://en.wikipedia.org/wiki/Immutable_object),这是一个特性设计以保证一个给定的值的完整性。

然而,这一提议也认识到,大多数客户端和服务器端应用程序将需要能够轻松地更新Message某一部分,因此，当更新时也提供接口方法来创建一个新的实例，这些方法通常前缀是with or without。

当塑造Http Message Value objects 给提供了几个好处:

- 改变URI状态时不会改变这个request构成的URI实例。
- 改变message的header时不会改变这个request的header

从本质上讲，将Http Message设置未value objects可以确保该message状态的完整性，并避免需要双向依赖的问题，这往往可以去除不同步或导致调试或性能问题。

对于Http客户端来说，他们允许用户去建立一个包含标准URI和请求headers的base request数据，而无需建立一个全新的request或为客户端发送的每条消息重置状态。

```php
$uri = new Uri('http://api.kittencup.com');
$baseRequest = new Request($uri, null, [
    'Authorization' => 'Bearer ' . $token,
    'Accept'        => 'application/json',
]);;

$request = $baseRequest->withUri($uri->withPath('/user'))->withMethod('GET');
$response = $client->send($request);

// get user id from $response

$body = new StringStream(json_encode(['tasks' => [
    'Code',
    'Coffee',
]]));;
$request = $baseRequest
    ->withUri($uri->withPath('/tasks/user/' . $userId))
    ->withMethod('POST')
    ->withHeader('Content-Type' => 'application/json')
    ->withBody($body);
$response = $client->send($request)

// No need to overwrite headers or body!
$request = $baseRequest->withUri($uri->withPath('/tasks'))->withMethod('GET');
$response = $client->send($request);
```

在服务器端，开发人员将需要：

- 反序列化请求消息体。
- 解码HTTP cookies。
- 写入响应(response)。

这些操作对于使用value objects可以实现，并会有一些好处:

原始请求的状态可以存储检索任何消费。
- 原始请求的状态可以存储供任何消费者检索.
- 默认响应状态(response state)可以创建默认的标题(headers)和/或消息体(message body). 

最流行的PHP框架今天有完全可变的Http Message，主要的变化是有必要的使用value object:

- 而不是调用setter方法或设置公共属性，转换函数将被调用，并分配结构
- 开发人员必须通知应用程序状态的改变

作为一个例子,在Zend Framework 2中,代替以下:

```php
function (MvcEvent $e)
{
    $response = $e->getResponse();
    $response->setHeaderLine('x-foo', 'bar');
}
```

现在这样写:

```php
function (MvcEvent $e)
{
    $response = $e->getResponse();
    $e->setResponse(
        $response->withHeader('x-foo', 'bar');
    )
}
```

上面代码在单个调用中结合了分配和通知

这种做法有一个明确的好处，会更改整个应用程序状态。

### 使用流(Stream)代替 X

`MessageInterface` 的body值必须实现 `StreamableInterface`. 这种设计决定使开发人员可以发送和接受(和/或接受和发送)Http MessageH数据超过实际存储在内存中的，同时还允许便利的使用message bodies作为字符串进行交互。尽管PHP提供了流抽象包装器(stream wrappers),但流资源使用会很麻烦，流资源只能通过`stream_get_contents()`转换为字符串，或者手动读取一个字符串的剩余部分。自定义添加对Stream操作行为需要注册一个Stream Filter;然后,只有在php中先注册Stream filter才能对流使用这个stream filter(即没有stream filter自动加载的机制)

使用一个明确定义的Stream 接口允许灵活的Stream 装饰可以添加到请求或响应前启用，像加密，压缩，确保了下载的字节数反映在response中的`Content-lenght`中,等。装饰流在[Java](http://docs.oracle.com/javase/7/docs/api/java/io/package-tree.html)和[Node](http://nodejs.org/api/stream.html#stream_class_stream_transform_1)社区是一种行之有效的模式,它允许非常灵活的数据流

大多数`StreamableInterface` API是基于[Python的IO模块](http://docs.python.org/3.1/library/io.html)，它提供了一种实用的API，而不是实现类似WritableStreamInterface和ReadableStreamInterface流功能，Stream提供方法像isReadable(),isWritable(),等等，这种方法被使用在Python,[C#, C++](http://msdn.microsoft.com/en-us/library/system.io.stream.aspx),[Ruby](http://www.ruby-doc.org/core-2.0.0/IO.html),[Node](http://nodejs.org/api/stream.html),和其他语言。

### 为什么Streams是可变的?

`StreamableInterface` API 包含了很多方法诸如write(),它可以改变message内容 -- 这直接违背了不可变Message。

出现的问题是由于这样一个事实:接口是为了包装一个PHP流或类似的。因此写操作将代理写入到包装的流中。即使我们`StreamableInterface`不可变,一旦Stream已经更新,任何实例包装流也将被更新 --使得不变性不可能实现的

我们的建议是，为服务器端的请求和客户端的响应实现使用只读流。

### ServerRequestInterface的理由

所述`RequestInterface`和`ResponseInterface`在[RFC 7230](http://www.ietf.org/rfc/rfc7230.txt)中描述的请求和响应消息基本上具有为1：1的相关性，它们提供的接口实现对应于它们模拟特定的Http Message类型的value objects。

然而服务器端应用程序对传入请求还有其他注意事项:

- 访问服务器的参数(可能来自请求,但也可能服务器配置的结果,而且通常代表通过`$_SERVER`获取;这些都是PHP服务器 API的一部分(SAPI))
- 访问查询字符串参数 (通过`$_GET`全局变量).
- 访问body数据 (也就是说，从传入请求体反序列化数据;在PHP中，这是POST请求的`application/x-www-urlencoded`的内容类型，并封装在`$_POST`全局变量).
- 访问上传内容 (在PHP中通过`$_FILES`全局变量).
- 访问cookies (在PHP中通过`$_COOKIE`全局变量).
- 访问request的属性 (通常，但不限于，那些相匹配的URL路径).

对这些参数的统一访问增加框架和库之间的互操作性的可行性,因为他们现在可以假设如果请求实现`ServerRequestInterface`,他们可以获取这些值。它也解决了在PHP语言本身的问题:

- 直到 5.6.0, `php://input` 将被只读一次; 因此，作为第一访问的`php://input`将是唯一的一个接收该数据，实例化来自多个框架的多个请求frameworks/libraries可能导致不一致的状态。
- 对超全局单元测试（如`$_GET`，`$_FILES`等）是困难的。封装他们在`ServerRequestInterface`内部是实现简化测试的考虑。

### 什么是 "特殊" header 值?

一些header值包含了独特的表现要求，可带来使用以及生成问题，特别是cookies和`Accept` header。

这个提议并没有提供任何头类型的任何特殊处理。`MessageInterface`提供了header检索和设置方法，并且获取所有header值返回的是字符串值

鼓励开发者为这些特殊的header value编写类库，无论目的是解析或生成。用户需要与这些特殊的header value交互时，可以使用这些库，这样的例子，实践中已经存在的库，如[willdurand/Negotiation](https://github.com/willdurand/Negotiation)和[aura/accept](https://github.com/pmjones/Aura.Accept)，只要该库能铸造成一个字符串的值，这些对象可以被用来填充Http Message的头。
