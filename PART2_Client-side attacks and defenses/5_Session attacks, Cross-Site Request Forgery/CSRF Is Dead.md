# 跨站点请求伪造已死！

在网络上使用 Cross-Site Request Forgery 苦苦挣扎之后，我们终于有了一个合适的解决方案。网站所有者没有技术负担，没有困难的实施，部署非常简单，它是同站点 Cookie。

#### 与 Web 本身一样古老

跨站点请求伪造，也称为 CSRF 或 XSRF，基本上一直存在。它源于一个站点必须向另一个站点发出请求的简单功能。假设我在这个页面中嵌入了以下表单。

```
<form action="https://your-bank.com/transfer" method="POST" id="stealMoney">
<input type="hidden" name="to" value="Scott Helme">
<input type="hidden" name="account" value="14278935">
<input type="hidden" name="amount" value="£1,000">
```

您的浏览器会加载此页面，结果是上面的表单，然后我也使用我的页面上的一个简单的 JS 提交了该表单。

```
document.getElementById("stealMoney").submit();
```

这就是 CSRF 这个名字的由来。我正在伪造一个跨站点发送到您的银行的请求。这里真正的问题不是我发送了请求，而是您的浏览器会随它发送您的 cookie。该请求将以您目前拥有的全部权限发送，这意味着如果您登录到您的银行，您刚刚向我捐赠了 1,000 英镑。谢谢！如果您没有登录，那么请求将是无害的，因为您无法在未登录的情况下转账。目前您的银行可以通过几种方法来缓解这些 CSRF 攻击。

#### CSRF 缓解措施

我不会详细介绍这些，因为网络上有大量关于此主题的信息，但我想快速介绍它们以展示实现它们的技术要求。

##### 检查产地

收到请求时，我们可能有两条信息可供我们使用，指示请求来自何处。这些是 Origin 标头和 Referer 标头。您可以检查这些值中的一个或两个，以查看请求是否来自与您自己不同的来源。如果请求是跨域的，您只需将其丢弃。Origin 和 Referer 标头确实得到了浏览器的一些保护以防止篡改，但它们也可能并不总是存在。

```
accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
accept-encoding: gzip, deflate, br
cache-control: max-age=0
content-length: 166
content-type: application/x-www-form-urlencoded
dnt: 1
origin: https://report-uri.io
referer: https://report-uri.io/login
upgrade-insecure-requests: 1
user-agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36
```

##### 反 CSRF 代币

有两种不同的方式可以使用 Anti-CSRF 令牌，但原理保持不变。当访问者请求一个页面时，例如上面示例中的转账页面，您将一个随机令牌嵌入到表单中。当真正的用户提交此表单时，会返回随机令牌，您可以检查它是否与您在表单中发出的令牌匹配。在 CSRF 攻击场景中，攻击者永远无法获得此值，即使他们请求了该页面也无法获得它，因为同源策略 (SOP) 会阻止攻击者读取包含令牌的响应。这种方法效果很好，但需要站点跟踪 Anti-CSRF 令牌的发行和返回。一种类似的方法是将令牌嵌入到表单中并向浏览器发出包含相同值的 cookie。当真正的用户提交他们的表单时，cookie 中的值和网站收到的表单将匹配。当攻击者发送伪造请求时，浏览器不会设置 CSRF cookie，测试将失败。

```
<form action="https://report-uri.io/login/auth" method="POST">
    <input type="hidden" name="csrf_token" value="d82c90fc4a14b01224gde6ddebc23bf0">
    <input type="email" id="email" name="email">
    <input type="password" id="password" name="password">
    <button type="submit" class="btn btn-primary">Login</button>
</form>
```

#### 问题

