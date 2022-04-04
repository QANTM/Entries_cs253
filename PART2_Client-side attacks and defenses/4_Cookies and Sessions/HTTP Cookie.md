# 使用 HTTP cookie

HTTP cookie 是服务器发送到用户 Web 浏览器的一小段数据。浏览器可能会存储 cookie 并将其与稍后的请求一起发送回同一服务器。通常，HTTP cookie 用于判断两个请求是否来自同一个浏览器——例如，让用户保持登录状态。[它为无状态](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview#http_is_stateless_but_not_sessionless)HTTP 协议 记住有状态信息。

Cookies主要用于三个目的：

- 会话管理：登录、购物车、游戏分数或服务器应记住的任何其他内容
- 个性化：用户偏好、主题和其他设置
- 追踪：记录和分析用户行为

Cookie 曾经用于一般的客户端存储。虽然当它们是在客户端存储数据的唯一方式时这是有道理的，但现在推荐使用现代存储 API。每次请求都会发送 Cookie，因此它们会降低性能（尤其是对于移动数据连接）。

用于客户端存储的现代 API 是[Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)（`localStorage`和`sessionStorage`）和[IndexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)。

**注意：**要查看存储的 cookie（以及网页可以使用的其他存储），您可以在开发人员工具中启用[存储检查器](https://firefox-source-docs.mozilla.org/devtools-user/storage_inspector/index.html)并从存储树中选择 Cookie。

## 创建 cookie

接收到 HTTP 请求后，服务器可以发送一个或多个[`Set-Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)响应头。浏览器通常存储 cookie 并将其与[`Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cookie)HTTP 标头内的同一服务器的请求一起发送。您可以指定不应发送 cookie 的过期日期或时间段。您还可以对特定域和路径设置附加限制，以限制 cookie 的发送位置。下文提到的 header 属性的详细信息，请参阅[`Set-Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)参考文章。

### `Set-Cookie`和标`Cookie`头

HTTP 响应标头将[`Set-Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)cookie 从服务器发送到用户代理。一个简单的 cookie 设置如下：

```
Set-Cookie: <cookie-name>=<cookie-value>
```

复制到剪贴板

这指示服务器发送标头告诉客户端存储一对 cookie：

```
HTTP/2.0 200 OK
Content-Type: text/html
Set-Cookie: yummy_cookie=choco
Set-Cookie: tasty_cookie=strawberry

[page content]
```

复制到剪贴板

然后，对于服务器的每个后续请求，浏览器都会使用标头将所有先前存储的 cookie 发送回服务器[`Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cookie)。

```
GET /sample_page.html HTTP/2.0
Host: www.example.org
Cookie: yummy_cookie=choco; tasty_cookie=strawberry
```

复制到剪贴板

**注意：**以下是`Set-Cookie`在各种服务器端应用程序中使用标头的方法：

- [PHP](https://secure.php.net/manual/en/function.setcookie.php)
- [节点.JS](https://nodejs.org/dist/latest-v14.x/docs/api/http.html#http_response_setheader_name_value)
- [Python](https://docs.python.org/3/library/http.cookies.html)
- [Ruby on Rails](https://api.rubyonrails.org/classes/ActionDispatch/Cookies.html)

### 定义 cookie 的生命周期

cookie 的生命周期可以通过两种方式定义：

- 当前会话结束时会删除*会话cookie。*浏览器定义“当前会话”何时结束，一些浏览器在重启时使用*会话恢复。*这可能会导致会话 cookie 无限期地持续。
- *永久*cookie 在`Expires`属性指定的日期或属性指定的一段时间后删除`Max-Age`。

例如：

```
Set-Cookie: id=a3fWa; Expires=Thu, 31 Oct 2021 07:28:00 GMT;
```

**注意：**当您设置`Expires`日期和时间时，它们与设置 cookie 的客户端相关，而不是服务器。

如果您的站点对用户进行身份验证，它应该重新生成并重新发送会话 cookie，即使是已经存在的，只要用户进行身份验证。这种方法有助于防止[会话固定攻击](https://developer.mozilla.org/en-US/docs/Web/Security/Types_of_attacks#session_fixation)，第三方可以重用用户的会话。

### 限制对 cookie 的访问

您可以通过以下两种方式之一确保 cookie 安全发送，并且不会被非预期方或脚本访问：使用`Secure`属性和`HttpOnly`属性。

带有该`Secure`属性的 cookie 仅通过 HTTPS 协议通过加密请求发送到服务器。它永远不会使用不安全的 HTTP 发送（本地主机除外），这意味着[中间人](https://developer.mozilla.org/en-US/docs/Glossary/MitM)攻击者无法轻松访问它。不安全的站点（`http:`在 URL 中带有）无法使用该`Secure`属性设置 cookie。但是，不要假设这会`Secure`阻止对 cookie 中敏感信息的所有访问。例如，有权访问客户端硬盘（或 JavaScript，如果`HttpOnly`未设置该属性）的人可以读取和修改信息。

JavaScript API`HttpOnly`无法访问具有该属性的 cookie ；[`Document.cookie`](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie)它只发送到服务器。例如，持续存在于服务器端会话中的 cookie 不需要对 JavaScript 可用，并且应该具有该`HttpOnly`属性。这种预防措施有助于缓解跨站点脚本 ( [XSS](https://developer.mozilla.org/en-US/docs/Web/Security/Types_of_attacks#cross-site_scripting_(xss)) ) 攻击。

这是一个例子：

```
Set-Cookie: id=a3fWa; Expires=Thu, 21 Oct 2021 07:28:00 GMT; Secure; HttpOnly
```

### 定义 cookie 的发送位置

`Domain`和`Path`属性定义了 cookie的*范围*：cookie 应该发送到哪些 URL。

#### 域属性

该`Domain`属性指定哪些主机可以接收 cookie。如果未指定，则该属性默认为设置 cookie 的同一[主机](https://developer.mozilla.org/en-US/docs/Glossary/Host)，*不包括 subdomains*。如果指定，`Domain` *则*始终包含子域。因此，指定`Domain`比省略它的限制要小。但是，当子域需要共享有关用户的信息时，它会很有帮助。

例如，如果您设置`Domain=mozilla.org`，则 cookie 可用于子域，如`developer.mozilla.org`.

#### 路径属性

该`Path`属性指示在请求的 URL 中必须存在的 URL 路径，以便发送`Cookie`标头。( `%x2F`"/") 字符被视为目录分隔符，子目录也匹配。

例如，如果您设置`Path=/docs`，则这些请求路径匹配：

- `/docs`
- `/docs/`
- `/docs/Web/`
- `/docs/Web/HTTP`

但是这些请求路径不会：

- `/`
- `/docsets`
- `/fr/docs`

#### SameSite 属性

该[`SameSite`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite)属性允许服务器指定是否/何时通过跨站点请求发送 cookie（其中[站点](https://developer.mozilla.org/en-US/docs/Glossary/Site)由可注册域和*方案*定义：http 或 https）。这提供了一些针对跨站点请求伪造攻击 ( [CSRF](https://developer.mozilla.org/en-US/docs/Glossary/CSRF) ) 的保护。它需要三个可能的值：`Strict`、`Lax`和`None`。

使用`Strict`时，cookie 只会发送到它的来源站点。 `Lax`类似，除了当用户*导航*到 cookie 的源站点时发送 cookie。例如，通过跟踪来自外部站点的链接。`None`指定在发起请求和跨站点请求时都发送 cookie，但*仅在安全上下文中*（即，如果还必须设置`SameSite=None`该属性）。`Secure`如果未`SameSite`设置任何属性，则 cookie 被视为`Lax`.

这是一个例子：

```
Set-Cookie: mykey=myvalue; SameSite=Strict
```

**注意：**与最近更改的标准相关`SameSite`（MDN 记录了上面的新行为）。有关在特定浏览器版本中如何处理属性的信息， 请参阅 cookie[浏览器兼容性表：](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite#browser_compatibility)

- `SameSite=Lax`如果`SameSite`未指定，则为新默认值。以前，默认情况下会为所有请求发送 cookie。
- 带有必须的 Cookie`SameSite=None`现在还指定`Secure`属性（它们需要安全上下文）。
- `http:`如果使用不同的方案 (或)发送来自同一域的 Cookie，则不再将其视为来自同一站点`https:`。

#### Cookie 前缀

由于 cookie 机制的设计，服务器无法确认 cookie 是从安全来源设置的，甚至无法判断cookie 最初设置的*位置。*

子域上的易受攻击的应用程序可以使用该`Domain`属性设置 cookie，从而可以访问所有其他子域上的该 cookie。这种机制可以在*会话固定*攻击中被滥用。有关主要缓解方法，请参阅[会话固定](https://developer.mozilla.org/en-US/docs/Web/Security/Types_of_attacks#session_fixation)。

但是，作为[纵深防御措施](https://en.wikipedia.org/wiki/Defense_in_depth_(computing))，您可以使用*cookie 前缀*来断言有关 cookie 的特定事实。有两个前缀可用：

- `__Host-`

  如果 cookie 名称具有此前缀，则[`Set-Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)仅当它也标有`Secure`属性、从安全源发送、不*包含*属性`Domain`且`Path`属性设置为时，它才会在标头中被接受`/`。这样，这些 cookie 可以被视为“域锁定”。

- `__Secure-`

  如果 cookie 名称具有此前缀，则[`Set-Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)只有在标有该`Secure`属性并从安全源发送时，它才会在标头中被接受。这比`__Host-`前缀弱。

浏览器将拒绝带有这些不符合其限制的前缀的 cookie。请注意，这确保了带有前缀的子域创建的 cookie 要么被限制在子域中，要么被完全忽略。由于应用程序服务器在确定用户是否经过身份验证或 CSRF 令牌是否正确时仅检查特定的 cookie 名称，因此这有效地充当了针对会话固定的防御措施。

**注意：**在应用程序服务器上，Web 应用程序*必须*检查包含前缀的完整 cookie 名称。用户代理在将其发送到请求的标头中之前*不会*[`Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cookie)从 cookie 中去除前缀。

有关 cookie 前缀和浏览器支持的当前状态的更多信息，请参阅[Set-Cookie 参考文章的前缀部分](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#cookie_prefixes)。

#### 使用 Document.cookie 访问 JavaScript

[`Document.cookie`](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie)您可以使用该属性通过 JavaScript 创建新的 cookie 。`HttpOnly`如果未设置该标志，您也可以从 JavaScript 访问现有的 cookie 。

```
document.cookie = "yummy_cookie=choco";
document.cookie = "tasty_cookie=strawberry";
console.log(document.cookie);
// logs "yummy_cookie=choco; tasty_cookie=strawberry"
```

复制到剪贴板

通过 JavaScript 创建的 Cookie 不能包含该`HttpOnly`标志。

请注意下面“安全”部分中的安全[问题](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#security)。JavaScript 可用的 Cookie 可以通过 XSS 窃取。

## 安全

**注意：**当您在 cookie 中存储信息时，请记住所有 cookie 值对最终用户都是可见的，并且可以由最终用户更改。根据应用程序的不同，您可能希望使用服务器查找的不透明标识符，或研究替代身份验证/保密机制，例如 JSON Web 令牌。

缓解涉及 cookie 的攻击的方法：

- 使用该`HttpOnly`属性防止通过 JavaScript 访问 cookie 值。
- 用于敏感信息（例如指示身份验证）的 Cookie 应具有较短的生命周期，其`SameSite`属性设置为`Strict`或`Lax`。（请参阅上面的[SameSite 属性](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#samesite_attribute)。）在[支持 SameSite 的浏览器中](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#browser_compatibility)，这可确保身份验证 cookie 不会随跨站点请求一起发送。这将使请求有效地未经身份验证应用程序服务器。

## 跟踪和隐私

### 第三方cookies

cookie 与特定域和方案（例如`http`或）相关联，如果设置了属性`https`，也可能与子域相关联。如果 cookie 域和方案与当前页面匹配，则认为该 cookie 与该页面来自同一站点，并称为*第一方 cookie*。 [`Set-Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie) `Domain`

如果域和方案不同，则 cookie 不被认为来自同一站点，称为*第三方 cookie*。虽然托管网页的服务器设置了第一方 cookie，但该页面可能包含存储在其他域中的服务器上的图像或其他组件（例如，广告横幅），可能会设置第三方 cookie。这些主要用于网络上的广告和跟踪。例如，[谷歌使用的 cookie 类型](https://policies.google.com/technologies/types)。

第三方服务器可以根据同一浏览器在访问多个站点时发送给它的 cookie 来创建用户浏览历史记录和习惯的配置文件。默认情况下，Firefox 会阻止已知包含跟踪器的第三方 cookie。第三方 cookie（或仅跟踪 cookie）也可能被其他浏览器设置或扩展程序阻止。Cookie 阻止可能导致某些第三方组件（例如社交媒体小部件）无法按预期运行。

**注意：**服务器可以（并且应该）设置 cookie [SameSite 属性](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite)以指定是否可以将 cookie 发送到第三方站点。

### Cookie 相关规定

涵盖使用 cookie 的立法或法规包括：

- 欧盟通用数据隐私条例 (GDPR)
- 欧盟的电子隐私指令
- 加州消费者隐私法

这些法规具有全球影响力。它们适用于来自这些司法管辖区的用户访问的*万维网*上的任何站点（欧盟和加利福尼亚州，但需要注意的是，加利福尼亚州的法律仅适用于总收入超过 2500 万美元的实体等）。

这些法规包括以下要求：

- 通知用户您的网站使用 cookie。
- 允许用户选择不接收部分或全部 cookie。
- 允许用户在不接收 cookie 的情况下使用您的大部分服务。

您所在地区可能还有其他法规管理 cookie 的使用。您有责任了解并遵守这些规定。有些公司提供“cookie 横幅”代码来帮助您遵守这些规定。

## 在浏览器中存储信息的其他方式

在浏览器中存储数据的另一种方法是[Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API/Using_the_Web_Storage_API)。[window.sessionStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage)和[window.localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)属性对应会话和永久 cookie 的持续时间，但比 cookie 具有更大的存储限制，并且永远不会发送到服务器。[可以使用IndexedDB API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)或基于其构建的库来存储更结构化和更大量的数据。

有一些技术旨在在 cookie 被删除后重新创建它们。这些被称为“僵尸”cookie。这些技术违反了用户隐私和用户控制的原则，可能违反数据隐私法规，并可能使使用它们的网站承担法律责任。

## 也可以看看

- [`Set-Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie)
- [`Cookie`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cookie)
- [`Document.cookie`](https://developer.mozilla.org/en-US/docs/Web/API/Document/cookie)
- [`Navigator.cookieEnabled`](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/cookieEnabled)
- [SameSite cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite)
- [使用 Storage Inspector 检查 cookie](https://firefox-source-docs.mozilla.org/devtools-user/storage_inspector/index.html)
- [Cookie 规范：RFC 6265](https://datatracker.ietf.org/doc/html/rfc6265)
- [维基百科上的 HTTP cookie](https://en.wikipedia.org/wiki/HTTP_cookie)
- [Cookie、GDPR 和 ePrivacy 指令](https://gdpr.eu/cookies/)
