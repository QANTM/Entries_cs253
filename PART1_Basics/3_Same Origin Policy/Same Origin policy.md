# 同源政策

**同源策略**是一种关键的安全机制，它限制一个源加载的文档或脚本如何[与](https://developer.mozilla.org/en-US/docs/Glossary/Origin)另一个源的资源进行交互。

它有助于隔离潜在的恶意文档，减少可能的攻击媒介。例如，它可以防止 Internet 上的恶意网站在浏览器中运行 JS，以从第三方 Web 邮件服务（用户登录）或公司内网（通过以下方式防止攻击者直接访问）读取数据没有公共 IP 地址）并将该数据转发给攻击者。

## [原产地的定义](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#definition_of_an_origin)

如果两个 URL的[协议](https://developer.mozilla.org/en-US/docs/Glossary/Protocol)、[端口](https://developer.mozilla.org/en-US/docs/Glossary/Port)（如果指定）和[主机](https://developer.mozilla.org/en-US/docs/Glossary/Host)相同，则两个 URL 具有*相同的来源*。您可能会看到这被称为“方案/主机/端口元组”，或者只是“元组”。（“元组”是一组项目，它们共同构成一个整体——双/三/四/五/等的通用形式。）

下表给出了与 URL 的来源比较示例`http://store.company.com/dir/page.html`：

| 网址                                              | 结果 | 原因                                |
| :------------------------------------------------ | :--- | :---------------------------------- |
| `http://store.company.com/dir2/other.html`        | 同源 | 只是路径不同                        |
| `http://store.company.com/dir/inner/another.html` | 同源 | 只是路径不同                        |
| `https://store.company.com/page.html`             | 失败 | 不同的协议                          |
| `http://store.company.com:81/dir/page.html`       | 失败 | 不同的端口（`http://`默认为80端口） |
| `http://news.company.com/dir/page.html`           | 失败 | 不同的主机                          |

### [继承的起源](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#inherited_origins)

从具有`about:blank`或`javascript:`URL 的页面执行的脚本会继承包含该 URL 的文档的来源，因为这些类型的 URL 不包含有关源服务器的信息。

例如，`about:blank`通常用作新的空弹出窗口的 URL，父脚本将内容写入其中（例如通过[`Window.open()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/open)机制）。如果此弹出窗口还包含 JavaScript，则该脚本将继承与创建它的脚本相同的来源。

`data:`URL 获得一个新的、空的、安全上下文。

### [Internet Explorer 中的异常](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#exceptions_in_internet_explorer)

Internet Explorer 对同源策略有两个主要例外：

- 信任区

  如果两个域都在*高度信任的区域*（例如公司 Intranet 域），则不应用同源限制。

- 港口

  IE 不将端口包含在同源检查中。因此，`https://company.com:81/index.html`和`https://company.com/index.html`被认为是相同的来源，没有任何限制。

这些例外是非标准的，并且在任何其他浏览器中均不受支持。

### [文件来源](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#file_origins)

`file:///`现代浏览器通常将使用模式 加载的文件的来源视为*不透明的来源*。这意味着如果一个文件包含来自同一文件夹的其他文件（例如），则它们不会被假定来自同一来源，并且可能会触发[CORS](https://developer.mozilla.org/en-US/docs/Glossary/CORS)错误。

请注意，[URL 规范](https://url.spec.whatwg.org/#origin)声明文件的来源取决于实现，并且某些浏览器可能会将同一目录或子目录中的文件视为同源，即使这具有[安全隐患](https://www.mozilla.org/en-US/security/advisories/mfsa2019-21/#CVE-2019-11730)。

## [改变原点](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#changing_origin)

**警告：**此处描述的方法（使用[`document.domain`](https://developer.mozilla.org/en-US/docs/Web/API/Document/domain)setter）已被弃用，因为它破坏了同源策略提供的安全保护，并使浏览器中的源模型复杂化，从而导致互操作性问题和安全漏洞。

一个页面可能会改变它自己的来源，但有一些限制。脚本可以将 的值设置[`document.domain`](https://developer.mozilla.org/en-US/docs/Web/API/Document/domain)为其当前域或其当前域的超域。如果设置为当前域的超域，则较短的超域用于同源检查。

例如，假设文档 at 中的脚本`http://store.company.com/dir/other.html`执行以下操作：

```
document.domain = "company.com";
```

复制到剪贴板

之后，页面可以通过同源检查`http://company.com/dir/page.html`（假设`http://company.com/dir/page.html`将其设置`document.domain`为“ `company.com`”以表示它希望允许 - 更多信息请参阅[`document.domain`](https://developer.mozilla.org/en-US/docs/Web/API/Document/domain)）。但是，`company.com`不能设置为，因为那不是 的超级**域**。`document.domain``othercompany.com``company.com`

端口号由浏览器单独检查。任何对 的调用`document.domain`，包括`document.domain = document.domain`，都会导致端口号被 覆盖`null`。因此，仅设置在第一个**是无法**与人`company.com:8080`交谈的。它必须在两者中设置，因此它们的端口号都是.`company.com``document.domain = "company.com"``null`

该机制有一些局限性。例如，如果启用或文档在沙箱中，它将抛出一个“ `SecurityError`” ，并且以这种方式更改源不会影响许多 Web API 使用的源检查（例如，、、、、）。可以在[Document.domain > Failures](https://developer.mozilla.org/en-US/docs/Web/API/Document/domain#failures)中找到更详尽的失败案例列表。[`DOMException`](https://developer.mozilla.org/en-US/docs/Web/API/DOMException)[`document-domain`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Feature-Policy/document-domain) [`Feature-Policy`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Feature-Policy)[``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe)[`localStorage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)[`indexedDB`](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)[`BroadcastChannel`](https://developer.mozilla.org/en-US/docs/Web/API/BroadcastChannel)[`SharedWorker`](https://developer.mozilla.org/en-US/docs/Web/API/SharedWorker)

**注意：**当使用`document.domain`允许子域访问其父域时，需要在父域和子域中设置`document.domain`为*相同的值。*即使这样做是将父域设置回其原始值，这也是必要的。不这样做可能会导致权限错误。

## [跨域网络访问](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#cross-origin_network_access)

同源策略控制两个不同源之间的交互，例如当您使用[`XMLHttpRequest`](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest)或[``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img)元素时。这些交互通常分为三类：

- 通常允许跨域*写入。*示例是链接、重定向和表单提交。一些 HTTP 请求需要[预检](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#preflighted_requests)。
- 通常允许跨域*嵌入。*（示例如下。）
- 跨域*读取*通常是不允许的，但读取访问权限通常会通过嵌入泄露。例如，您可以读取嵌入图像的尺寸、嵌入脚本的操作或[嵌入资源的可用性](https://bugzilla.mozilla.org/show_bug.cgi?id=629094)。

以下是一些可以跨域嵌入的资源示例：

- 带有`<script src="…"></script>`. 语法错误的错误详细信息仅适用于同源脚本。
- CSS 与`<link rel="stylesheet" href="…">`. 由于 CSS 的[语法规则宽松](https://scarybeastsecurity.blogspot.com/2009/12/generic-cross-browser-cross-domain.html)，跨域 CSS 需要正确的`Content-Type`标头。限制因浏览器而异：[Internet Explorer](https://docs.microsoft.com/previous-versions/windows/internet-explorer/ie-developer/compatibility/gg622939(v=vs.85)?redirectedfrom=MSDN)、[Firefox](https://www.mozilla.org/en-US/security/advisories/mfsa2010-46/)、[Chrome](https://bugs.chromium.org/p/chromium/issues/detail?id=9877)、[Safari](https://support.apple.com/kb/HT4070)（向下滚动到 CVE-2010-0051）和[Opera](https://security.opera.com/cross-domain-data-theft-with-css-load-opera-security-advisories/)。
- 显示的图像[``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img)。
- [``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video)和播放的媒体[``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/audio)。
- 嵌入了[``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/object)和的外部资源[``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/embed)。
- 应用的字体[`@font-face`](https://developer.mozilla.org/en-US/docs/Web/CSS/@font-face)。有些浏览器允许跨域字体，有些则需要同源。
- 嵌入的任何东西[``](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe)。站点可以使用[`X-Frame-Options`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options)标题来防止跨域框架。

### [如何允许跨域访问](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#how_to_allow_cross-origin_access)

使用[CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)允许跨域访问。CORS 是[HTTP](https://developer.mozilla.org/en-US/docs/Glossary/HTTP)的一部分，它允许服务器指定浏览器应允许从其加载内容的任何其他主机。

### [如何阻止跨域访问](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#how_to_block_cross-origin_access)

- 为防止跨域写入，请检查请求中不可猜测的令牌 - 称为[跨站点请求伪造 (CSRF)](https://owasp.org/www-community/attacks/csrf)令牌。您必须防止跨域读取需要此令牌的页面。
- 为防止资源的跨域读取，请确保它不可嵌入。通常有必要防止嵌入，因为嵌入资源总是会泄露一些关于它的信息。
- 为防止跨域嵌入，请确保您的资源不能被解释为上面列出的可嵌入格式之一。浏览器可能不尊重`Content-Type`标题。例如，如果您将`<script>`标签指向 HTML 文档，浏览器将尝试将 HTML 解析为 JavaScript。当您的资源不是站点的入口点时，您还可以使用 CSRF 令牌来防止嵌入。

## [跨域脚本API访问](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#cross-origin_script_api_access)

JavaScript API 像[`iframe.contentWindow`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLIFrameElement/contentWindow), [`window.parent`](https://developer.mozilla.org/en-US/docs/Web/API/Window/parent),[`window.open`](https://developer.mozilla.org/en-US/docs/Web/API/Window/open)和[`window.opener`](https://developer.mozilla.org/en-US/docs/Web/API/Window/opener)允许文档直接相互引用。当两个文档的来源不同时，这些引用提供对对象的访问非常有限[`Window`](https://developer.mozilla.org/en-US/docs/Web/API/Window)，[`Location`](https://developer.mozilla.org/en-US/docs/Web/API/Location)如下两节所述。

要在来自不同来源的文档之间进行通信，请使用[`window.postMessage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage).

规范：[HTML 生活标准§ 跨域对象](https://html.spec.whatwg.org/multipage/browsers.html#cross-origin-objects)。

### [窗户](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#window)

`Window`允许对这些属性进行以下跨域访问：

| 方法                                                         |
| :----------------------------------------------------------- |
| [`window.blur`](https://developer.mozilla.org/en-US/docs/Web/API/Window/blur) |
| [`window.close`](https://developer.mozilla.org/en-US/docs/Web/API/Window/close) |
| [`window.focus`](https://developer.mozilla.org/en-US/docs/Web/API/Window/focus) |
| [`window.postMessage`](https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) |

| 属性                                                         |         |
| :----------------------------------------------------------- | :------ |
| [`window.closed`](https://developer.mozilla.org/en-US/docs/Web/API/Window/closed) | 只读。  |
| [`window.frames`](https://developer.mozilla.org/en-US/docs/Web/API/Window/frames) | 只读。  |
| [`window.length`](https://developer.mozilla.org/en-US/docs/Web/API/Window/length) | 只读。  |
| [`window.location`](https://developer.mozilla.org/en-US/docs/Web/API/Window/location) | 读/写。 |
| [`window.opener`](https://developer.mozilla.org/en-US/docs/Web/API/Window/opener) | 只读。  |
| [`window.parent`](https://developer.mozilla.org/en-US/docs/Web/API/Window/parent) | 只读。  |
| [`window.self`](https://developer.mozilla.org/en-US/docs/Web/API/Window/self) | 只读。  |
| [`window.top`](https://developer.mozilla.org/en-US/docs/Web/API/Window/top) | 只读。  |
| [`window.window`](https://developer.mozilla.org/en-US/docs/Web/API/Window/window) | 只读。  |

一些浏览器允许访问比上述更多的属性。

### [地点](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#location)

`Location`允许对属性进行以下跨域访问：

| 方法                                                         |
| :----------------------------------------------------------- |
| [`location.replace`](https://developer.mozilla.org/en-US/docs/Web/API/Location/replace) |

| 属性            |        |
| :-------------- | :----- |
| `URLUtils.href` | 只写。 |

一些浏览器允许访问比上述更多的属性。

## [跨域数据存储访问](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#cross-origin_data_storage_access)

对存储在浏览器中的数据（例如[Web Storage](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)和[IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API) ）的访问按来源分开。每个源都有自己独立的存储，一个源中的 JavaScript 无法读取或写入属于另一个源的存储。

[Cookie](https://developer.mozilla.org/en-US/docs/Glossary/Cookie)使用单独的来源定义。页面可以为自己的域或任何父域设置 cookie，只要父域不是公共后缀即可。Firefox 和 Chrome 使用[公共后缀列表](https://publicsuffix.org/)来确定域是否为公共后缀。Internet Explorer 使用自己的内部方法来确定域是否为公共后缀。无论使用哪种协议 (HTTP/HTTPS) 或端口，浏览器都会为给定域（包括任何子域）提供 cookie。设置 cookie 时，您可以使用`Domain`、`Path`、`Secure`和`HttpOnly`标志来限制其可用性。当您读取 cookie 时，您无法看到它的设置位置。即使您只使用安全的 https 连接，您看到的任何 cookie 也可能是使用不安全的连接设置的。

## [也可以看看](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy#see_also)

- [W3C 的同源政策](https://www.w3.org/Security/wiki/Same_Origin_Policy)
- [web.dev 的同源策略](https://web.dev/secure/same-origin-policy)
- [`Cross-Origin-Resource-Policy`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Resource-Policy)
- [`Cross-Origin-Embedder-Policy`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cross-Origin-Embedder-Policy)

#### 发现此页面有问题？
