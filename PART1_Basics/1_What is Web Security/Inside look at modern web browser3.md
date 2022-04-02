# 深入了解现代网络浏览器 3

### 渲染器进程的内部工作

这是 4 部分博客系列中的第 3 部分，着眼于浏览器的工作原理。之前，我们介绍了[多进程架构](https://developers.google.com/web/updates/2018/09/inside-browser-part1)和[导航流程](https://developers.google.com/web/updates/2018/09/inside-browser-part2)。在这篇文章中，我们将看看渲染器进程内部发生了什么。

渲染器过程涉及 Web 性能的许多方面。由于渲染器进程内部发生了很多事情，这篇文章只是一个概述。如果您想深入挖掘，[Web Fundamentals 的性能部分](https://developers.google.com/web/fundamentals/performance/why-performance-matters/)有更多资源。

### [#](https://developer.chrome.com/blog/inside-browser-part3/#renderer-processes-handle-web-contents)渲染器进程处理 Web 内容

渲染器进程负责选项卡内发生的所有事情。在渲染器进程中，主线程处理您发送给用户的大部分代码。如果您使用 Web Worker 或 Service Worker，有时您的 JavaScript 的某些部分由工作线程处理。合成器和光栅线程也在渲染器进程内部运行，以高效、流畅地渲染页面。

渲染器进程的核心工作是将 HTML、CSS 和 JavaScript 转换为用户可以与之交互的网页。![渲染器进程](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/uIqf0QQZxF6mHPDWFEjz.png?auto=format)

### 解析

##### [#](https://developer.chrome.com/blog/inside-browser-part3/#construction-of-a-dom)DOM的构建

**当渲染器进程接收到导航的提交消息并开始接收 HTML 数据时，主**线程开始**解析**文本字符串 (HTML) 并将其转换为**文档**对象模型( **DOM** )。

DOM 是浏览器对页面的内部表示，也是 Web 开发人员可以通过 JavaScript 与之交互的数据结构和 API。

将 HTML 文档解析为 DOM 由[HTML 标准](https://html.spec.whatwg.org/)定义。您可能已经注意到，将 HTML 提供给浏览器永远不会引发错误。例如，缺少结束`</p>`标记是一个有效的 HTML。像（b 标签在 i 标签之前关闭）这样的错误标记`Hi! <b>I'm <i>Chrome</b>!</i>`被视为您编写了`Hi! <b>I'm <i>Chrome</i></b><i>!</i>`. 这是因为 HTML 规范旨在优雅地处理这些错误。如果您对这些事情是如何完成的感到好奇，您可以阅读HTML 规范的“[解析器中的错误处理和奇怪案例简介](https://html.spec.whatwg.org/multipage/parsing.html#an-introduction-to-error-handling-and-strange-cases-in-the-parser)”部分。

##### 子资源加载

网站通常使用图像、CSS 和 JavaScript 等外部资源。这些文件需要从网络或缓存中加载。主线程*可以*在解析构建DOM的过程中找到它们时一一请求，但为了加快速度，“预加载扫描器”是并发运行的。如果 HTML 文档中有类似的东西`<img>`，`<link>`预加载扫描器会查看 HTML 解析器生成的令牌，并向浏览器进程中的网络线程发送请求。

![DOM](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/qmuN5aduuEit6SZfwVOi.png?auto=format)

##### JavaScript 可以阻止解析

当 HTML 解析器找到一个`<script>`标签时，它会暂停 HTML 文档的解析，并且必须加载、解析和执行 JavaScript 代码。为什么？`document.write()`因为 JavaScript 可以通过改变整个 DOM 结构之类的东西来改变文档的形状（HTML 规范[中解析模型的概述](https://html.spec.whatwg.org/multipage/parsing.html#overview-of-the-parsing-model)有一个很好的图表）。这就是为什么 HTML 解析器必须等待 JavaScript 运行才能恢复对 HTML 文档的解析。如果您对 JavaScript 执行过程中发生的事情感到好奇，[V8 团队有关于此的演讲和博客文章](https://mathiasbynens.be/notes/shapes-ics)。

### 提示浏览器如何加载资源

Web 开发人员可以通过多种方式向浏览器发送提示，以便更好地加载资源。如果您的 JavaScript 不使用`document.write()`，您可以添加[`async`](https://developer.mozilla.org/docs/Web/HTML/Element/script#attr-async)或[`defer`](https://developer.mozilla.org/docs/Web/HTML/Element/script#attr-defer)属性到`<script>`标记。然后浏览器异步加载和运行 JavaScript 代码，并且不会阻止解析。如果合适的话，你也可以使用[JavaScript 模块。](https://developers.google.com/web/fundamentals/primers/modules)`<link rel="preload">`是一种通知浏览器当前导航肯定需要该资源并且您希望尽快下载的方法。您可以在[资源优先级 - 让浏览器为您提供帮助](https://developers.google.com/web/fundamentals/performance/resource-prioritization)中阅读更多相关信息。

### 样式计算

拥有 DOM 不足以了解页面的外观，因为我们可以在 CSS 中设置页面元素的样式。主线程解析 CSS 并确定每个 DOM 节点的计算样式。这是关于基于 CSS 选择器将哪种样式应用于每个元素的信息。`computed`您可以在DevTools 部分查看此信息。![计算风格](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/hGqtsAuYpEYX4emJd5Jw.png?auto=format)

即使您不提供任何 CSS，每个 DOM 节点也有一个计算样式。`<h1>`tag 显示得比`<h2>`tag 大，并且为每个元素定义了边距。这是因为浏览器有一个默认样式表。如果你想知道 Chrome 的默认 CSS 是什么样的，[可以在这里查看源代码](https://cs.chromium.org/chromium/src/third_party/blink/renderer/core/html/resources/html.css)。

### 布局

现在渲染器进程知道文档的结构和每个节点的样式，但这还不足以渲染页面。想象一下，您正试图通过电话向您的朋友描述一幅画。“有一个大红色圆圈和一个小蓝色方块”不足以让您的朋友知道这幅画的确切外观。

布局是寻找元素几何形状的过程。主线程遍历 DOM 和计算样式，并创建包含 xy 坐标和边界框大小等信息的布局树。布局树可能类似于 DOM 树的结构，但它只包含与页面上可见内容相关的信息。如果`display: none`已应用，则该元素不是布局树的一部分（但是，具有的元素`visibility: hidden`在布局树中）。类似地，如果`p::before{content:"Hi!"}`应用了内容为 like 的伪类，即使它不在 DOM 中，它也会包含在布局树中。

![布局](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/0JqiVwHxNab2YL6qWHbS.png?auto=format)

确定页面的布局是一项具有挑战性的任务。即使是最简单的页面布局，比如从上到下的块流，也必须考虑字体有多大以及在哪里换行，因为这些会影响段落的大小和形状；然后影响下一段需要的位置。

CSS 可以使元素浮动到一侧，掩盖溢出项，改变书写方向。可以想象，这个布局阶段任务艰巨。在 Chrome 中，整个工程师团队都在处理布局。如果你想了解他们的工作细节，[BlinkOn 会议](https://www.youtube.com/watch?v=Y5Xa4H2wtVA)上的演讲很少被记录下来，非常有趣。

### 画

拥有 DOM、样式和布局仍然不足以呈现页面。假设您正在尝试复制一幅画。您知道元素的大小、形状和位置，但您仍然必须判断绘制它们的顺序。

例如，`z-index`可能为某些元素设置，在这种情况下，按照 HTML 中编写的元素顺序绘制将导致不正确的呈现。

![z-index 失败](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/4x9etJ64cg0x4a6Ktt5T.png?auto=format)图 8：页面元素按 HTML 标记的顺序出现，由于没有考虑 z-index，导致呈现错误的图像

在此绘制步骤中，主线程遍历布局树以创建绘制记录。绘画记录是“先背景，后文字，后矩形”的绘画过程的记录。`<canvas>`如果您使用 JavaScript绘制元素，那么您可能对这个过程很熟悉。

![油漆记录](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/zs8wNimWDPhu7NIhJDcc.png?auto=format)图 9：主线程遍历布局树并生成绘制记录

### 更新渲染管道成本高昂

<video autoplay="" controls="" loop="" muted="" playsinline="" class="float-right" style="box-sizing: border-box; width: 700px;"></video>

图 10：DOM+Style、Layout 和 Paint 树的生成顺序

在渲染管道中要掌握的最重要的事情是，在每个步骤中，前一个操作的结果都用于创建新数据。例如，如果布局树发生变化，则需要为文档的受影响部分重新生成绘制顺序。

如果您正在为元素设置动画，浏览器必须在每一帧之间运行这些操作。我们的大多数显示器每秒刷新屏幕 60 次 (60 fps)；当您在每一帧都在屏幕上移动物体时，动画对人眼来说会显得很流畅。但是，如果动画错过了中间的帧，那么页面将出现“janky”。

![jage jank 丢失帧](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/b3nyw5eLlDIM7rl9bxFC.png?auto=format)图 11：时间轴上的动画帧

即使你的渲染操作跟上屏幕刷新，这些计算也在主线程上运行，这意味着当你的应用程序运行 JavaScript 时它可能会被阻塞。

![JavaScript 的 jage jank](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/FryonpF90Ei9JYYGi1UI.png?auto=format)图 12：时间轴上的动画帧，但一帧被 JavaScript 阻止

您可以将 JavaScript 操作分成小块，并使用`requestAnimationFrame()`. 有关此主题的更多信息，请参阅[优化 JavaScript 执行](https://developers.google.com/web/fundamentals/performance/rendering/optimize-javascript-execution)。您也可以[在 Web Workers 中运行 JavaScript](https://www.youtube.com/watch?v=X57mh8tKkgE)以避免阻塞主线程。

![请求动画帧](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/ypzLFiu34WCuhNHm7F0B.png?auto=format)图 13：在带有动画帧的时间轴上运行的较小的 JavaScript 块

### 合成

##### 你将如何绘制页面？

<video autoplay="" controls="" loop="" muted="" playsinline="" style="box-sizing: border-box; width: 700px;"></video>

图 14：原始光栅过程的动画

既然浏览器知道了文档的结构、每个元素的样式、页面的几何形状以及绘制顺序，那么它是如何绘制页面的呢？将此信息转换为屏幕上的像素称为光栅化。

也许处理这个问题的一种天真的方法是在视口内光栅化部分。如果用户滚动页面，则移动光栅框架，并通过光栅更多来填充缺失的部分。这就是 Chrome 在首次发布时处理光栅化的方式。然而，现代浏览器运行一个更复杂的过程，称为合成。

##### 什么是合成

<video autoplay="" controls="" loop="" muted="" playsinline="" style="box-sizing: border-box; width: 700px;"></video>

图 15：合成过程动画

合成是一种技术，可以将页面的各个部分分成图层，分别将它们光栅化，然后在称为合成器线程的单独线程中合成为页面。如果发生滚动，由于图层已经被光栅化，它所要做的就是合成一个新的帧。可以通过移动图层并合成新帧以相同的方式实现动画。

[您可以使用“层”面板](https://blog.logrocket.com/eliminate-content-repaints-with-the-new-layers-panel-in-chrome-e2c306d4d752?gi=cd6271834cea)在 DevTools 中查看您的网站是如何划分层的。

##### 分层

为了找出哪些元素需要在哪些层中，主线程遍历布局树来创建层树（这部分在 DevTools 性能面板中称为“更新层树”）。如果页面的某些部分应该是单独的层（如滑入式侧边菜单）没有得到一个，那么您可以通过使用`will-change`CSS 中的属性来提示浏览器。

![层树](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/V667Geh9MtTviJjDkGZq.png?auto=format)图 16：主线程遍历布局树生成层树

您可能很想为每个元素赋予图层，但是在过多的图层上进行合成可能会导致比每帧光栅化页面的小部分更慢的操作，因此测量应用程序的渲染性能至关重要。有关该主题的更多信息，请参阅[坚持仅合成器属性和管理层数](https://developers.google.com/web/fundamentals/performance/rendering/stick-to-compositor-only-properties-and-manage-layer-count)。

##### 主线程的光栅和合成

一旦创建了层树并确定了绘制顺序，主线程就会将该信息提交给合成器线程。合成器线程然后光栅化每一层。一个层可能像一页的整个长度一样大，因此合成器线程将它们分成小块并将每个小块发送到光栅线程。光栅线程光栅化每个图块并将它们存储在 GPU 内存中。

![光栅](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/SL4KO5UsGgBNLrOwb0wC.png?auto=format)图 17：创建图块位图并发送到 GPU 的光栅线程

合成器线程可以优先考虑不同的光栅线程，以便可以首先对视口（或附近）内的事物进行光栅化。一个图层还具有针对不同分辨率的多个平铺，以处理诸如放大操作之类的事情。

一旦瓦片被光栅化，合成器线程收集称为**绘制四边形**的瓦片信息以创建**合成器框架**。

| 绘制四边形 | 考虑到页面合成，包含诸如图块在内存中的位置以及在页面中绘制图块的位置等信息。 |
| ---------- | ------------------------------------------------------------ |
| 合成器框架 | 代表页面框架的绘制四边形集合。                               |

然后通过 IPC 将合成器框架提交给浏览器进程。此时，可以从 UI 线程添加另一个合成器框架以更改浏览器 UI，或从其他渲染器进程添加用于扩展。这些合成器帧被发送到 GPU 以在屏幕上显示。如果出现滚动事件，合成器线程会创建另一个合成器帧以发送到 GPU。

![合成](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/tG4AzFeS3IdfTSawnFL6.png?auto=format)图 18：合成器线程创建合成帧。帧被发送到浏览器进程然后到 GPU

合成的好处是它是在不涉及主线程的情况下完成的。合成器线程不需要等待样式计算或 JavaScript 执行。这就是为什么[只合成动画](https://www.html5rocks.com/en/tutorials/speed/high-performance-animations/)被认为是流畅性能的最佳选择。如果需要再次计算布局或绘制，则必须涉及主线程。

在这篇文章中，我们研究了从解析到合成的渲染管道。希望您现在能够阅读有关网站性能优化的更多信息。