长期以来，上述方法为我们提供了相当强大的 CSRF 保护。检查 Origin 和 Referer 标头并不是 100% 可靠的，大多数网站都采用了 Anti-CSRF 令牌方法的一些变体。麻烦的是，尽管这些都对站点提出了某种要求来实施和维护解决方案。它们可能不是世界上技术上最复杂的东西，但我们仍在构建一个解决方案来围绕浏览器做一些我们不希望它做的事情。相反，我们为什么不直接告诉浏览器停止做我们不希望它做的事情呢？...现在我们可以了！

#### 同站点 Cookie

您可能已经在我最近的名为[Tough Cookies](https://scotthelme.co.uk/tough-cookies/)的博客中看到了 Same-Site Cookies，但我将在这里通过一些示例对其进行更深入的研究。从本质上讲，Same-Site Cookies 完全有效地抵消了 CSRF 攻击。死的。菲尼托。再见！Same-Site Cookie 捕捉到了我们真正需要在网络上赢得安全战斗的精髓，部署起来非常简单，*非常*简单。获取您现有的 cookie：

```
Set-Cookie: sess=abc123; path=/
```

只需添加 SameSite 属性。

```
Set-Cookie: sess=abc123; path=/; SameSite=lax
```

你完成了。说真的，就是这样！在 cookie 上启用此属性将指示浏览器为此 cookie 提供某些保护。您可以在两种模式下启用此保护，严格或松懈，具体取决于您想要获得的严重程度。

```
SameSite=Strict
SameSite=Lax
```

##### 严格的

将 SameSite 保护设置为严格模式显然是首选，但我们有两个选项的原因是，并非所有站点都相同，也没有相同的要求。当在 Strict 模式下运行时，浏览器根本不会在任何跨域请求上发送 cookie，因此 CSRF 完全死在水中。您可能遇到的唯一问题是它也不会在顶级导航上发送 cookie（更改地址栏中的 URL）。如果我提供了指向[https://facebook.com的链接](https://facebook.com/)并且 Facebook 将 SameSite cookie 设置为严格模式，当您单击该链接以打开 Facebook 时，您将不会登录。无论您是否已经登录，在新标签中打开它，无论您做什么，您都不会从该链接访问时登录到 Facebook。这对用户来说可能有点烦人和/或意外，但确实提供了令人难以置信的强大保护。Facebook 在这里需要做的与亚马逊所做的类似，他们有 2 个 cookie。一种是一种“基本”cookie，可将您识别为用户并允许您获得登录体验，但如果您想做一些敏感的事情，例如购买或更改帐户中的某些内容，则需要第二个 cookie，即'真正的' cookie，可让您做重要的事情。在这种情况下，第一个 cookie 不会 没有设置 SameSite 属性，因为它是一个“便利”cookie，它实际上不允许您做任何敏感的事情，如果攻击者可以使用它发出跨域请求，则不会发生任何事情。然而，第二个 cookie，即敏感 cookie，将设置 SameSite 属性，并且攻击者不能在跨域请求中滥用其权限。这对于用户和安全来说都是理想的解决方案。这并不总是可行的，因为我们希望 SameSite cookie 易于部署，所以还有第二个选项。t 在跨域请求中滥用其权限。这对于用户和安全来说都是理想的解决方案。这并不总是可行的，因为我们希望 SameSite cookie 易于部署，所以还有第二个选项。t 在跨域请求中滥用其权限。这对于用户和安全来说都是理想的解决方案。这并不总是可行的，因为我们希望 SameSite cookie 易于部署，所以还有第二个选项。

##### 松懈

将 SameSite 保护设置为 Lax 模式可修复上述严格模式下用户单击链接但如果他们已经登录则未登录目标站点的问题。在 Lax 模式下，有一个例外允许 cookie附加到使用安全 HTTP 方法的顶级导航。“安全”HTTP 方法在[RFC 7321 的第 4.2.1 节](https://tools.ietf.org/html/rfc7231#section-4.2.1)中定义为 GET、HEAD、OPTIONS 和 TRACE，我们对这里的 GET 方法感兴趣。这意味着我们的顶级导航到[https://facebook.com](https://facebook.com/)当用户单击链接时，现在在浏览器发出请求时附加了 SameSite 标记的 cookie，从而保持预期的用户体验。我们仍然完全可以抵御基于 POST 的 CSRF 攻击。回到顶部的示例，这种攻击在 Lax 模式下仍然不起作用。

```
<form action="https://your-bank.com/transfer" method="POST" id="stealMoney">
<input type="hidden" name="to" value="Scott Helme">
<input type="hidden" name="account" value="14278935">
<input type="hidden" name="amount" value="£1,000">
```

因为 POST 方法被认为不安全，所以浏览器不会在请求中附加 cookie。攻击者当然可以自由地将方法更改为“安全”方法并发出相同的请求。

```
<form action="https://your-bank.com/transfer" method="GET" id="stealMoney">
<input type="hidden" name="to" value="Scott Helme">
<input type="hidden" name="account" value="14278935">
<input type="hidden" name="amount" value="£1,000">
```

只要我们不接受 GET 请求代替 POST 请求，那么这种攻击是不可能的，但是在 Lax 模式下操作时需要注意。此外，如果攻击者可以触发顶级导航或弹出新窗口，他们还可以导致浏览器发出带有 cookie 的 GET 请求。这是在 Lax 模式下操作的权衡，我们保持用户体验不变，但接受支付的风险很小。

#### 其他用途

该博客旨在通过 SameSite Cookie 缓解 CSRF，但您可能已经猜到，此机制也有其他用途。规范中列出的第一个是跨站点脚本包含 (XSSI)，它是浏览器请求资产的地方，例如脚本，该脚本将根据用户是否经过身份验证而改变。在跨站点请求场景中，攻击者不能滥用 SameSite Cookie 的环境权限来产生不同的响应。[这里](https://www.contextis.com/documents/2/Browser_Timing_Attacks.pdf)还详细介绍了一些有趣的定时攻击，它们也可以得到缓解。

另一个没有详细说明的有趣用途是防止在 BEAST 风格的压缩攻击（[CRIME](https://en.wikipedia.org/wiki/CRIME_(security_exploit))、[BREACH](https://en.wikipedia.org/wiki/BREACH_(security_exploit))、[HEIST](https://tom.vg/papers/heist_blackhat2016.pdf)、[TIME](https://www.blackhat.com/eu-13/briefings.html#Beery)）中泄露会话 cookie 的值。这确实是高级别的，但基本场景是 MiTM 可以强制浏览器通过他们喜欢的任何机制发出跨域请求并监视它们。通过滥用请求负载大小的变化，攻击者可以通过改变浏览器发出的请求并观察它们在网络上的大小来一次一个字节地猜测会话 ID 值。使用 SameSite Cookie，浏览器不会在此类请求中包含 cookie，因此攻击者无法猜测它们的值。

#### 浏览器支持

借助浏览器中的大多数新安全功能，您可以期待 Firefox 或 Chrome 引领潮流，这里的情况也没有什么不同。Chrome 从 v51 开始支持 Same-Site Cookie，这意味着 Opera、Android 浏览器和 Android 上的 Chrome 也支持。您可以在[caniuse.com](https://caniuse.com/#search=SameSite)上查看列出当前支持的详细信息，并且 Firefox 有一个[错误](https://bugzilla.mozilla.org/show_bug.cgi?id=795346)打开也可以添加支持。即使支持尚未普及，我们仍应将 SameSite 属性添加到我们的 cookie。理解它的浏览器将尊重该设置并为 cookie 提供额外保护，而那些不理解它的浏览器将简单地忽略它并继续。这里没有什么可失去的，它形成了一种非常好的纵深防御方法。我们需要很长时间才能考虑移除传统的反 CSRF 机制，但在这些机制之上添加 SameSite 可以为我们提供令人难以置信的强大防御。
