# Entries_cs253_PART1_Notes

# 典型的 HTTP 会话

在客户端-服务器协议（如 HTTP）中，会话由三个阶段组成：

1. 客户端建立一个 TCP 连接（如果传输层不是 TCP，则建立适当的连接）。
2. 客户端发送它的请求，并等待答复。
3. 服务器处理请求，发回其答案，提供状态代码和适当的数据。

从 HTTP/1.1 开始，连接在完成第三阶段后不再关闭，并且客户端现在被授予进一步的请求：这意味着第二和第三阶段现在可以执行任意次数。

## [建立连接](https://developer.mozilla.org/en-US/docs/Web/HTTP/Session#establishing_a_connection)

在客户端-服务器协议中，建立连接的是客户端。在 HTTP 中打开连接意味着在底层传输层中启动连接，通常是 TCP。

使用 TCP 时，计算机上的 HTTP 服务器的默认端口是端口 80。也可以使用其他端口，例如 8000 或 8080。要获取的页面的 URL 包含域名和端口号，尽管如果后者是 80，则可以省略后者。有关更多详细信息，请参阅[标识 Web 上的资源。](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Identifying_resources_on_the_Web)

**注意：**客户端-服务器模型不允许服务器在没有明确请求的情况下向客户端发送数据。为了解决这个问题，Web 开发人员使用了几种技术：通过 API 定期 ping 服务器[`XMLHTTPRequest`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)，[`fetch()`](https://developer.mozilla.org/en-US/docs/Web/API/fetch)使用[WebSockets API](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API)或类似协议。

## [发送客户端请求](https://developer.mozilla.org/en-US/docs/Web/HTTP/Session#sending_a_client_request)

建立连接后，用户代理可以发送请求（用户代理通常是 Web 浏览器，但也可以是其他任何东西，例如爬虫）。客户端请求由文本指令组成，由 CRLF（回车，后跟换行）分隔，分为三个块：

1. 第一行包含一个请求方法，后跟它的参数：
   - 文档的路径，作为没有协议或域名的绝对 URL
   - HTTP 协议版本
2. 随后的行代表一个 HTTP 标头，向服务器提供有关哪种类型的数据是合适的（例如，什么语言、什么 MIME 类型）或其他改变其行为的数据（例如，如果它已被缓存，则不发送答案） . 这些 HTTP 标头形成一个以空行结尾的块。
3. 最后一个块是一个可选的数据块，它可能包含更多的数据，主要由 POST 方法使用。

### [示例请求](https://developer.mozilla.org/en-US/docs/Web/HTTP/Session#example_requests)

获取 developer.mozilla.org 的根页面 ( `https://developer.mozilla.org/`)，并告诉服务器用户代理更喜欢法语页面，如果可能的话：

```
GET / HTTP/1.1
Host: developer.mozilla.org
Accept-Language: fr
```

观察最后的空行，这将数据块与标题块分开。由于 HTTP 标头中没有`Content-Length`提供，因此此数据块显示为空，标记标头的结尾，允许服务器在收到此空行时处理请求。

例如，发送一个表单的结果：

```
POST /contact_form.php HTTP/1.1
Host: developer.mozilla.org
Content-Length: 64
Content-Type: application/x-www-form-urlencoded

name=Joe%20User&request=Send%20me%20one%20of%20your%20catalogue
```

### [请求方法](https://developer.mozilla.org/en-US/docs/Web/HTTP/Session#request_methods)

HTTP 定义了一组[请求方法](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)，指示要对资源执行的所需操作。尽管它们也可以是名词，但这些请求方法有时被称为 HTTP 动词。最常见的请求是`GET`and `POST`：

- 该[`GET`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/GET)方法请求指定资源的数据表示。使用的请求`GET`应该只检索数据。
- 该[`POST`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST)方法将数据发送到服务器，因此它可以更改其状态。这是[HTML 表单](https://developer.mozilla.org/en-US/docs/Learn/Forms)经常使用的方法。

## [服务器响应的结构](https://developer.mozilla.org/en-US/docs/Web/HTTP/Session#structure_of_a_server_response)

连接的代理发送请求后，Web 服务器对其进行处理，并最终返回响应。与客户端请求类似，服务器响应由文本指令组成，由 CRLF 分隔，但分为三个块：

1. 第一行，*状态行*，包含对使用的 HTTP 版本的确认，然后是响应状态代码（及其在人类可读文本中的简要含义）。
2. 随后的行表示特定的 HTTP 标头，为客户端提供有关已发送数据的信息（例如，类型、数据大小、使用的压缩算法、有关缓存的提示）。与客户端请求的 HTTP 标头块类似，这些 HTTP 标头形成一个以空行结尾的块。
3. 最后一个块是一个数据块，其中包含可选数据。

### [示例响应](https://developer.mozilla.org/en-US/docs/Web/HTTP/Session#example_responses)

成功的网页响应：

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 55743
Connection: keep-alive
Cache-Control: s-maxage=300, public, max-age=0
Content-Language: en-US
Date: Thu, 06 Dec 2018 17:37:18 GMT
ETag: "2e77ad1dc6ab0b53a2996dfd4653c1c3"
Server: meinheld/0.6.1
Strict-Transport-Security: max-age=63072000
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Vary: Accept-Encoding,Cookie
Age: 7

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>A simple webpage</title>
</head>
<body>
  <h1>Simple HTML5 webpage</h1>
  <p>Hello, world!</p>
</body>
</html>
```

请求的资源已永久移动的通知：

```
HTTP/1.1 301 Moved Permanently
Server: Apache/2.4.37 (Red Hat)
Content-Type: text/html; charset=utf-8
Date: Thu, 06 Dec 2018 17:33:08 GMT
Location: https://developer.mozilla.org/ (this is the new link to the resource; it is expected that the user-agent will fetch it)
Keep-Alive: timeout=15, max=98
Accept-Ranges: bytes
Via: Moz-Cache-zlb05
Connection: Keep-Alive
Content-Length: 325 (the content contains a default page to display if the user-agent is not able to follow the link)

<!DOCTYPE html... (contains a site-customized page helping the user to find the missing resource)
```

请求的资源不存在的通知：

```
HTTP/1.1 404 Not Found
Content-Type: text/html; charset=utf-8
Content-Length: 38217
Connection: keep-alive
Cache-Control: no-cache, no-store, must-revalidate, max-age=0
Content-Language: en-US
Date: Thu, 06 Dec 2018 17:35:13 GMT
Expires: Thu, 06 Dec 2018 17:35:13 GMT
Server: meinheld/0.6.1
Strict-Transport-Security: max-age=63072000
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Vary: Accept-Encoding,Cookie
X-Cache: Error from cloudfront

<!DOCTYPE html... (contains a site-customized page helping the user to find the missing resource)
```

### [响应状态代码](https://developer.mozilla.org/en-US/docs/Web/HTTP/Session#response_status_codes)

[HTTP 响应状态代码](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)指示特定 HTTP 请求是否已成功完成。响应分为五类：信息响应、成功响应、重定向、客户端错误和服务器错误。

- [`200`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/200)： 好的。请求已成功。
- [`301`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/301): 永久移动。此响应代码表示请求资源的 URI 已更改。
- [`404`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404)： 未找到。服务器找不到请求的资源。

## [也可以看看](https://developer.mozilla.org/en-US/docs/Web/HTTP/Session#see_also)

- [识别网络上的资源](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Identifying_resources_on_the_Web)
- [HTTP 标头](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)
- [HTTP 请求方法](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)
- [HTTP 响应状态码](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
