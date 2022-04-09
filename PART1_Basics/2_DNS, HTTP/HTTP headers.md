# HTTP 标头

**HTTP 标头**允许客户端和服务器通过 HTTP 请求或响应传递附加信息。HTTP 标头由不区分大小写的名称后跟冒号 ( `:`) 和其值组成。忽略值之前的[空格。](https://developer.mozilla.org/en-US/docs/Glossary/Whitespace)

自定义专有标头在历史上一直与前缀一起使用，但该约定在 2012 年 6 月被弃用，因为当非标准字段在[RFC 6648](https://datatracker.ietf.org/doc/html/rfc6648)`X-`中成为标准时它造成的不便；[其他人在IANA 注册机构](https://www.iana.org/assignments/message-headers/perm-headers.html)中列出，其原始内容在[RFC 4229](https://datatracker.ietf.org/doc/html/rfc4229)中定义。IANA 还维护一个[提议的新 HTTP 标头的注册表](https://www.iana.org/assignments/message-headers/prov-headers.html)。

标题可以根据其上下文进行分组：

- [请求标头](https://developer.mozilla.org/en-US/docs/Glossary/Request_header)包含有关要获取的资源或请求资源的客户端的更多信息。
- [响应标头](https://developer.mozilla.org/en-US/docs/Glossary/Response_header)包含有关响应的附加信息，例如其位置或提供它的服务器。
- [表示头](https://developer.mozilla.org/en-US/docs/Glossary/Representation_header)包含有关资源主体的信息，例如其[MIME 类型](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types)或应用的编码/压缩。
- [有效负载标头](https://developer.mozilla.org/en-US/docs/Glossary/Payload_header)包含有关有效负载数据的与表示无关的信息，包括内容长度和用于传输的编码。

标头也可以根据[代理](https://developer.mozilla.org/en-US/docs/Glossary/Proxy_server)处理它们的方式进行分组：

- [`Connection`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Connection)
- [`Keep-Alive`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Keep-Alive)
- [`Proxy-Authenticate`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Proxy-Authenticate)
- [`Proxy-Authorization`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Proxy-Authorization)
- [`TE`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/TE)
- [`Trailer`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Trailer)
- [`Transfer-Encoding`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Transfer-Encoding)
- [`Upgrade`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Upgrade)（另见[协议升级机制](https://developer.mozilla.org/en-US/docs/Web/HTTP/Protocol_upgrade_mechanism)）。

- 端到端标头

  这些标头*必须*传输给消息的最终接收者：请求的服务器或响应的客户端。中间代理必须重新传输这些未修改的标头，并且缓存必须存储它们。

- 逐跳标头

  这些标头仅对单个传输级连接有意义，并且*不得*由代理重新传输或缓存。请注意，只能使用标头设置逐跳标[`Connection`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Connection)头。

## 验证

- [`WWW-Authenticate`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/WWW-Authenticate)

  定义应用于访问资源的身份验证方法。

- [`Authorization`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization)

  包含用于向服务器验证用户代理的凭据。

- [`Proxy-Authenticate`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Proxy-Authenticate)

  定义应用于访问代理服务器后面的资源的身份验证方法。

- [`Proxy-Authorization`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Proxy-Authorization)

  包含使用代理服务器对用户代理进行身份验证的凭据。

## 缓存

- [`Age`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Age)

  对象在代理缓存中的时间（以秒为单位）。

- [`Cache-Control`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control)

  请求和响应中缓存机制的指令。

- [`Clear-Site-Data`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Clear-Site-Data)

  清除与请求网站相关的浏览数据（例如 cookie、存储、缓存）。

- [`Expires`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Expires)

  响应被视为过时的日期/时间。

- [`Pragma`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Pragma)

  特定于实现的标头，可能在请求-响应链的任何地方产生各种影响。用于向后兼容`Cache-Control`标头尚不存在的 HTTP/1.0 缓存。

- [`Warning`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Warning) 

  有关可能出现的问题的一般警告信息。

## 客户提示

HTTP[客户端提示](https://developer.mozilla.org/en-US/docs/Web/HTTP/Client_hints)是一组请求标头，它们提供有关客户端的有用信息，例如设备类型和网络条件，并允许服务器优化针对这些条件提供的服务。

服务器主动向客户端请求他们感兴趣的客户端提示头，使用[`Accept-CH`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-CH). 然后，客户端可以选择在后续请求中包含请求的标头。

- [`Accept-CH`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-CH) 

  服务器可以使用标头字段或具有属性`Accept-CH`的等效 HTML`<meta>`元素来宣传对客户端提示的支持。[`http-equiv`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/meta#attr-http-equiv)

- [`Accept-CH-Lifetime`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-CH-Lifetime) 

  服务器可以要求客户端记住服务器在指定时间段内支持的客户端提示集，以便在后续请求中将客户端提示传递到服务器的源。

下面列出了不同类别的客户端提示。

### 用户代理客户端提示

[UA 客户端提示](https://developer.mozilla.org/en-US/docs/Web/HTTP/Client_hints#user-agent_client_hints)是请求标头，提供有关用户代理及其运行的平台/架构的信息：

- [`Sec-CH-UA`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-CH-UA) 

  用户代理的品牌和版本。

- [`Sec-CH-UA-Arch`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-CH-UA-Arch) 

  用户代理的底层平台架构。

- [`Sec-CH-UA-Bitness`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-CH-UA-Bitness) 

  用户代理的底层 CPU 架构位数（例如“64”位）。

- [`Sec-CH-UA-Full-Version`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-CH-UA-Full-Version) 

  用户代理的完整语义版本字符串。

- [`Sec-CH-UA-Full-Version-List`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-CH-UA-Full-Version-List) 

  用户代理品牌列表中每个品牌的完整版。

- [`Sec-CH-UA-Mobile`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-CH-UA-Mobile) 

  用户代理在移动设备上运行，或者更一般地说，更喜欢“移动”用户体验。

- [`Sec-CH-UA-Model`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-CH-UA-Model) 

  用户代理的设备型号。

- [`Sec-CH-UA-Platform`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-CH-UA-Platform) 

  用户代理的底层操作系统/平台。

- [`Sec-CH-UA-Platform-Version`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-CH-UA-Platform-Version) 

  用户代理的底层操作系统版本。

### 设备客户端提示

- [`Content-DPR`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-DPR) 

  *响应标头*[`DPR`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/DPR)用于在客户端提示用于选择图像资源的请求中确认图像设备与像素的比率。

- [`Device-Memory`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Device-Memory) 

  可用客户端 RAM 内存的大致数量。这是[设备内存 API](https://developer.mozilla.org/en-US/docs/Web/API/Device_Memory_API)的一部分。

- [`DPR`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/DPR) 

  客户端设备像素比 (DPR)，即每个 CSS 像素对应的物理设备像素数。

- [`Viewport-Width`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Viewport-Width) 

  一个数字，以 CSS 像素表示布局视口宽度。提供的像素值是一个四舍五入到最小后续整数（即上限值）的数字。

- [`Width`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Width) 

  一个数字，以物理像素表示所需的资源宽度（即图像的固有大小）。

### 网络客户端提示

网络客户端提示允许服务器根据用户选择以及网络带宽和延迟来选择发送什么信息。

- [`Downlink`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Downlink)

  客户端与服务器连接的近似带宽，以 Mbps 为单位。这是[网络信息 API](https://developer.mozilla.org/en-US/docs/Web/API/Network_Information_API)的一部分。

- [`ECT`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ECT)

  与连接的延迟和带宽最匹配的[有效连接类型](https://developer.mozilla.org/en-US/docs/Glossary/Effective_connection_type)（“网络配置文件”）。这是[网络信息 API](https://developer.mozilla.org/en-US/docs/Web/API/Network_Information_API)的一部分。

- [`RTT`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/RTT)

  应用层往返时间 (RTT) 以毫秒为单位，其中包括服务器处理时间。这是[网络信息 API](https://developer.mozilla.org/en-US/docs/Web/API/Network_Information_API)的一部分。

- [`Save-Data`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Save-Data) 

  一个布尔值，指示用户代理对减少数据使用的偏好。

## 条件句

- [`Last-Modified`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Last-Modified)

  资源的最后修改日期，用于比较同一资源的多个版本。它不如 准确[`ETag`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag)，但在某些环境中更容易计算。条件请求使用[`If-Modified-Since`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Modified-Since)并[`If-Unmodified-Since`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Unmodified-Since)使用此值来更改请求的行为。

- [`ETag`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/ETag)

  标识资源版本的唯一字符串。条件请求使用[`If-Match`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Match)并[`If-None-Match`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-None-Match)使用此值来更改请求的行为。

- [`If-Match`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Match)

  使请求有条件，并仅在存储的资源与给定的 ETag 之一匹配时应用该方法。

- [`If-None-Match`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-None-Match)

  使请求有条件，并仅在存储的资源与任何给定的 ETag*不匹配时应用该方法。*这用于更新缓存（用于安全请求），或防止在已经存在新资源时上传新资源。

- [`If-Modified-Since`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Modified-Since)

  使请求有条件，并期望仅在给定日期之后修改资源时才传输资源。这仅用于在缓存过期时传输数据。

- [`If-Unmodified-Since`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Unmodified-Since)

  使请求有条件，并期望仅在给定日期之后未修改资源时才传输资源。这确保了特定范围的新片段与以前的片段的一致性，或者在修改现有文档时实现乐观并发控制系统。

- [`Vary`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Vary)

  确定如何匹配请求标头以决定是否可以使用缓存的响应，而不是从源服务器请求新的响应。

## 连接管理

- [`Connection`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Connection)

  控制当前事务完成后网络连接是否保持打开状态。

- [`Keep-Alive`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Keep-Alive)

  控制持久连接应保持打开状态的时间。

## Content_negotiation

内容协商标头。

- [`Accept`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept)

  通知服务器可以发回的数据[类型。](https://developer.mozilla.org/en-US/docs/Glossary/MIME_type)

- [`Accept-Encoding`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Encoding)

  编码算法，通常是[压缩算法](https://developer.mozilla.org/en-US/docs/Web/HTTP/Compression)，可用于发回的资源。

- [`Accept-Language`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Language)

  通知服务器有关服务器应发回的人类语言。这是一个提示，不一定在用户的完全控制之下：服务器应始终注意不要覆盖明确的用户选择（例如从下拉列表中选择语言）。

## Controls

- [`Expect`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Expect)

  表示需要由服务器满足以正确处理请求的期望。

- [`Max-Forwards`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Max-Forwards)

  待定

## Cookies

- [`Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cookie)

  包含服务器先前发送的带有标头的存储的[HTTP cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)[`Set-Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)。

- [`Set-Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)

  将 cookie 从服务器发送到用户代理。

## CORS

*[在此处](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)了解有关 CORS的更多信息。*

- [`Access-Control-Allow-Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Origin)

  指示是否可以共享响应。

- [`Access-Control-Allow-Credentials`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Credentials)

  指示当凭证标志为真时是否可以公开对请求的响应。

- [`Access-Control-Allow-Headers`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Headers)

  用于响应[预检请求](https://developer.mozilla.org/en-US/docs/Glossary/Preflight_request)，以指示在发出实际请求时可以使用哪些 HTTP 标头。

- [`Access-Control-Allow-Methods`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Allow-Methods)

  指定在访问资源以响应预检请求时允许的方法。

- [`Access-Control-Expose-Headers`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Expose-Headers)

  通过列出其名称来指示哪些标头可以作为响应的一部分公开。

- [`Access-Control-Max-Age`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Max-Age)

  指示预检请求的结果可以缓存多长时间。

- [`Access-Control-Request-Headers`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Request-Headers)

  在发出预检请求时使用，以让服务器知道在发出实际请求时将使用哪些 HTTP 标头。

- [`Access-Control-Request-Method`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Access-Control-Request-Method)

  在发出预检请求时使用，以让服务器知道在发出实际请求时将使用哪种[HTTP 方法。](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods)

- [`Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Origin)

  指示提取的来源。

- [`Timing-Allow-Origin`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Timing-Allow-Origin)

  指定允许查看通过[Resource Timing API](https://developer.mozilla.org/en-US/docs/Web/API/Resource_Timing_API)的功能检索到的属性值的来源，否则由于跨域限制，这些属性将被报告为零。

## Downloads

- [`Content-Disposition`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition)

  指示传输的资源是否应内联显示（默认行为不带标头），或者是否应像下载一样处理并且浏览器应显示“另存为”对话框。

## Message_body_information)

- [`Content-Length`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Length)

  资源的大小，以十进制字节数表示。

- [`Content-Type`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type)

  指示资源的媒体类型。

- [`Content-Encoding`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Encoding)

  用于指定压缩算法。

- [`Content-Language`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Language)

  描述面向受众的人类语言，以便用户可以根据自己的首选语言进行区分。

- [`Content-Location`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Location)

  指示返回数据的备用位置。

## Proxies)

- [`Forwarded`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Forwarded)

  包含来自代理服务器面向客户端的信息，当请求路径中涉及代理时，这些信息会被更改或丢失。

- [`X-Forwarded-For`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-For) 

  标识通过 HTTP 代理或负载平衡器连接到 Web 服务器的客户端的原始 IP 地址。

- [`X-Forwarded-Host`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Host) 

  标识请求客户端用于连接到您的代理或负载平衡器的原始主机。

- [`X-Forwarded-Proto`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Forwarded-Proto) 

  标识客户端用于连接到您的代理或负载均衡器的协议（HTTP 或 HTTPS）。

- [`Via`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Via)

  由代理添加，包括正向和反向代理，并且可以出现在请求标头和响应标头中。

## Redirects

- [`Location`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Location)

  指示要将页面重定向到的 URL。

## Request_context

- [`From`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/From)

  包含控制请求用户代理的人类用户的 Internet 电子邮件地址。

- [`Host`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Host)

  指定服务器的域名（用于虚拟主机）和（可选）服务器正在侦听的 TCP 端口号。

- [`Referer`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referer)

  前一个网页的地址，从该网页链接到当前请求的页面。

- [`Referrer-Policy`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy)

  管理在请求头中发送的引用信息[`Referer`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referer)应该包含在请求中。

- [`User-Agent`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent)

  包含一个特征字符串，允许网络协议对等体识别请求软件用户代理的应用程序类型、操作系统、软件供应商或软件版本。另请参阅[Firefox 用户代理字符串参考](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent/Firefox)。

## Response_context

- [`Allow`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Allow)

  列出资源支持的一组 HTTP 请求方法。

- [`Server`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Server)

  包含有关源服务器用于处理请求的软件的信息。

## Range_requests

- [`Accept-Ranges`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Ranges)

  指示服务器是否支持范围请求，如果支持，范围可以用哪个单位表示。

- [`Range`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Range)

  指示服务器应返回的文档部分。

- [`If-Range`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Range)

  创建一个条件范围请求，仅当给定的 etag 或日期与远程资源匹配时才满足。用于防止从不兼容版本的资源下载两个范围。

- [`Content-Range`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Range)

  指示部分消息在完整消息中的位置。

## Security

- [`Cross-Origin-Embedder-Policy`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Embedder-Policy)(COEP)

  允许服务器为给定文档声明嵌入器策略。

- [`Cross-Origin-Opener-Policy`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Opener-Policy)（合作社）

  防止其他域打开/控制窗口。

- [`Cross-Origin-Resource-Policy`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Resource-Policy)(公司)

  防止其他域读取应用此标头的资源的响应。

- [`Content-Security-Policy`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)( [CSP](https://developer.mozilla.org/en-US/docs/Glossary/CSP) )

  控制允许用户代理为给定页面加载的资源。

- [`Content-Security-Policy-Report-Only`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy-Report-Only)

  允许 Web 开发人员通过监控但不强制执行策略的效果来试验策略。这些违规报告包含通过 HTTP请求发送到指定 URI的[JSON文档。](https://developer.mozilla.org/en-US/docs/Glossary/JSON)`POST`

- [`Expect-CT`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Expect-CT)

  允许站点选择报告和/或执行证书透明度要求，从而防止该站点使用错误颁发的证书而被忽视。当一个站点启用 Expect-CT 标头时，他们要求 Chrome 检查该站点的任何证书是否出现在公共 CT 日志中。

- [`Feature-Policy`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Feature-Policy)

  提供一种机制来允许和拒绝在其自己的框架中以及在其嵌入的 iframe 中使用浏览器功能。

- [`Origin-Isolation`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Origin-Isolation) 

  提供一种机制以允许 Web 应用程序隔离其来源。

- [`Strict-Transport-Security`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)( [HSTS](https://developer.mozilla.org/en-US/docs/Glossary/HSTS) )

  强制使用 HTTPS 而不是 HTTP 进行通信。

- [`Upgrade-Insecure-Requests`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Upgrade-Insecure-Requests)

  向服务器发送一个信号，表示客户端对加密和经过身份验证的响应的偏好，并且它可以成功处理该[`upgrade-insecure-requests`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/upgrade-insecure-requests)指令。

- [`X-Content-Type-Options`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options)

  禁用 MIME 嗅探并强制浏览器使用[`Content-Type`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Type).

- [`X-Download-Options`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Download-Options)

  HTTP标[`X-Download-Options`](https://docs.microsoft.com/previous-versions/windows/internet-explorer/ie-developer/compatibility/jj542450(v=vs.85)?#the-noopen-directive)头指示浏览器 (Internet Explorer) 不应显示“打开”已从应用程序下载的文件的选项，以防止网络钓鱼攻击，否则该文件将获得在应用程序上下文中执行的访问权限。（注意：相关的[MS Edge 错误](https://developer.microsoft.com/en-us/microsoft-edge/platform/issues/18488178/)）。

- [`X-Frame-Options`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options)(XFO)

  指示是否应允许浏览器以[``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/frame)、[``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe)或呈现页面。[``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/embed)[``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/object)

- [`X-Permitted-Cross-Domain-Policies`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Permitted-Cross-Domain-Policies)

  指定是否允许跨域策略文件 ( `crossdomain.xml`)。该文件可能会定义一项策略，以授予客户端（例如 Adobe 的 Flash Player（现已过时）、Adobe Acrobat、Microsoft Silverlight（现已过时）或 Apache Flex）处理跨域数据的权限，否则这些域会因相同的限制而受到限制[。原产地政策](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy)。有关详细信息，请参阅[跨域策略文件规范](https://www.adobe.com/devnet/articles/crossdomain_policy_file_spec.html)。

- [`X-Powered-By`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Powered-By)

  可以由托管环境或其他框架设置，并包含有关它们的信息，但不会为应用程序或其访问者提供任何用处。取消设置此标头以避免暴露潜在的漏洞。

- [`X-XSS-Protection`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection)

  启用跨站点脚本过滤。

### HTTP_public_key_pinning（hpkp）

[HTTP 公钥固定](https://developer.mozilla.org/en-US/docs/Glossary/HPKP)已被弃用并删除，以支持证书透明度和[`Expect-CT`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Expect-CT).

- [`Public-Key-Pins`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Public-Key-Pins)

  将特定的加密公钥与特定的 Web 服务器相关联，以降低使用伪造证书进行[MITM攻击的风险。](https://developer.mozilla.org/en-US/docs/Glossary/MitM)

- [`Public-Key-Pins-Report-Only`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Public-Key-Pins-Report-Only)

  将报告发送到标头中指定的报告 uri，即使违反了固定，仍然允许客户端连接到服务器。

### Fetch_metadata_request_headers

[获取元数据请求标头](https://developer.mozilla.org/en-US/docs/Glossary/Fetch_metadata_request_header)提供有关发起请求的上下文的信息。这允许服务器根据请求的来源以及资源的使用方式来决定是否应允许请求。

- [`Sec-Fetch-Site`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-Fetch-Site)

  它是一个请求标头，指示请求发起者的来源与其目标的来源之间的关系。它是一个结构化标头，其值是具有可能值`cross-site`、`same-origin`、`same-site`和的标记`none`。

- [`Sec-Fetch-Mode`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-Fetch-Mode)

  它是一个请求标头，向服务器指示请求的模式。它是一个结构化标头，其值是具有可能值`cors`、`navigate`、`no-cors`、`same-origin`和的标记`websocket`。

- [`Sec-Fetch-User`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-Fetch-User)

  它是一个请求标头，指示导航请求是否由用户激活触发。它是一个结构化标题，其值为布尔值，因此可能的值为`?0`false 和`?1`true。

- [`Sec-Fetch-Dest`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-Fetch-Dest)

  它是一个请求标头，指示请求到服务器的目的地。它是一个结构化标题，其值是具有可能值的标记`audio`, `audioworklet`, `document`, `embed`, `empty`, `font`, `image`, , `manifest`, `object`, `paintworklet`, `report`, `script`, `serviceworker`, `sharedworker`, `style`, `track`, `video`, `worker`, 和`xslt`。

- [`Service-Worker-Navigation-Preload`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Service-Worker-Navigation-Preload)

  [`fetch()`](https://developer.mozilla.org/en-US/docs/Web/API/fetch)在服务工作者启动期间 以抢占式请求发送到资源的请求标头。用 设置的值[`NavigationPreloadManager.setHeaderValue()`](https://developer.mozilla.org/en-US/docs/Web/API/NavigationPreloadManager/setHeaderValue)可用于通知服务器应返回与正常`fetch()`操作不同的资源。

## Server-sent events

- [`Last-Event-ID`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Last-Event-ID)

  待定

- [`NEL`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/NEL) 

  定义一种机制，使开发人员能够声明网络错误报告策略。

- [`Ping-From`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Ping-From)

  待定

- [`Ping-To`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Ping-To)

  待定

- [`Report-To`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Report-To)

  用于指定浏览器向其发送警告和错误报告的服务器端点。

## Transfer_coding

- [`Transfer-Encoding`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Transfer-Encoding)

  指定用于将资源安全地传输给用户的编码形式。

- [`TE`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/TE)

  指定用户代理愿意接受的传输编码。

- [`Trailer`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Trailer)

  允许发件人在分块消息的末尾包含其他字段。

## Websockets

- [`Sec-WebSocket-Key`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-WebSocket-Key)

  待定

- [`Sec-WebSocket-Extensions`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-WebSocket-Extensions)

  待定

- [`Sec-WebSocket-Accept`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-WebSocket-Accept)

  待定

- [`Sec-WebSocket-Protocol`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-WebSocket-Protocol)

  待定

- [`Sec-WebSocket-Version`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-WebSocket-Version)

  待定

## Other

- [`Accept-Push-Policy`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Push-Policy) 

  客户端可以通过在请求中发送[`Accept-Push-Policy`](https://datatracker.ietf.org/doc/html/draft-ruellan-http-accept-push-policy-00#section-3.1)标头字段来表达请求的所需推送策略。

- [`Accept-Signature`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Signature) 

  客户端可以发送[`Accept-Signature`](https://wicg.github.io/webpackage/draft-yasskin-http-origin-signed-responses.html#rfc.section.3.7)标头字段来指示利用任何可用签名的意图并指示它支持哪些类型的签名。

- [`Alt-Svc`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Alt-Svc)

  用于列出访问此服务的替代方式。

- [`Date`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Date)

  包含生成消息的日期和时间。

- [`Early-Data`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Early-Data) 

  表示请求已在 TLS 早期数据中传送。

- [`Large-Allocation`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Large-Allocation) 

  告诉浏览器正在加载的页面将要执行大分配。

- [`Link`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Link)

  [`Link`](https://datatracker.ietf.org/doc/html/rfc5988#section-5)entity-header 字段提供了一种序列化 HTTP 标头中的一个或多个链接的方法。它在语义上等同于 HTML[``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/link)元素。

- [`Push-Policy`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Push-Policy) 

  A[`Push-Policy`](https://datatracker.ietf.org/doc/html/draft-ruellan-http-accept-push-policy-00#section-3.2)定义处理请求时有关推送的服务器行为。

- [`Retry-After`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Retry-After)

  指示用户代理在发出后续请求之前应等待多长时间。

- [`Signature`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Signature) 

  标[`Signature`](https://wicg.github.io/webpackage/draft-yasskin-http-origin-signed-responses.html#rfc.section.3.1)头字段传达了交换的签名列表，每个签名都附带有关如何确定该签名的权限和刷新该签名的信息。

- [`Signed-Headers`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Signed-Headers) 

  标[`Signed-Headers`](https://wicg.github.io/webpackage/draft-yasskin-http-origin-signed-responses.html#rfc.section.5.1.2)头字段标识要包含在签名中的响应标头字段的有序列表。

- [`Server-Timing`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Server-Timing)

  传达给定请求-响应周期的一个或多个指标和描述。

- [`Service-Worker-Allowed`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Service-Worker-Allowed)

  用于通过[在 Service Worker 脚本的响应中](https://w3c.github.io/ServiceWorker/#service-worker-script-response)包含此标头来删除[路径限制](https://w3c.github.io/ServiceWorker/#path-restriction)。

- [`SourceMap`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/SourceMap)

  将生成的代码链接到[源映射](https://firefox-source-docs.mozilla.org/devtools-user/debugger/how_to/use_a_source_map/index.html)。

- [`Upgrade`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Upgrade)

  [升级标头字段](https://datatracker.ietf.org/doc/html/rfc7230#section-6.7)的相关 RFC 文档是 RFC 7230，第 6.7 节。该标准建立了在当前客户端、服务器、传输协议连接上升级或更改为不同协议的规则。例如，此标头标准允许客户端从 HTTP 1.1 更改为 HTTP 2.0，假设服务器决定确认并实施 Upgrade 标头字段。任何一方都不需要接受 Upgrade 标头字段中指定的条款。它可以在客户端和服务器头中使用。如果指定了 Upgrade 头域，那么发送者也必须发送带有指定升级选项的 Connection 头域。有关 Connection 标头字段的详细信息，[请参阅上述 RFC 的第 6.1 节](https://datatracker.ietf.org/doc/html/rfc7230#section-6.1)。

- [`X-DNS-Prefetch-Control`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-DNS-Prefetch-Control)

  控制 DNS 预取，这是浏览器主动对用户可能选择关注的两个链接以及文档引用的项目（包括图像、CSS、JavaScript 等）的 URL 执行域名解析的功能。

- [`X-Firefox-Spdy`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Firefox-Spdy) 

  待定

- [`X-Pingback`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Pingback) 

  待定

- [`X-Requested-With`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Requested-With)

  待定

- [`X-Robots-Tag`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Robots-Tag)

  HTTP标[`X-Robots-Tag`](https://developers.google.com/search/reference/robots_meta_tag#xrobotstag)头用于指示如何在公共搜索引擎结果中索引网页。标头实际上等效于`<meta name="robots" content="...">`.

- [`X-UA-Compatible`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-UA-Compatible) 

  由 Internet Explorer 用来指示要使用的文档模式。
