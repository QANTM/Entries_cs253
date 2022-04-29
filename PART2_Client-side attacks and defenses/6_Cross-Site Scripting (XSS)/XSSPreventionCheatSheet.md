# 跨站点脚本预防备忘单

## 介绍

此备忘单提供了防止 XSS 漏洞的指导。

跨站点脚本 (XSS) 用词不当。该名称起源于攻击的早期版本，其中跨站点窃取数据是主要关注点。从那时起，它已经扩展到包括基本上任何内容的注入，但我们仍然将其称为 XSS。XSS 很严重，可能导致帐户冒充、观察用户行为、加载外部内容、窃取敏感数据等。

**此备忘单是防止或限制 XSS 影响的技术列表。没有一种技术可以解决 XSS。使用正确的防御技术组合对于防止 XSS 是必要的。**

## 框架安全

使用现代 Web 框架构建的应用程序中出现的 XSS 错误更少。这些框架引导开发人员采用良好的安全实践，并通过使用模板、自动转义等来帮助缓解 XSS。也就是说，开发人员需要注意在不安全地使用框架时可能出现的问题，例如：

- 框架用来直接操作 DOM 的*逃生舱口*
- React`dangerouslySetInnerHTML`没有清理 HTML
- 没有专门的验证， React 无法处理`javascript:`或`data:`URL
- 角的`bypassSecurityTrustAs*`功能
- 模板注入
- 过时的框架插件或组件
- 和更多

了解您的框架如何防止 XSS 以及它在哪里存在差距。有时您需要在框架提供的保护之外做一些事情。这就是输出编码和 HTML 清理至关重要的地方。OWASP 正在为 React、Vue 和 Angular 制作特定于框架的备忘单。

## XSS 防御理念[¶](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html#xss-defense-philosophy)

要使 XSS 攻击成功，攻击者需要在网页中插入和执行恶意内容。Web 应用程序中的每个变量都需要受到保护。确保**所有变量**都经过验证，然后被转义或消毒，这被称为完美的抗注入性。任何未经过此过程的变量都是潜在的弱点。框架使确保变量得到正确验证和转义或清理变得容易。

然而，框架并不完美，在 React 和 Angular 等流行框架中仍然存在安全漏洞。输出编码和 HTML 清理有助于解决这些差距。

## 输出编码

当您需要完全按照用户输入的数据安全地显示数据时，建议使用输出编码。变量不应被解释为代码而不是文本。本节介绍了每种形式的输出编码，在哪里使用它，以及在哪里完全避免使用动态变量。

当您希望在用户输入数据时显示数据时，首先使用框架的默认输出编码保护。大多数框架都内置了自动编码和转义功能。

如果您不使用框架或需要弥补框架中的空白，那么您应该使用输出编码库。用户界面中使用的每个变量都应通过输出编码函数传递。输出编码库列表包含在附录中。

有许多不同的输出编码方法，因为浏览器解析 HTML、JS、URL 和 CSS 的方式不同。使用错误的编码方法可能会引入弱点或损害应用程序的功能。

### “HTML 上下文”的输出编码

“HTML 上下文”是指在两个基本 HTML 标记（如 a`<div>`或`<b>`. 例如..

```
<div> $varUnsafe </div>
```

攻击者可以修改呈现为`$varUnsafe`. 例如，这可能会导致将攻击添加到网页中。

```
<div> <script>alert`1`</script> </div> // Example Attack
```

为了安全地将变量添加到 HTML 上下文，请在将变量添加到 Web 模板时对该变量使用 HTML 实体编码。

以下是特定字符的编码值的一些示例。

如果您使用 JavaScript 写入 HTML，请查看该`.textContent`属性，因为它是一个**安全接收**器，并且会自动进行 HTML 实体编码。

```
&    &amp;
<    &lt;
>    &gt;
"    &quot;
'    &#x27;
```

### “HTML 属性上下文”的输出编码

“HTML 属性上下文”是指在 HTML 属性值中放置一个变量。您可能希望这样做来更改超链接、隐藏元素、为图像添加替代文本或更改内联 CSS 样式。您应该将 HTML 属性编码应用于放置在大多数 HTML 属性中的变量。**安全接收器**部分提供了安全 HTML 属性列表。

