# SameSite Cookie 说明

了解如何明确设置跨站点 cookie 以确保您的站点安全。

本文是处理更改 cookie `SameSite`属性的系列文章的一部分。

- [SameSite Cookie 说明](https://web.dev/samesite-cookies-explained/)
- [SameSite Cookie Recipe](https://web.dev/samesite-cookie-recipes/)
- [Schemeful Samesite](https://web.dev/schemeful-samesite)

Cookie 是使您的网站处于永久状态的可用方法之一。多年来，它的功能不断发展壮大，但它留下了可能导致平台出现问题的老式问题。为了解决这个问题，浏览器（包括 Chrome、Firefox 和 Edge）的行为已被修改，以便默认设置更好地保护隐私。

每个 cookie 由`key=value`一对 cookie 和几个属性组成，这些属性控制何时何地使用 cookie。您可能已经使用这些属性来设置过期日期或将 cookie 设置为仅通过 HTTPS 发送。`Set-Cookie`服务器通过在响应中发送一个命名良好的标头来设置 cookie。您可以在“ [RFC6265bis ”中查看详细信息，但我也会在这里简要说明。](https://tools.ietf.org/html/draft-ietf-httpbis-rfc6265bis-03#section-4.1)

例如，假设您运行一个博客并希望您的用户看到“最新消息”促销。用户可以关闭促销显示，促销会消失一段时间。如果您将这些设置存储在 cookie 中，将它们设置为在一个月（260 万秒）后过期，并且仅通过 HTTPS 发送，则标头如下所示：

Set-Cookie: promo_shown=1; Max-Age=2600000; Secure

![三个 cookie 从服务器发送到浏览器的响应](https://web-dev.imgix.net/image/tcFciHGuF3MxnTr1y5ue01OGLBn2/jJ1fqcsAk9Ig3hManFBO.png?auto=format)服务器使用`Set-Cookie`标头设置 cookie。

如果页面的阅读者访问了满足这些条件的页面，即在一个月内建立了安全连接并创建了 cookie，则阅读者的浏览器将请求以下标头：将被发送。



```text
Cookie: promo_shown=1
```

![在单个请求中从浏览器发送到服务器的三个 cookie](https://web-dev.imgix.net/image/tcFciHGuF3MxnTr1y5ue01OGLBn2/Rq21WQpOZFvfgS9bbjmc.png?auto=format)浏览器`Cookie`在标头中发回一个 cookie。

您还可以`document.cookie`使用 JavaScript 添加或读取该站点上可用的 cookie。`document.cookie`您可以指定使用该键创建或覆盖 cookie。例如，在浏览器的 JavaScript 控制台中尝试以下操作：



```text
→ document.cookie = "promo_shown=1; Max-Age=2600000; Secure"
← "promo_shown=1; Max-Age=2600000; Secure"
```

`document.cookie`将打印在当前上下文中可访问的所有 cookie，每个 cookie 用分号分隔。



```text
→ document.cookie;
← "promo_shown=1; color_theme=peachpuff; sidebar_loc=left"
```

![用于访问浏览器中 cookie 的 JavaScript](https://web-dev.imgix.net/image/tcFciHGuF3MxnTr1y5ue01OGLBn2/mbV00Gy5VAPTUls0i7cM.png?auto=format)JavaScript 可`document.cookie`用于访问 cookie。

如果您查看热门网站，您会发现大多数网站都有三个或更多 cookie。在大多数情况下，这些 cookie 会在对该域的每个请求中发送，这有很多含义。对于用户而言，上传带宽通常比下载带宽更受限制，因此所有发送请求的开销都会对首字节时间 (TTFB) 产生不利影响。对您设置的 cookie 的数量和大小保持保守。`Max-Age`使用属性来防止 cookie 停留时间过长。

## 什么是第一方 Cookie 和第三方 Cookie？[#](https://web.dev/samesite-cookies-explained/#cookie-cookie)

如果您返回之前浏览的网站，您可能会发现有针对不同域的 cookie，而不仅仅是您当前访问的域。与您当前访问的站点（在您的浏览器地址栏中显示的站点）的域相匹配的cookie 称为**第一方**cookie，来自您当前访问的站点以外的域的 cookie 也称为**第三方**cookie。叫。这不是绝对的分类，而是根据用户上下文的相对分类。同一个 cookie 可以是第一方 cookie 或第三方 cookie，具体取决于用户当时所在的站点。

![从同一页面上的不同请求发送到浏览器的三个 cookie](https://web-dev.imgix.net/image/tcFciHGuF3MxnTr1y5ue01OGLBn2/zjXpDz2jAdXMT83Nm3IT.png?auto=format)Cookie 可以来自页面内的不同域。

在上面的示例中，假设您在运行的博客文章中有一张特别漂亮的猫照片，并且它`/blog/img/amazing-cat.png`托管在上面。并假设图像非常棒，以至于其他人直接在他们的网站上使用它。如果您网站的访问者过去曾访问过您的博客并`promo_shown`持有 cookie，那么当您`amazing-cat.png`浏览由他人运营的网站时，将**在您**对该图像的请求中发送一个 cookie 。在这种情况下，`promo_shown`它并不是特别有用，因为它不用于其他人运行的站点上的任何内容，它只会增加请求的开销。

如果这种行为是无意的，为什么它会这样？事实上，即使在第三方上下文中使用此机制，您也可以维护站点的状态。例如，如果您的网站中嵌入了 YouTube 视频，访问者的播放器将看到“稍后观看”选项。如果访问者已登录 YouTube，则该会话将通过第三方 cookie 在嵌入式播放器中可用。这意味着您可以使用“稍后观看”按钮立即保存您的视频，而不会提示您登录或从您正在浏览的页面移动到 YouTube。

![在 3 个不同的上下文中发送相同的 cookie](https://web-dev.imgix.net/image/tcFciHGuF3MxnTr1y5ue01OGLBn2/u9chHBLm3i27yFRwHx5W.png?auto=format)第三方上下文 cookie 在各种页面访问时发送。

Web 的文化特征之一是其本质上的开放倾向。这使许多人能够创建自己的内容和应用程序。然而，另一方面，也存在许多安全和隐私问题。跨站点请求伪造 (CSRF) 攻击利用将 cookie 附加到特定来源的请求的行为，无论是谁发起了请求。例如，当您`evil.example`访问时`your-blog.example`，可能会触发对（您的博客的域）的请求，并且您的浏览器可能愿意附加相关的 cookie。如果您的博客不注意验证这些请求，可能会采取`evil.example`删除文章或添加您自己的内容等操作。

用户越来越意识到如何使用 cookie 来跟踪多个站点的用户活动。然而，直到现在，还没有办法在 cookie 中表达这种意图。`promo_shown`Cookie 只能在第一方上下文中发送，而用于嵌入另一个站点的小部件的会话 cookie 旨在在第三方上下文中提供登录状态。存在 Cookie。

## `SameSite`使用属性来指定 cookie 的使用条款[#](https://web.dev/samesite-cookies-explained/#samesite-cookie)

[随着RFC6265bis](https://tools.ietf.org/html/draft-ietf-httpbis-cookie-same-site-00)中定义的属性的引入，`SameSite`现在可以声明 cookie 授权是否仅限于第一方或同一站点上下文。在这里，准确理解“站点”的含义是有帮助的。站点是域后缀和紧接在其前面的域部分的组合。例如，`www.web.dev`域是`web.dev`站点的一部分。

**重要条款**

如果用户正在`www.web.dev`浏览并`static.web.dev`请求图像，则这是**同一个站点**请求。

[公共后缀列表](https://publicsuffix.org/)对此进行了定义，并包括诸如此类的服务`.com`以及诸如此类的顶级域。`github.io`因此，`your-project.github.io`和`my-project.github.io`可以被视为单独的站点。

**重要条款**

如果用户正在`your-project.github.io`浏览和`my-project.github.io`请求图像，则这是一个**跨站点**请求。

将属性引入cookie`SameSite`为您提供了三种不同的方法来控制此行为。您可以不指定该属性，也可以使用 或 的值将`Strict`cookie`Lax`

`SameSite`如果您将该属性设置为`Strict`，cookie 将仅在第一方上下文中发送。在以用户为中心的条款中，只有当 cookie 站点与您浏览器 URL 栏中当前显示的站点匹配时，才会发送 cookie。这`promo_shown`适用于 cookie 设置如下：



```text
Set-Cookie: promo_shown=1; SameSite=Strict
```

当用户访问您的网站时，cookie 会按预期随您的请求一起发送。但是，如果您点击指向您网站的链接，例如来自其他网站或来自朋友的电子邮件，则第一个请求不会向您发送 cookie。如果您有与初始导航中立即可用的功能相关的 cookie，例如更改密码或购买商品，这很好，但它的`promo_shown`限制性太强。.. 读者可能希望发送一个 cookie 来反映他们的设置，即使他们点击了指向该网站的链接。

在这些情况下，最好允许这些顶级导航发送 cookie `SameSite=Lax`。回到上面的 cat 文章示例，考虑另一个站点引用您的内容的情况。其他用户直接使用您的猫的图片并提供指向您原始文章的链接。



```html
<p>この素晴らしい猫を見て！</p>
<img src="https://blog.example/blog/img/amazing-cat.png" />
<p>
  この<a href="https://blog.example/blog/cat.html">記事</a
  >を読んでみてください。
</p>
```

并且cookie设置如下。



```text
Set-Cookie: promo_shown=1; SameSite=Lax
```

如果您的读者正在浏览其他用户的博客，则在您的浏览器`amazing-cat.png`请求时**不会发送**cookie 。但是，如果读者通过链接`cat.html`访问，则该请求**包含**一个 cookie 。从这些方面可以说，影响网站显示的`Lax`cookies适用于与用户行为相关的cookies `Strict`。

**警告**

此外，就站点安全性而言，`Strict`两者都不是完整的解决方案。`Lax`Cookie 是作为用户请求的一部分发送的，应与任何其他用户输入一样对待。这意味着您需要清理和验证您的输入。不要使用 cookie 来存储看似服务器端机密信息的数据。

最后，关于不指定值的选项，该选项传统上是在所有上下文中发送 cookie 的隐式断言。并且在最新的[RFC 6265bis](https://tools.ietf.org/html/draft-ietf-httpbis-rfc6265bis-03)`SameSite=None`草案中，引入了一个新的值，现在已经明确了。换句话说，`None`您现在可以使用该值来明确传达在第三方上下文中有意发送 cookie。

![三个 cookie 标记为 None、Lax、Strict，具体取决于上下文](https://web-dev.imgix.net/image/tcFciHGuF3MxnTr1y5ue01OGLBn2/1MhNdg9exp0rKnHpwCWT.png?auto=format)将cookie上下文显式标记为`None`, 。`Lax``Strict`

如果您想提供其他网站使用的服务，例如小部件、嵌入式内容、联属网络营销计划、广告和跨站点登录，您`None`应该使用它们来清楚说明。

## 不使用 SameSite 时更改默认行为[#](https://web.dev/samesite-cookies-explained/#samesite)

`SameSite`属性被广泛支持，但不幸的是没有被开发人员广泛接受。到处发送 cookie 的开放默认行为意味着所有用例都可以工作，但它使用户面临 CSRF 和意外信息泄露的风险。为了向开发人员传达意图并为用户提供更安全的环境，IETF 的提案“[逐步改进 Cookie](https://tools.ietf.org/html/draft-west-cookie-incrementalism-00) ”提出了两个重要的变化：

- `SameSite`没有属性的 Cookie 被`SameSite=Lax`视为。
- `SameSite=None``Secure`的 cookie也需要指定，这意味着需要安全上下文。

Chrome 从 84 版开始默认实现了这种行为。[Firefox](https://groups.google.com/d/msg/mozilla.dev.platform/nx2uP0CzA9k/BNVPWDHsAQAJ)可以使用 Firefox 69 进行测试，并将成为未来的默认行为。要在 Firefox 中测试这些行为，请[`about:config`](http://kb.mozillazine.org/About:config)打开`network.cookie.sameSite.laxByDefault`并设置。[Edge](https://groups.google.com/a/chromium.org/d/msg/blink-dev/AknSSyQTGYs/8lMmI5DwEAAJ)中的默认行为也会发生变化。

本文将在其他浏览器宣布支持时更新。

### [#](https://web.dev/samesite-cookies-explained/#samesitelax)作为默认值`SameSite=Lax`

没有设置属性



```text
Set-Cookie: promo_shown=1
```

`SameSite`如果您发送的 cookie 没有指定属性...

应用默认行为



```text
Set-Cookie: promo_shown=1; SameSite=Lax
```

浏览器`SameSite=Lax`将 cookie 视为已指定。

进行此更改的目的是应用更安全的默认行为，`SameSite`但理想情况下，您应该明确设置属性，而不是依赖浏览器应用程序。这阐明了 cookie 的意图，并增加了跨不同浏览器提供一致用户体验的可能性。

**警告**

Chrome 中应用的默认行为`SameSite=Lax`比显式更宽容，因为顶级导航中的 POST 请求允许发送某些 cookie。有关详细信息，请参阅[blink-dev 公告](https://groups.google.com/a/chromium.org/d/msg/blink-dev/AknSSyQTGYs/YKBxPCScCwAJ)。这是出于临时缓解目的，`SameSite=None; Secure`应修改为使用跨站点 cookie。

### `SameSite=None`请务必为[#指定 Secure](https://web.dev/samesite-cookies-explained/#samesitenone-secure)

被拒绝



```text
Set-Cookie: widget_session=abc123; SameSite=None
```

`Secure`未指定的 Cookie 设置将被**拒绝**。

公认



```text
Set-Cookie: widget_session=abc123; SameSite=None; Secure
```

`SameSite=None`必须与`Secure`属性配对。

您可以通过将Chrome 版本 76 或更高版本`about://flags/#cookies-without-same-site-must-be-secure`以及 Firefox 版本 69 或更高版本设置为[`about:config`](http://kb.mozillazine.org/About:config).`network.cookie.sameSite.noneRequiresSecure`

我们建议您在设置新 cookie 并主动更新现有 cookie（即使它不会过期）时应用此设置。

如果您的网站依赖于提供第三方内容的服务，您还应该与您的提供商核实该服务是否已更新。您可能还需要更新您的依赖项和代码片段以反映您网站上的新行为。

所有这些更改都`SameSite`向后兼容浏览器，这些浏览器要么正确实现了先前版本的属性，要么根本不支持它们。通过将这些更改应用于 cookie，您可以有意指定使用情况，而无需依赖浏览器的默认行为。如果客户端`SameSite=None`尚未识别它，它将被忽略并视为没有设置此属性。

**警告**

许多浏览器的旧版本，包括 Chrome、Safari 和 UC 浏览器，`None`与新值不兼容，可能会忽略或限制 cookie。此行为在当前版本中已修复，但您需要查看和了解流量，了解有多少用户会受到此行为的影响。有关更多信息，请参阅Chromium 站点上的“[已知不兼容客户端](https://www.chromium.org/updates/same-site/incompatible-clients)”列表。
