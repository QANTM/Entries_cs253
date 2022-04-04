# 跨站点请求防伪备忘单

## 介绍

[跨站点请求伪造 (CSRF)](https://owasp.org/www-community/attacks/csrf) 是一种攻击类型，当恶意网站、电子邮件、博客、即时消息或程序导致用户的 Web 浏览器在用户通过身份验证时在受信任的站点上执行不需要的操作时，就会发生这种攻击。CSRF 攻击之所以有效，是因为浏览器请求会自动包含所有 cookie，包括会话 cookie。因此，如果用户通过了站点的身份验证，则站点无法区分合法的授权请求和伪造的经过身份验证的请求。当使用适当的授权时，这种攻击会被阻止，这意味着需要一个挑战-响应机制来验证请求者的身份和权限。

成功的 CSRF 攻击的影响仅限于易受攻击的应用程序暴露的能力和用户的权限。例如，这种攻击可能导致资金转移、更改密码或使用用户凭证进行购买。实际上，攻击者使用 CSRF 攻击使目标系统通过受害者的浏览器执行一项功能，在受害者不知情的情况下，至少在提交未经授权的事务之前。

总之，防御CSRF应该遵循以下原则：

- 检查你的框架是否有[内置的 CSRF 保护](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#use-built-in-or-existing-csrf-implementations-for-csrf-protection)并使用它
  - **如果框架没有内置的 CSRF 保护，则将[CSRF 令牌](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#token-based-mitigation)添加到所有状态更改请求（导致站点上的操作的请求）并在后端验证它们**
- **对于有状态的软件，使用[同步器令牌模式](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#synchronizer-token-pattern)**
- **对于无状态软件，使用[双重提交 cookie](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#double-submit-cookie)**
- [从深度防御](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#defense-in-depth-techniques)部分实施至少一项缓解措施
  - **考虑会话 cookie 的[SameSite Cookie 属性](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#samesite-cookie-attribute)**，但请注意不要专门为域设置 cookie，因为这会引入该域的所有子域共享 cookie 的安全漏洞。当子域的 CNAME 指向不受您控制的域时，这尤其是一个问题。
  - **考虑为高度敏感的操作实施[基于用户交互的保护](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#user-interaction-based-csrf-defense)**
  - **考虑[使用自定义请求标头](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#use-of-custom-request-headers)**
  - **考虑[使用标准标题验证来源](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#verifying-origin-with-standard-headers)**
- 请记住，任何跨站点脚本 (XSS) 都可以用来击败所有 CSRF 缓解技术！
  -  **有关如何防止 XSS 缺陷的详细指导，请参阅 OWASP [XSS 预防备忘单。](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)**
- 不要将 GET 请求用于状态更改操作。
  - **如果您出于任何原因这样做，请保护这些资源免受 CSRF 的影响**

## 基于令牌的缓解

[同步器令牌模式](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#synchronizer-token-pattern)是缓解 CSRF 的最流行和推荐的方法之一。

### 使用内置或现有的 CSRF 实现进行 CSRF 保护

同步器令牌防御已内置到许多框架中。强烈建议您在尝试构建自定义令牌生成系统之前研究您使用的框架是否具有默认实现 CSRF 保护的选项。例如，.NET 具有向 CSRF 易受攻击的资源添加令牌的[内置保护](https://docs.microsoft.com/en-us/aspnet/core/security/anti-request-forgery?view=aspnetcore-2.1)。在使用这些生成令牌以保护 CSRF 易受攻击资源的内置 CSRF 保护之前，您有责任进行适当的配置（例如密钥管理和令牌管理）。

### 同步器令牌模式

CSRF 令牌应该在服务器端生成。它们可以为每个用户会话或每个请求生成一次。每个请求令牌比每个会话令牌更安全，因为攻击者利用被盗令牌的时间范围很小。但是，这可能会导致可用性问题。例如，“返回”按钮浏览器功能经常受到阻碍，因为前一页可能包含不再有效的令牌。与上一页的交互将导致服务器上出现 CSRF 误报安全事件。在初始生成令牌后的每个会话令牌实现中，该值存储在会话中并用于每个后续请求，直到会话到期。

当客户端发出请求时，服务器端组件必须与在用户会话中找到的令牌相比，验证请求中令牌的存在和有效性。如果在请求中未找到令牌，或者提供的值与用户会话中的值不匹配，则应中止请求。还应考虑其他操作，例如将事件记录为正在进行的潜在 CSRF 攻击。

CSRF 令牌应该是：

- 每个用户会话唯一。
- 秘密
- [不可预测（安全方法](https://cheatsheetseries.owasp.org/cheatsheets/Cryptographic_Storage_Cheat_Sheet.html#rule---use-cryptographically-secure-pseudo-random-number-generators-csprng)生成的大随机值）。

CSRF 令牌阻止 CSRF，因为没有令牌，攻击者无法创建对后端服务器的有效请求。

**CSRF 令牌不应使用 cookie 传输。**

CSRF 令牌可以通过隐藏字段、标题添加，并且可以与表单和 AJAX 调用一起使用。确保令牌没有在服务器日志或 URL 中泄露。GET 请求中的 CSRF 令牌可能会在多个位置泄露，例如浏览器历史记录、日志文件、记录 HTTP 请求第一行的网络设备，以及受保护站点链接到外部站点时的 Referer 标头。

例如：

```
<form action="/transfer.do" method="post">
<input type="hidden" name="CSRFToken" value="OWY4NmQwODE4ODRjN2Q2NTlhMmZlYWEwYzU1YWQwMTVhM2JmNGYxYjJiMGI4MjJjZDE1ZDZMGYwMGEwOA==">
[...]
</form>
```

通过 JavaScript 在自定义 HTTP 请求标头中插入 CSRF 令牌被认为比在隐藏字段表单参数中添加令牌更安全，因为它[使用自定义请求标头](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#use-of-custom-request-headers)。

### 双重提交 Cookie

如果在服务器端维护 CSRF 令牌的状态是有问题的，另一种防御方法是使用双重提交 cookie 技术。这种技术易于实现并且是无状态的。在这种技术中，我们在 cookie 和请求参数中发送一个随机值，服务器验证 cookie 值和请求值是否匹配。当用户访问时（甚至在进行身份验证以防止登录 CSRF 之前），站点应该生成一个（加密性强的）伪随机值，并将其设置为用户机器上的 cookie，与会话标识符分开。然后，该站点要求每个事务请求都包含此伪随机值作为隐藏表单值（或其他请求参数/标头）。如果它们都在服务器端匹配，则服务器将其作为合法请求接受，如果不匹配，

为了增强此解决方案的安全性，将令牌包含在加密 cookie 中 - 除了身份验证 cookie（因为它们通常在子域中共享） - 然后在服务器端将其与隐藏的令牌匹配（在解密加密的 cookie 之后） AJAX 调用的表单字段或参数/标题。这是因为如果没有必要的信息（如加密密钥），子域无法覆盖正确制作的加密 cookie。

加密 cookie 的一个更简单的替代方法是使用只有服务器知道的密钥对令牌进行 HMAC 处理，并将该值放入 cookie 中。这类似于加密的 cookie（两者都需要只有服务器拥有的知识），但比加密和解密 cookie 的计算量要小。无论是使用加密还是 HMAC，攻击者都无法在不知道服务器机密的情况下从普通令牌重新创建 cookie 值。

## 纵深防御技术

### SameSite Cookie 属性

SameSite 是一个 cookie 属性（类似于 HTTPOnly、Secure 等），旨在缓解 CSRF 攻击。它在[RFC6265bis](https://tools.ietf.org/html/draft-ietf-httpbis-rfc6265bis-02#section-5.3.7)中定义。此属性帮助浏览器决定是否将 cookie 与跨站点请求一起发送。此属性的可能值为`Lax`、`Strict`或`None`。

Strict 值将阻止浏览器在所有跨站点浏览上下文中将 cookie 发送到目标站点，即使遵循常规链接也是如此。例如，对于类似 GitHub 的网站，这意味着如果登录的用户点击公司讨论论坛或电子邮件上发布的私人 GitHub 项目的链接，GitHub 将不会收到会话 cookie，并且用户将无法访问该项目。然而，银行网站不希望允许从外部网站链接任何交易页面，因此 Strict 标志将是最合适的。

对于希望在用户从外部链接到达后保持用户登录会话的网站，默认的 Lax 值在安全性和可用性之间提供了合理的平衡。在上面的 GitHub 场景中，当从外部网站跟踪常规链接时，会话 cookie 将被允许，而在 POST 等 CSRF 倾向的请求方法中阻止它。只有在 Lax 模式下允许的跨站点请求是具有顶级导航并且也是[安全](https://tools.ietf.org/html/rfc7231#section-4.2.1)的HTTP 方法的请求。

有关这些值的更多详细信息，请查看[rfc](https://tools.ietf.org/html/draft-ietf-httpbis-rfc6265bis-02)中的`SameSite`以下[部分](https://tools.ietf.org/html/draft-ietf-httpbis-rfc6265bis-02#section-5.3.7.1)。

使用此属性的 cookie 示例：

```
Set-Cookie: JSESSIONID=xxxxx; SameSite=Strict
Set-Cookie: JSESSIONID=xxxxx; SameSite=Lax
```

现在所有桌面浏览器和几乎所有移动浏览器都支持该`SameSite`属性。要跟踪实现它的浏览器和属性的使用情况，请参阅以下[服务](https://caniuse.com/#feat=same-site-cookie-attribute)。请注意，Chrome 已经[宣布](https://blog.chromium.org/2019/10/developers-get-ready-for-new.html)他们`SameSite=Lax`将从 Chrome 80（2020 年 2 月到期）默认将 cookie 标记为默认值，Firefox 和 Edge 都计划效仿。此外，`Secure`标记为 的 cookie 将需要该标志`SameSite=None`。

需要注意的是，这个属性应该作为一个附加层*的纵深防御*概念来实现。该属性通过支持它的浏览器保护用户，并且它还包含 2 种绕过它的方法，如[下一节所述](https://tools.ietf.org/html/draft-ietf-httpbis-rfc6265bis-02#section-5.3.7.1)。此属性不应取代拥有 CSRF 令牌。相反，它应该与该令牌共存，以便以更强大的方式保护用户。

### 使用标准标题验证来源

这种缓解有两个步骤，这两个步骤都依赖于检查 HTTP 请求标头值。

1. 确定请求的来源（来源来源）。可以通过 Origin 或 Referer 标头完成。
2. 确定请求的来源（目标来源）。

在服务器端，我们验证它们是否匹配。如果他们这样做，我们接受请求是合法的（意味着它是同源请求），如果他们不这样做，我们会丢弃请求（意味着请求来自跨域）。这些标头的可靠性来自于它们不能以编程方式更改，因为它们属于[禁止的标头](https://developer.mozilla.org/en-US/docs/Glossary/Forbidden_header_name)列表，这意味着只有浏览器可以设置它们。

#### 识别来源来源（通过 Origin/Referer 标头）

##### 检查源头

如果存在 Origin 标头，请验证其值是否与目标源相匹配。与 Referer 不同，Origin 标头将出现在源自 HTTPS URL 的 HTTP 请求中。

##### 检查引用标头

如果 Origin 标头不存在，请验证 Referer 标头中的主机名与目标源匹配。这种缓解 CSRF 的方法也常用于未经身份验证的请求，例如在建立会话状态之前发出的请求，这是跟踪同步令牌所必需的。

在这两种情况下，请确保目标原点检查是强的。例如，如果您的网站`example.org`确定`example.org.attacker.com`没有通过您的来源检查（即，匹配来源之后的尾随 / 以确保您与整个来源相匹配）。

如果这些标头都不存在，您可以接受或阻止请求。我们建议**阻止**。或者，您可能希望记录所有此类实例，监控它们的用例/行为，然后只有在您获得足够的信心后才开始阻止请求。

#### 识别目标原点

您可能认为确定目标原点很容易，但通常并非如此。`#`第一个想法是简单地从请求中的 URL 中获取目标来源（即它的主机名和端口）。但是，应用程序服务器经常位于一个或多个代理之后，并且原始 URL 与应用程序服务器实际接收的 URL 不同。如果您的应用程序服务器由其用户直接访问，那么在 URL 中使用源就可以了，一切就绪。

如果您使用代理，则有许多选项需要考虑。

- **配置您的应用程序以简单地了解其目标来源：**这是您的应用程序，因此您可以找到它的目标来源并在某些服务器配置条目中设置该值。这将是最安全的方法，因为它是在服务器端定义的，因此它是一个值得信赖的值。但是，如果您的应用程序部署在许多地方（例如，开发、测试、QA、生产和可能的多个生产实例），这可能难以维护。为每种情况设置正确的值可能很困难，但是如果您可以通过一些中央配置来完成并提供您的实例以从中获取价值，那就太好了！（**注意：**确保集中配置存储得到安全维护，因为您的 CSRF 防御的主要部分依赖于它。）
- **使用 Host 标头值：**如果您希望应用程序找到自己的目标，因此不必为每个部署的实例进行配置，我们建议使用 Host 系列标头。Host 标头的目的是包含请求的目标来源。但是，如果您的应用服务器位于代理后面，则 Host 标头值很可能被代理更改为代理后面 URL 的目标源，这与原始 URL 不同。此修改后的 Host 标头来源与原始 Origin 或 Referer 标头中的源来源不匹配。
- **使用 X-Forwarded-Host 标头值：**为了避免代理更改主机标头的问题，还有另一个标头称为 X-Forwarded-Host，其目的是包含代理收到的原始 Host 标头值。大多数代理将在 X-Forwarded-Host 标头中传递原始 Host 标头值。因此，该标头值很可能是您需要与 Origin 或 Referer 标头中的源源进行比较的目标源值。

当请求中存在源或引荐标头时，此缓解措施正常工作。尽管**大部分**时间都包含这些标头，但很少有不包含它们的用例（其中大多数是出于保护用户隐私/调整浏览器生态系统的正当理由）。下面列出了一些用例：

- Internet Explorer 11 不会在跨受信任区域的站点的 CORS 请求中添加 Origin 标头。Referer 标头将仍然是 UI 来源的唯一指示。[请参阅此处](https://stackoverflow.com/questions/20784209/internet-explorer-11-does-not-add-the-origin-header-on-a-cors-request)和[此处](https://github.com/silverstripe/silverstripe-graphql/issues/118)的 Stack Overflow 中的以下参考资料。
- 在[302 重定向 cross-origin](https://stackoverflow.com/questions/22397072/are-there-any-browsers-that-set-the-origin-header-to-null-for-privacy-sensitiv)之后的实例中，重定向请求中不包含 Origin，因为这可能被视为不应发送到其他来源的敏感信息。
- 在某些[隐私上下文](https://wiki.mozilla.org/Security/Origin#Privacy-Sensitive_Contexts)中，Origin 设置为“null”例如，请参见以下[内容](https://www.google.com/search?q=origin+header+sent+null+value+site%3Astackoverflow.com&oq=origin+header+sent+null+value+site%3Astackoverflow.com)。
- Origin 标头包含在所有跨域请求中，但对于同源请求，在大多数浏览器中仅包含在 POST/DELETE/PUT**注意：**虽然并不理想，但许多开发人员使用 GET 请求进行状态更改操作。
- Referer 标头也不例外。有多个用例也省略了引用标题（[1](https://stackoverflow.com/questions/6880659/in-what-cases-will-http-referer-be-empty)、[2](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referer)、[3](https://en.wikipedia.org/wiki/HTTP_referer#Referer_hiding)、[4](https://seclab.stanford.edu/websec/csrf/csrf.pdf)和[5](https://www.google.com/search?q=referrer+header+sent+null+value+site:stackoverflow.com)）。众所周知，负载平衡器、代理和嵌入式网络设备由于在记录它们时的隐私原因会剥离引荐来源标头。

通常，一小部分流量确实属于上述类别（[1-2%](https://homakov.blogspot.com/2012/04/playing-with-referer-origin-disquscom.html)），没有企业愿意失去这些流量。在 Internet 上使用的一种流行技术使该技术更有用，如果 Origin/referrer 匹配您配置的域列表“或”一个空值（[此处](https://homakov.blogspot.com/2012/04/playing-with-referer-origin-disquscom.html)的示例。空值是为了覆盖边缘情况），则接受请求上面提到不发送这些标头的地方）。请注意，攻击者可以利用这一点，但人们更喜欢将此技术用作深度防御措施，因为部署它所涉及的工作量很小。

#### 带有 __Host- 前缀的 Cookie

这个问题的另一个解决方案是使用`Cookie Prefixes`带有 CSRF 令牌的 for cookie。如果 cookie 有`__Host-`前缀，例如`Set-Cookie: __Host-token=RANDOM; path=/; Secure`，那么 cookie：

- 不能从另一个子域（覆盖）写入。
- 必须有的路径`/`。
- 必须标记为安全（即，不能通过未加密的 HTTP 发送）。

[自 2020 年 7 月起，除 Internet Explorer 之外的所有主要浏览器都支持](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#Browser_compatibility)cookie 前缀。

有关 cookie 前缀的更多信息，请参阅[Mozilla 开发者网络](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie#Directives)和[IETF 草案](https://tools.ietf.org/html/draft-west-cookie-prefixes-05)。

### 使用自定义请求标头

添加 CSRF 令牌、双重提交 cookie 和值、加密令牌或其他涉及更改 UI 的防御措施通常很复杂或有其他问题。特别适合 AJAX 或 API 端点的另一种防御方法是使用**自定义请求标头**。这种防御依赖于[同源策略 (SOP)](https://en.wikipedia.org/wiki/Same-origin_policy)限制，即只有 JavaScript 可以用于添加自定义标头，并且只能在其源中。默认情况下，浏览器不允许 JavaScript 使用自定义标头进行跨源请求。

如果您的系统是这种情况，您可以简单地验证所有服务器端 AJAX 端点上是否存在此标头和值，以防止 CSRF 攻击。这种方法具有通常不需要更改 UI 和不引入任何服务器端状态的双重优势，这对 REST 服务特别有吸引力。如果愿意，您可以随时添加自己的**自定义标头**和值。

这种技术显然适用于 AJAX 调用，但您仍然需要`<form>`使用本文档中描述的方法（例如令牌）来保护标签。此外，CORS 配置也应该是健壮的，以使该解决方案有效地工作（因为来自其他域的请求的自定义标头会触发飞行前 CORS 检查）。

### 基于用户交互的 CSRF 防御

虽然此处引用的所有技术都不需要任何用户交互，但有时让用户参与交易以防止未经授权的操作（通过 CSRF 或其他方式伪造）更容易或更合适。以下是一些在正确实施时可以充当强大的 CSRF 防御的技术示例。

- ~~Re-Authentication~~ 授权机制（密码或更强）
- 一次性令牌
- CAPTCHA（更喜欢没有用户交互或视觉模式匹配的较新的 CAPTCHA 版本）

虽然这些是非常强大的 CSRF 防御，但它会对用户体验产生重大影响。因此，它们通常仅用于安全关键操作（例如密码更改、汇款等），以及本备忘单中讨论的其他防御措施。

## 登录 CSRF

大多数开发人员倾向于忽略登录表单上的 CSRF 漏洞，因为他们认为 CSRF 不适用于登录表单，因为用户在那个阶段没有经过身份验证，但是这种假设并不总是正确的。CSRF 漏洞仍然可能发生在用户未经过身份验证的登录表单上，但影响和风险不同。

例如，如果攻击者使用 CSRF 使用攻击者的帐户在购物网站上假设目标受害者的身份经过验证，然后受害者输入他们的信用卡信息，攻击者可能能够使用受害者存储的卡详细信息购买商品. 有关登录 CSRF 和其他风险的更多信息，请参阅本文第 3[节](https://seclab.stanford.edu/websec/csrf/csrf.pdf)。

登录 CSRF 可以通过创建预会话（用户通过身份验证之前的会话）并在登录表单中包含令牌来缓解。您可以使用上面提到的任何技术来生成令牌。请记住，一旦用户通过身份验证，预会话就不能转换为真正的会话 - 会话应该被销毁并且应该创建一个新会话以避免[会话固定攻击](http://www.acrossecurity.com/papers/session_fixation.pdf)。此技术在[跨站点请求伪造的稳健防御第 4.1 节](https://seclab.stanford.edu/websec/csrf/csrf.pdf)中进行了描述。

## Java 参考示例

以下[JEE Web 过滤器](https://github.com/righettod/poc-csrf/blob/master/src/main/java/eu/righettod/poccsrf/filter/CSRFValidationFilter.java)为本备忘单中描述的一些概念提供了示例参考。它实现了以下无状态缓解（[OWASP CSRFGuard](https://github.com/aramrami/OWASP-CSRFGuard)，涵盖有状态方法）。

- 使用标准标题验证相同的来源
- 双重提交 cookie
- SameSite cookie 属性

**请注意**，它仅作为参考样本，并不完整（例如：它没有块来指导控制流，当源头和引荐来源标头检查成功时，也没有引荐来源标头的端口/主机/协议级别验证）。建议开发人员在此参考示例的基础上构建完整的缓解措施。在检查 CSRF 被认为有效之前，开发人员还应该实施身份验证和授权机制。

完整源代码位于[此处](https://github.com/righettod/poc-csrf)并提供可运行的 POC。

## 将 CSRF 令牌自动包含为 AJAX 请求标头的 JavaScript 指南

以下指南认为**GET**、**HEAD**和**OPTIONS**方法是安全操作。因此**GET**、**HEAD**和**OPTIONS**方法 AJAX 调用不需要附加 CSRF 令牌标头。但是，如果动词用于执行状态更改操作，它们还需要 CSRF 令牌标头（尽管这是不好的做法，应该避免）。

**POST**、**PUT**、**PATCH**和**DELETE**方法是状态改变动词，应该有一个 CSRF 令牌附加到请求。以下指南将演示如何在 JavaScript 库中创建覆盖，以使 CSRF 令牌自动包含在上述状态更改方法的每个 AJAX 请求中。

### 将 CSRF 令牌值存储在 DOM 中

CSRF 令牌可以包含在`<meta>`标签中，如下所示。页面中的所有后续调用都可以从该`<meta>`标签中提取 CSRF 令牌。它也可以存储在 JavaScript 变量中或 DOM 上的任何位置。但是，不建议将其存储在 cookie 或浏览器本地存储中。

以下代码片段可用于将 CSRF 令牌作为`<meta>`标签包含在内：

```
<meta name="csrf-token" content="{{ csrf_token() }}">
```

填充内容属性的确切语法取决于您的 Web 应用程序的后端编程语言。

### 覆盖默认值以设置自定义标题

一些 JavaScript 库允许覆盖默认设置以将标头自动添加到所有 AJAX 请求中。

#### XMLHttpRequest（原生 JavaScript)

XMLHttpRequest 的 open() 方法可以被覆盖以在下次调用`anti-csrf-token`该方法时设置标头。`open()`下面定义的函数`csrfSafeMethod()`将过滤掉安全的 HTTP 方法，只将标头添加到不安全的 HTTP 方法中。

这可以按照以下代码片段所示完成：

```
<script type="text/javascript">
    var csrf_token = document.querySelector("meta[name='csrf-token']").getAttribute("content");
    function csrfSafeMethod(method) {
        // these HTTP methods do not require CSRF protection
        return (/^(GET|HEAD|OPTIONS)$/.test(method));
    }
    var o = XMLHttpRequest.prototype.open;
    XMLHttpRequest.prototype.open = function(){
        var res = o.apply(this, arguments);
        var err = new Error();
        if (!csrfSafeMethod(arguments[0])) {
            this.setRequestHeader('anti-csrf-token', csrf_token);
        }
        return res;
    };
 </script>
```

#### AngularJS

AngularJS 允许为 HTTP 操作设置默认标头。更多文档可以在 AngularJS 的[$httpProvider](https://docs.angularjs.org/api/ng/provider/$httpProvider#defaults)文档中找到。

```
<script>
    var csrf_token = document.querySelector("meta[name='csrf-token']").getAttribute("content");

    var app = angular.module("app", []);

    app.config(['$httpProvider', function ($httpProvider) {
        $httpProvider.defaults.headers.post["anti-csrf-token"] = csrf_token;
        $httpProvider.defaults.headers.put["anti-csrf-token"] = csrf_token;
        $httpProvider.defaults.headers.patch["anti-csrf-token"] = csrf_token;
        // AngularJS does not create an object for DELETE and TRACE methods by default, and has to be manually created.
        $httpProvider.defaults.headers.delete = {
            "Content-Type" : "application/json;charset=utf-8",
            "anti-csrf-token" : csrf_token
        };
        $httpProvider.defaults.headers.trace = {
            "Content-Type" : "application/json;charset=utf-8",
            "anti-csrf-token" : csrf_token
        };
      }]);
 </script>
```

此代码片段已使用 AngularJS 版本 1.7.7 进行了测试。

#### AXIOS

[Axios](https://github.com/axios/axios)允许我们为 POST、PUT、DELETE 和 PATCH 操作设置默认标题。

```
<script type="text/javascript">
    var csrf_token = document.querySelector("meta[name='csrf-token']").getAttribute("content");

    axios.defaults.headers.post['anti-csrf-token'] = csrf_token;
    axios.defaults.headers.put['anti-csrf-token'] = csrf_token;
    axios.defaults.headers.delete['anti-csrf-token'] = csrf_token;
    axios.defaults.headers.patch['anti-csrf-token'] = csrf_token;

    // Axios does not create an object for TRACE method by default, and has to be created manually.
    axios.defaults.headers.trace = {}
    axios.defaults.headers.trace['anti-csrf-token'] = csrf_token
</script>
```

此代码段已使用 Axios 版本 0.18.0 进行了测试。

#### jQuery

JQuery 公开了一个名为 $.ajaxSetup() 的 API，可用于将`anti-csrf-token`标头添加到 AJAX 请求。`$.ajaxSetup()`可以在这里找到API 文档。下面定义的函数`csrfSafeMethod()`将过滤掉安全的 HTTP 方法，只将标头添加到不安全的 HTTP 方法中。

您可以通过采用以下代码片段将 jQuery 配置为自动将令牌添加到所有请求标头中。这为基于 AJAX 的应用程序提供了简单方便的 CSRF 保护：

```
<script type="text/javascript">
    var csrf_token = $('meta[name="csrf-token"]').attr('content');

    function csrfSafeMethod(method) {
        // these HTTP methods do not require CSRF protection
        return (/^(GET|HEAD|OPTIONS)$/.test(method));
    }

    $.ajaxSetup({
        beforeSend: function(xhr, settings) {
            if (!csrfSafeMethod(settings.type) && !this.crossDomain) {
                xhr.setRequestHeader("anti-csrf-token", csrf_token);
            }
        }
    });
</script>
```

此代码片段已使用 jQuery 版本 3.3.1 进行了测试。

## 参考

### CSRF

- [OWASP 跨站请求伪造 (CSRF)](https://owasp.org/www-community/attacks/csrf)
- [PortSwigger 网络安全学院](https://portswigger.net/web-security/csrf)
- [Mozilla 网络安全备忘单](https://infosec.mozilla.org/guidelines/web_security#csrf-prevention)
- [常见的 CSRF 预防误解](https://www.nccgroup.trust/us/about-us/newsroom-and-events/blog/2017/september/common-csrf-prevention-misconceptions/)
- [跨站点请求伪造的强大防御](https://seclab.stanford.edu/websec/csrf/csrf.pdf)
- 对于 Java：OWASP [CSRF Guard](https://owasp.org/www-project-csrfguard/)或[Spring Security](https://docs.spring.io/spring-security/site/docs/3.2.0.CI-SNAPSHOT/reference/html/csrf.html)
- 对于 PHP 和 Apache：[CSRFProtector 项目](https://owasp.org/www-project-csrfprotector/)
- 对于 AngularJS：[跨站点请求伪造 (XSRF) 保护](https://docs.angularjs.org/api/ng/service/$http#cross-site-request-forgery-xsrf-protection)
