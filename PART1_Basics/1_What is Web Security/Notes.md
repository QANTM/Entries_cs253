# 深入了解现代网络浏览器 1

### CPU、GPU、内存和多进程架构

在这个由 4 部分组成的博客系列中，我们将深入了解 Chrome 浏览器，从高级架构到渲染管道的细节。如果您想知道浏览器如何将您的代码变成功能性网站，或者您不确定为什么建议使用特定技术来提高性能，那么本系列适合您。

作为本系列的第 1 部分，我们将了解核心计算术语和 Chrome 的多进程架构。

如果您熟悉 CPU/GPU 和进程/线程的概念，您可以跳到[浏览器架构](https://developer.chrome.com/blog/inside-browser-part1/#browser-architecture)。

### CPU & GPU 进程与线程

![硬件、操作系统、应用程序](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/9M8aKlSl3207o9C3QVVp.png?auto=format)

当您启动应用程序时，会创建一个进程。该程序可能会创建线程来帮助它工作，但这是可选的。操作系统为进程提供了一块可使用的内存“平板”，所有应用程序状态都保存在该私有内存空间中。当您关闭应用程序时，该进程也会消失，并且操作系统会释放内存。

![过程和记忆](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/x5h2ZL6SWI1vF5jSa8YB.svg)

一个进程可以要求操作系统启动另一个进程来运行不同的任务。发生这种情况时，将为新进程分配内存的不同部分。**如果**两个进程需要通信，它们可以通过使用进程间**通信****(** IPC )来**实现。**许多应用程序被设计为以这种方式工作，因此如果一个工作进程没有响应，它可以在不停止运行应用程序不同部分的其他进程的情况下重新启动。![工作进程和IPC](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/OdFbLc2ufRmkJoHinTUL.svg)

##  浏览器架构（使用进程和线程构建 Web 浏览器）

进程/线程图中的不同浏览器架构：![浏览器架构](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/BG4tvT7y95iPAelkeadP.png?auto=format)

这些不同的架构是实现细节。没有关于如何构建 Web 浏览器的标准规范。一种浏览器的方法可能与另一种完全不同。

下图中描述的 Chrome 的最新架构。

顶部是浏览器进程与处理应用程序不同部分的其他进程协调。对于渲染器进程，会创建多个进程并将其分配给每个选项卡。直到最近，Chrome 还在可能的情况下为每个选项卡提供了一个进程；现在它尝试为每个站点提供自己的进程，包括 iframe（请参阅[站点隔离](https://developer.chrome.com/blog/inside-browser-part1/#site-isolation)）。

![浏览器架构](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/JvSL0B5q1DmZAKgRHj42.png?auto=format)

## 哪个过程控制什么？

下表描述了每个 Chrome 进程及其控制的内容：

| Process and What it controls | Process and What it controls                                 |
| ---------------------------- | ------------------------------------------------------------ |
| Browser                      | Controls "chrome" part of the application including address bar, bookmarks, back and forward buttons. Also handles the invisible, privileged parts of a web browser such as network requests and file access. |
| Renderer                     | Controls anything inside of the tab where a website is displayed. |
| Plugin                       | Controls any plugins used by the website, for example, flash. |
| GPU                          | Handles GPU tasks in isolation from other processes. It is separated into different process because GPUs handles requests from multiple apps and draw them in the same surface. |

![Chrome processes](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/vl5sRzL8pFwlLSN7WW12.png?auto=format)

将浏览器的工作分成多个进程的另一个好处是安全性和沙盒。由于操作系统提供了一种限制进程权限的方法，因此浏览器可以对某些进程的某些功能进行沙箱处理。例如，Chrome 浏览器限制处理任意用户输入的进程（如渲染器进程）的任意文件访问。

因为进程有自己的私有内存空间，它们通常包含通用基础设施的副本（例如 V8，它是 Chrome 的 JavaScript 引擎）。这意味着更多的内存使用，因为它们不能像在同一进程中的线程那样共享。为了节省内存，Chrome 限制了它可以启动的进程数。限制取决于您的设备拥有多少内存和 CPU 能力，但是当 Chrome 达到限制时，它会开始在一个进程中运行来自同一站点的多个选项卡。

###  节省更多内存 - Chrome 中的服务化

相同的方法适用于浏览器进程。Chrome 正在经历架构更改，以将浏览器程序的每个部分作为服务运行，从而可以轻松拆分为不同的进程或聚合为一个。

一般的想法是，当 Chrome 在强大的硬件上运行时，它可能会将每个服务拆分为不同的进程以提供更高的稳定性，但如果它在资源受限的设备上，Chrome 会将服务整合到一个进程中以节省内存占用。在此更改之前，已在 Android 等平台上使用了类似的整合流程以减少内存使用的方法。

![Chrome 服务](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/8zHB7KNXrIKv5yAWvtBy.svg)

### 每帧渲染器进程 - 站点隔离

[站点隔离](https://developers.google.com//web/updates/2018/07/site-isolation)是 Chrome 中最近引入的一项功能，它为每个跨站点 iframe 运行单独的渲染器进程。我们一直在讨论每个选项卡模型一个渲染器进程，它允许跨站点 iframe 在单个渲染器进程中运行，并在不同站点之间共享内存空间。在同一个渲染器进程中运行 a.com 和 b.com 似乎没问题。[同源策略](https://developer.mozilla.org/docs/Web/Security/Same-origin_policy)是网络的核心安全模型；它确保一个站点在未经同意的情况下无法访问其他站点的数据。绕过此策略是安全攻击的主要目标。进程隔离是分隔站点的最有效方法。随着[熔毁和幽灵](https://developers.google.com/web/updates/2018/02/meltdown-spectre)，更明显的是，我们需要使用进程来分隔站点。自 Chrome 67 起，默认情况下在桌面上启用站点隔离，选项卡中的每个跨站点 iframe 都会获得一个单独的渲染器进程。

场地隔离示意图；指向站点内 iframe 的多个渲染器进程：

![现场隔离](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/7ilepBEw6b2yUuyABbpZ.png?auto=format)

启用站点隔离是一项多年的工程工作。站点隔离并不像分配不同的渲染器进程那么简单；它从根本上改变了 iframe 相互通信的方式。在不同进程上运行 iframe 的页面上打开 devtools 意味着 devtools 必须实施幕后工作以使其看起来无缝。即使运行一个简单的 Ctrl+F 在页面中查找单词也意味着在不同的渲染器进程中进行搜索。您可以看到浏览器工程师将 Site Isolation 的发布称为重要里程碑的原因！

###  总结

在这篇文章中，我们介绍了浏览器架构的高级视图，并介绍了多进程架构的好处。我们还介绍了与多进程架构密切相关的 Chrome 中的服务化和站点隔离。在下一篇文章中，我们将开始深入研究这些进程和线程之间发生的事情，以便显示一个网站。
