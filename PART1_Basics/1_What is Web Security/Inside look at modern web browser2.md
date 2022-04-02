# 深入了解现代网络浏览器 2

### 导航中会发生什么

上面研究了不同的进程和线程如何处理浏览器的不同部分。在该文中，深入探讨了每个进程和线程如何进行通信以显示网站。

让我们看一个简单的网页浏览用例：你在浏览器中输入一个 URL，然后浏览器从互联网上获取数据并显示一个页面。在这篇文章中，我们将重点关注用户请求站点和浏览器准备呈现页面的部分——也称为导航。![浏览器进程](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/lo3x7Zt4LZ4ltsQQjLns.png?auto=format)

正如我们在CPU、GPU、内存和多进程架构中介绍的那样，选项卡之外的所有内容都由浏览器进程处理。浏览器进程具有诸如绘制浏览器按钮和输入字段的 UI 线程、处理网络堆栈以从 Internet 接收数据的网络线程、控制对文件的访问的存储线程等线程。当您在地址栏中键入 URL 时，您的输入由浏览器进程的 UI 线程处理。

### 一个简单的导航

##### [#](https://developer.chrome.com/blog/inside-browser-part2/#step-1-handling-input)第 1 步：处理输入

当用户开始在地址栏中输入内容时，UI 线程询问的第一件事是“这是搜索查询还是 URL？”。在 Chrome 中，地址栏也是一个搜索输入字段，因此 UI 线程需要解析并决定是将您发送到搜索引擎还是您请求的站点。![处理用户输入](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/HDAB6c70Jo2IvsUl0giY.png?auto=format)

##### 第 2 步：开始导航

当用户点击回车时，UI 线程会发起网络调用以获取站点内容。加载微调器显示在选项卡的一角，网络线程通过适当的协议，如 DNS 查找和为请求建立 TLS 连接。![导航开始](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/nSD7ognQ9hNFoFOnFQlw.png?auto=format)

此时，网络线程可能会收到 HTTP 301 之类的服务器重定向标头。在这种情况下，网络线程会与服务器正在请求重定向的 UI 线程通信。然后，将发起另一个 URL 请求。

#####  第 3 步：阅读回复![HTTP 响应](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/PTmbGdEyTDdLDrAbJw4v.png?auto=format)

一旦响应主体（有效负载）开始进入，网络线程会在必要时查看流的前几个字节。响应的 Content-Type 标头应该说明它是什么类型的数据，但由于它可能丢失或错误，因此在这里进行[MIME 类型嗅探。](https://developer.mozilla.org/docs/Web/HTTP/Basics_of_HTTP/MIME_types)[如源代码](https://cs.chromium.org/chromium/src/net/base/mime_sniffer.cc?sq=package:chromium&dr=CS&l=5)中所述，这是一项“棘手的业务” 。您可以阅读评论以了解不同的浏览器如何处理内容类型/有效负载对。

如果响应是一个 HTML 文件，那么下一步就是将数据传递给渲染器进程，但如果它是一个 zip 文件或其他文件，那么这意味着它是一个下载请求，所以他们需要将数据传递给下载管理器。![MIME 类型嗅探](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/pn0zlnxoYgbyzFVKoTc9.png?auto=format)

这也是进行[安全浏览](https://safebrowsing.google.com/)检查的地方。如果域和响应数据似乎与已知的恶意站点相匹配，则网络线程会发出警报以显示警告页面。此外，[**发生跨源读取 B**](https://www.chromium.org/Home/chromium-security/corb-for-developers)[锁定](https://www.chromium.org/Home/chromium-security/corb-for-developers)[**(CORB**](https://www.chromium.org/Home/chromium-security/corb-for-developers)[ )](https://www.chromium.org/Home/chromium-security/corb-for-developers)[**检查****是**为了确保敏感**的**](https://www.chromium.org/Home/chromium-security/corb-for-developers)跨站点数据不会进入渲染器进程。

##### 第 4 步：查找渲染器进程

一旦完成所有检查并且网络线程确信浏览器应该导航到请求的站点，网络线程就会告诉 UI 线程数据已准备好。UI线程然后找到一个渲染器进程来进行网页的渲染。![查找渲染器进程](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/VAR3s7k8rIgTrfwEWMIo.png?auto=format)

由于网络请求可能需要数百毫秒才能得到响应，因此应用了加速此过程的优化。当 UI 线程在第 2 步向网络线程发送 URL 请求时，它已经知道他们正在导航到哪个站点。UI 线程尝试主动查找或启动与网络请求并行的渲染器进程。这样，如果一切按预期进行，当网络线程接收到数据时，渲染器进程已经处于待机位置。如果导航重定向跨站点，则可能不会使用此备用进程，在这种情况下，可能需要不同的进程。

##### 第 5 步：提交导航

现在数据和渲染器进程已准备就绪，从浏览器进程向渲染器进程发送 IPC 以提交导航。它还传递数据流，因此渲染器进程可以继续接收 HTML 数据。一旦浏览器进程听到在渲染器进程中发生提交的确认，导航就完成了，文档加载阶段开始了。

此时，地址栏更新，安全指示器和站点设置 UI 反映了新页面的站点信息。该选项卡的会话历史记录将被更新，因此后退/前进按钮将逐步浏览刚刚导航到的站点。为了在您关闭选项卡或窗口时促进选项卡/会话恢复，会话历史记录存储在磁盘上。![提交导航](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/kL6CLP7fLay9L99vRR3F.png?auto=format)

##### 额外步骤：初始加载完成

提交导航后，渲染器进程会继续加载资源并渲染页面。我们将在下一篇文章中详细介绍此阶段发生的事情。一旦渲染器进程“完成”渲染，它会将 IPC 发送回浏览器进程（这是在`onload`页面中的所有帧上触发所有事件并完成执行之后）。此时，UI 线程停止选项卡上的加载微调器。

我说“完成”，因为在此之后客户端 JavaScript 仍然可以加载额外的资源并呈现新视图。![页面完成加载](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/DwMkwQndYadDqnMtp8T3.png?auto=format)

##### 导航到其他站点

简单的导航就完成了！但是如果用户再次将不同的 URL 放到地址栏会发生什么？好吧，浏览器进程通过相同的步骤导航到不同的站点。但在此之前，它需要检查当前呈现的站点是否关心[`beforeunload`](https://developer.mozilla.org/docs/Web/Events/beforeunload)事件。

`beforeunload`可以创建“离开此站点？” 当您尝试离开或关闭选项卡时发出警报。选项卡内的所有内容，包括您的 JavaScript 代码，都由渲染器进程处理，因此当新的导航请求进来时，浏览器进程必须检查当前的渲染器进程。