```
<div attr="$varUnsafe">
<div attr=”*x” onblur=”alert(1)*”> // Example Attack
```

使用引号`"`或`'`将变量括起来至关重要。引用使更改变量操作的上下文变得困难，这有助于防止 XSS。引用还显着减少了您需要编码的字符集，使您的应用程序更可靠并且编码更易于实现。

如果您使用 JavaScript 写入 HTML 属性，请查看`.setAttribute`和`[attribute]`方法，因为它们是**安全接收器**，并且会自动进行 HTML 属性编码。

### “JavaScript 上下文”的输出编码

“JavaScript 上下文”是指将变量放入内联 JavaScript，然后嵌入到 HTML 文档中。这在大量使用嵌入在其网页中的自定义 JavaScript 的程序中很常见。

在 JavaScript 中放置变量的唯一“安全”位置是在“引用数据值”内。所有其他上下文都是不安全的，您不应在其中放置可变数据。

“引用数据值”示例

```
<script>alert('$varUnsafe’)</script>
<script>x=’$varUnsafe’</script>
<div onmouseover="'$varUnsafe'"</div>
```

`\xHH`使用该格式对所有字符进行编码。编码库通常有一个`EncodeForJavaScript`或类似的来支持这个功能。

请查看[OWASP Java Encoder JavaScript 编码示例](https://owasp.org/www-project-java-encoder/)，了解需要最少编码的正确 JavaScript 使用示例。

对于 JSON，验证`Content-Type`标头是否`application/json`为`text/html`防止 XSS。

### “CSS 上下文”的输出编码

“CSS 上下文”是指放置在内联 CSS 中的变量。当您希望用户能够自定义其网页的外观和感觉时，这很常见。CSS 出奇地强大，已被用于多种类型的攻击。变量只能放在 CSS 属性值中。其他“CSS 上下文”是不安全的，您不应该在其中放置可变数据。

```
<style> selector { property : $varUnsafe; } </style>
<style> selector { property : "$varUnsafe"; } </style>
<span style="property : $varUnsafe">Oh no</span>
```

如果您使用 JavaScript 更改 CSS 属性，请查看使用`style.property = x`. 这是一个**安全接收**器，会自动对其中的数据进行 CSS 编码。

// 添加 CSS 编码建议

### “URL 上下文”的输出编码

“URL 上下文”是指放置在 URL 中的变量。最常见的是，开发人员会将参数或 URL 片段添加到 URL 库，然后在某些操作中显示或使用。对这些场景使用 URL 编码。

```
<a href="http://www.owasp.org?test=$varUnsafe">link</a >
```

`%HH`使用编码格式对所有字符进行编码。确保所有属性都被完全引用，与 JS 和 CSS 相同。

#### 常见的错误

在某些情况下，您会在不同的上下文中使用 URL。最常见的方法是将其添加到标签的`href`或`src`属性中。`<a>`在这些情况下，您应该先进行 URL 编码，然后再进行 HTML 属性编码。

```
url = "https://site.com?data=" + urlencode(parameter)
<a href='attributeEncode(url)'>link</a>
```

如果您使用 JavaScript 构建 URL 查询值，请考虑使用`window.encodeURIComponent(x)`. 这是一个**安全接收**器，会自动对其中的数据进行 URL 编码。

### 危险的环境

输出编码并不完美。它不会总是阻止 XSS。这些位置被称为**危险环境**。危险环境包括：

```
<script>Directly in a script</script>
<!-- Inside an HTML comment -->
<style>Directly in CSS</style>
<div ToDefineAnAttribute=test />
<ToDefineATag href="/test" />
```

其他需要注意的领域包括：

- 回调函数
- URL 在代码中处理的地方，例如这个 CSS { background-url : “javascript:alert(xss)”; }
- 所有 JavaScript 事件处理程序 ( `onclick()`, `onerror()`, `onmouseover()`)。
- 不安全的 JS 函数，如`eval()`, `setInterval()`,`setTimeout()`

不要将变量放入危险的上下文中，即使使用输出编码，它也不会完全阻止 XSS 攻击。

## HTML 清理

有时用户需要编写 HTML。一种情况是允许用户在所见即所得编辑器中更改内容的样式或结构。此处的输出编码将防止 XSS，但会破坏应用程序的预期功能。不会呈现样式。在这些情况下，应该使用 HTML 清理。

HTML Sanitization 将从变量中去除危险的 HTML 并返回一个安全的 HTML 字符串。OWASP 建议使用[DOMPurify](https://github.com/cure53/DOMPurify)进行 HTML 清理。

```
let clean = DOMPurify.sanitize(dirty);
```

还有一些事情需要考虑：

- 如果您对内容进行清理，然后再对其进行修改，则很容易使您的安全工作无效。
- 如果您清理内容然后将其发送到库以供使用，请检查它是否不会以某种方式改变该字符串。否则，同样，您的安全努力是无效的。
- 您必须定期修补您使用的 DOMPurify 或其他 HTML 清理库。浏览器更改功能并定期发现绕过。

## 安全水槽

安全专家经常谈论来源和接收器。如果你污染了一条河流，它就会流向下游的某个地方。计算机安全也是如此。XSS 接收器是将变量放入网页的地方。

值得庆幸的是，许多可以放置变量的接收器都是安全的。这是因为这些接收器将变量视为文本并且永远不会执行它。尝试重构您的代码以删除对像 innerHTML 这样的不安全接收器的引用，而改用 textContent 或 value。

```
elem.textContent = dangerVariable;
elem.insertAdjacentText(dangerVariable);
elem.className = dangerVariable;
elem.setAttribute(safeName, dangerVariable);
formfield.value = dangerVariable;
document.createTextNode(dangerVariable);
document.createElement(dangerVariable);
elem.innerHTML = DOMPurify.sanitize(dangerVar);
```

**安全 HTML 属性包括：** `align` , `alink`, `alt`, `bgcolor`, `border`, `cellpadding`, `cellspacing`, `class`, `color`, `cols`, `colspan`, `coords`, `dir`, `face`, `height`, `hspace`, `ismap`, `lang`, `marginheight`, `marginwidth`, `multiple`, `nohref`, `noresize`, , `noshade`, `nowrap`, `ref`, `rel`, `rev`, `rows`, `rowspan`, `scrolling`, `shape`, `span`, `summary`, `tabindex`, `title`, `usemap`, `valign`, `value`, `vlink`, .`vspace``width`

如需完整列表，请查看[DOMPurify 许可名单](https://github.com/cure53/DOMPurify/blob/main/src/attrs.js)

## 其他控件

框架安全保护、输出编码和 HTML 清理将为您的应用程序提供最佳保护。OWASP 在所有情况下都推荐这些。

除上述之外，考虑采用以下控制措施。

- Cookie 属性 - 这些更改 JavaScript 和浏览器与 cookie 交互的方式。Cookie 属性试图限制 XSS 攻击的影响，但不会阻止恶意内容的执行或解决漏洞的根本原因。
- 内容安全策略 - 阻止加载内容的允许列表。实施很容易出错，因此它不应该是您的主要防御机制。使用 CSP 作为额外的防御层，并在[此处查看备忘](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)单。
- Web 应用程序防火墙 - 这些会查找已知的攻击字符串并阻止它们。WAF 不可靠，并且定期发现新的旁路技术。WAF 也不能解决 XSS 漏洞的根本原因。此外，WAF 还遗漏了一类仅在客户端运行的 XSS 漏洞。不建议使用 WAF 来防止 XSS，尤其是基于 DOM 的 XSS。

### XSS 防护规则总结[¶](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html#xss-prevention-rules-summary)

以下 HTML 片段演示了如何在各种不同的上下文中安全地呈现不受信任的数据。

| 数据类型 | 语境                               | 代码示例                                                     | 防御                                                         |
| :------- | :--------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 细绳     | HTML 正文                          | `<span>UNTRUSTED DATA </span>`                               | HTML 实体编码（规则 #1）。                                   |
| 细绳     | 安全 HTML 属性                     | `<input type="text" name="fname" value="UNTRUSTED DATA ">`   | 激进的 HTML 实体编码（规则 #2），仅将不受信任的数据放入安全属性列表（如下所列），严格验证背景、ID 和名称等不安全属性。 |
| 细绳     | 获取参数                           | `<a href="/site/search?value=UNTRUSTED DATA ">clickme</a>`   | URL 编码（规则 #5）。                                        |
| 细绳     | SRC 或 HREF 属性中的不受信任的 URL | `<a href="UNTRUSTED URL ">clickme</a> <iframe src="UNTRUSTED URL " />` | 规范化输入、URL 验证、安全 URL 验证、仅允许列表 http 和 HTTPS URL（避免使用 JavaScript 协议打开新窗口）、属性编码器。 |
| 细绳     | CSS 值                             | `HTML <div style="width: UNTRUSTED DATA ;">Selection</div>`  | 严格的结构验证（规则 #4），CSS 十六进制编码，良好的 CSS 功能设计。 |
| 细绳     | JavaScript 变量                    | `<script>var currentValue='UNTRUSTED DATA ';</script> <script>someFunction('UNTRUSTED DATA ');</script>` | 确保引用 JavaScript 变量、JavaScript 十六进制编码、JavaScript Unicode 编码、避免反斜杠编码（`\"`或`\'`或`\\`）。 |
| HTML     | HTML 正文                          | `<div>UNTRUSTED HTML</div>`                                  | HTML 验证（JSoup、AntiSamy、HTML Sanitizer...）。            |
| 细绳     | DOM XSS                            | `<script>document.write("UNTRUSTED INPUT: " + document.location.hash );<script/>` | [基于 DOM 的 XSS 预防备忘单](https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html) |

### 输出编码规则汇总[¶](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html#output-encoding-rules-summary)

输出编码（因为它与跨站点脚本相关）的目的是将不受信任的输入转换为安全的形式，在这种形式中，输入作为**数据**显示给用户，而不是在浏览器中作为**代码执行。**以下图表详细列出了停止跨站点脚本所需的关键输出编码方法。

| 编码类型         | 编码机制                                                     |
| :--------------- | :----------------------------------------------------------- |
| HTML 实体编码    | 转换`&`为`&`，转换`<`为`<`，转换`>`为`>`，转换`"`为`"`，转换`'`为`'`，转换`/`为`/` |
| HTML 属性编码    | 除字母数字字符外，使用 HTML 实体格式对所有字符进行编码`&#xHH;`，包括空格。（**HH** = 十六进制值） |
| 网址编码         | 标准百分比编码，请参见[此处](https://www.w3schools.com/tags/ref_urlencode.asp)。URL 编码应该只用于编码参数值，而不是整个 URL 或 URL 的路径片段。 |
| JavaScript 编码  | 除字母数字字符外，使用`\uXXXX`unicode 编码格式（**X** = Integer）对所有字符进行编码。 |
| CSS 十六进制编码 | CSS 编码支持`\XX`和`\XXXXXX`. 如果下一个字符继续编码序列，则使用两个字符编码可能会导致问题。有两种解决方案：（a）在 CSS 编码后添加一个空格（将被 CSS 解析器忽略）（b）通过零填充值来使用可能的全部 CSS 编码。 |

## 相关文章

**XSS 攻击备忘单：**

以下文章描述了如何利用本文创建的不同类型的 XSS 漏洞来帮助您避免：

- OWASP：[XSS 过滤器规避备忘单](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html)。

**XSS漏洞描述：**

- OWASP 关于[XSS](https://owasp.org/www-community/attacks/xss/)漏洞的文章。

**XSS漏洞类型讨论：**

- [跨站点脚本的类型](https://owasp.org/www-community/Types_of_Cross-Site_Scripting)。

**如何查看跨站点脚本漏洞的代码：**

- [OWASP 代码审查指南](https://owasp.org/www-project-code-review-guide/)文章关于[审查跨站点脚本](https://wiki.owasp.org/index.php/Reviewing_Code_for_Cross-site_scripting)漏洞的代码。

**如何测试跨站脚本漏洞：**

- 关于测试跨站点脚本漏洞的[OWASP 测试指南文章。](https://owasp.org/www-project-web-security-testing-guide/)
- [XSS 实验性最小编码规则](https://wiki.owasp.org/index.php/XSS_Experimental_Minimal_Encoding_Rules)
