# 深入了解现代网络浏览器 4

### 输入进入合成器

这是 Chrome 内部的 4 部分博客系列的最后一篇；调查它如何处理我们的代码以显示网站。在上一篇文章中，我们查看[了渲染过程并了解了合成器](https://developers.google.com//web/updates/2018/09/inside-browser-part3)。在这篇文章中，我们将了解当用户输入进入时合成器如何实现流畅的交互。

### 从浏览器的角度输入事件

当您听到“输入事件”时，您可能只会想到输入文本框或单击鼠标，但从浏览器的角度来看，输入意味着用户的任何手势。鼠标滚轮滚动是一个输入事件，触摸或鼠标悬停也是一个输入事件。

当用户在屏幕上发生触摸等手势时，浏览器进程首先接收到该手势。但是，浏览器进程只知道该手势发生的位置，因为选项卡内的内容由渲染器进程处理。因此浏览器进程将事件类型（如`touchstart`）及其坐标发送到渲染器进程。渲染器进程通过查找事件目标并运行附加的事件侦听器来适当地处理事件。

![输入事件](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/ahDODQbpiTZX6lauff5T.png?auto=format)图 1：通过浏览器进程路由到渲染器进程的输入事件

### 合成器接收输入事件

在上一篇文章中，我们研究了合成器如何通过合成光栅化图层来平滑地处理滚动。如果页面上没有附加输入事件侦听器，则合成器线程可以创建一个完全独立于主线程的新合成框架。但是，如果某些事件侦听器附加到页面怎么办？合成器线程如何确定是否需要处理事件？

### 了解非快速滚动区域

由于运行 JavaScript 是主线程的工作，因此当页面被合成时，合成器线程将页面中附加了事件处理程序的区域标记为“非快速滚动区域”。通过拥有这些信息，如果事件发生在该区域，合成器线程可以确保将输入事件发送到主线程。如果输入事件来自该区域之外，则合成器线程继续合成新帧，而无需等待主线程。

![有限的非快速滚动区域](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/F2nDPjKxnlXxuG1SAUnt.png?auto=format)图 3：非快速滚动区域的描述输入图

### 编写事件处理程序时要注意

Web 开发中常见的事件处理模式是事件委托。由于事件冒泡，您可以在最顶层元素附加一个事件处理程序并根据事件目标委派任务。您可能已经看到或编写过如下代码。

```javascript
document.body.addEventListener('touchstart', event => {
    if (event.target === area) {
        event.preventDefault();
    }
});
```

由于您只需要为所有元素编写一个事件处理程序，因此这种事件委托模式的人机工程学很有吸引力。但是，如果您从浏览器的角度来看这段代码，现在整个页面都被标记为非快速滚动区域。这意味着即使您的应用程序不关心来自页面某些部分的输入，合成器线程也必须与主线程通信并在每次输入事件进入时等待它。因此，合成器的平滑滚动能力被打败了。

![全页非快速滚动区域](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/X46tkweWpcsUIKzfAtuh.png?auto=format)图 4：覆盖整个页面的非快速滚动区域的描述输入图

为了减轻这种情况的发生，您可以`passive: true`在事件侦听器中传递选项。这向浏览器提示您仍想在主线程中收听事件，但合成器也可以继续合成新帧。

```javascript
document.body.addEventListener('touchstart', event => {
    if (event.target === area) {
        event.preventDefault()
    }
 }, {passive: true});
```

### 检查事件是否可取消

![页面滚动](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/Y3cPoWi9S1uczLToDxcx.png?auto=format)图 5：部分页面固定为水平滚动的网页

假设您在页面中有一个框，您希望将滚动方向限制为仅水平滚动。

在指针事件中使用`passive: true`选项意味着页面滚动可以平滑，但垂直滚动可能在您想要的时间开始，`preventDefault`以限制滚动方向。`event.cancelable`您可以使用方法对此进行检查。

```javascript
document.body.addEventListener('pointermove', event => {
    if (event.cancelable) {
        event.preventDefault(); // block the native scroll
        /*
        *  do what you want the application to do here
        */
    }
}, {passive: true});
```

或者，您可以使用 CSS 规则`touch-action`来完全消除事件处理程序。

```css
#area {
  touch-action: pan-x;
}
```

### 寻找事件目标

![命中测试](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/6dN5zsCK46dMNqwkO7EG.png?auto=format)图 6：查看绘制记录的主线程询问在 xy 点上绘制了什么

当合成器线程向主线程发送输入事件时，首先要运行的是命中测试以查找事件目标。命中测试使用在渲染过程中生成的绘制记录数据来找出事件发生的点坐标下方的内容。

### 最小化事件分派到主线程

在上一篇文章中，我们讨论了我们的典型显示器如何每秒刷新 60 次屏幕，以及我们需要如何跟上节奏以实现流畅的动画。对于输入，典型的触摸屏设备每秒传递 60-120 次触摸事件，而典型的鼠标每秒传递 100 次事件。输入事件具有比我们的屏幕刷新更高的保真度。

如果像每秒 120 次这样的连续事件`touchmove`发送到主线程，那么与屏幕刷新的速度相比，它可能会触发过多的命中测试和 JavaScript 执行。

![未过滤的事件](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/1cyHbX3uaB0CCSCuX8vJ.png?auto=format)图 7：事件淹没帧时间线导致页面卡顿

为了尽量减少对主线程的过多调用，Chrome 会合并连续事件（例如`wheel`, `mousewheel`, `mousemove`, `pointermove`, `touchmove`）并将调度延迟到下一个`requestAnimationFrame`.

![合并事件](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/XRCMvR1Us631HNEg8g62.png?auto=format)图 8：与以前相同的时间线，但事件被合并和延迟

任何离散事件，如`keydown`、`keyup`、`mouseup`、`mousedown`、`touchstart`和都会`touchend`立即分派。

### 用于`getCoalescedEvents`获取帧内事件

对于大多数 Web 应用程序，合并事件应该足以提供良好的用户体验。但是，如果您正在构建诸如绘图应用程序并基于`touchmove`坐标放置路径，您可能会丢失中间坐标以绘制平滑线。在这种情况下，您可以使用`getCoalescedEvents`指针事件中的方法来获取有关这些合并事件的信息。

![getCoalescedEvents](https://wd.imgix.net/image/T4FyVKpzu4WKF1kBNvXepbi08t52/P0kK5GGwmOEBXd3TjLDq.png?auto=format)图 9：左侧平滑触摸手势路径，右侧合并受限路径

```javascript
window.addEventListener('pointermove', event => {
    const events = event.getCoalescedEvents();
    for (let event of events) {
        const x = event.pageX;
        const y = event.pageY;
        // draw a line using x and y coordinates.
    }
});
```

### 下一步

在本系列中，我们介绍了 Web 浏览器的内部工作原理。如果您从未想过为什么 DevTools 建议添加`{passive: true}`事件处理程序或为什么您可能会`async`在脚本标签中写入属性，我希望本系列文章能够阐明为什么浏览器需要这些信息来提供更快、更流畅的 Web 体验。

##### 使用灯塔

如果您想让您的代码对浏览器友好，但不知道从哪里开始，[Lighthouse](https://developers.google.com/web/tools/lighthouse/)是一种工具，可以对任何网站进行审核，并为您提供关于哪些方面做得正确以及哪些方面需要改进的报告。阅读审核列表还可以让您了解浏览器关心什么样的事情。

##### 了解如何衡量绩效

不同网站的性能调整可能会有所不同，因此衡量网站的性能并确定最适合您网站的方法至关重要。Chrome DevTools 团队几乎没有关于[如何衡量网站性能的](https://developers.google.com/web/tools/chrome-devtools/speed/get-started)教程。

##### 将功能策略添加到您的站点

如果您想采取额外措施，[Feature Policy](https://developers.google.com/web/updates/2018/06/feature-policy)是一种新的 Web 平台功能，可以在您构建项目时成为您的护栏。开启功能策略可以保证您的应用程序的某些行为并防止您犯错误。例如，如果您想确保您的应用永远不会阻止解析，您可以在同步脚本策略上运行您的应用。启用后，将`sync-script: 'none'`阻止解析器阻止 JavaScript 执行。这可以防止您的任何代码阻塞解析器，并且浏览器无需担心暂停解析器。

##### 结束

当我开始构建网站时，我几乎只关心我将如何编写我的代码以及什么可以帮助我提高工作效率。这些东西很重要，但我们还应该考虑浏览器如何获取我们编写的代码。现代浏览器已经并将继续投资于为用户提供更好的网络体验的方法。通过组织我们的代码来对浏览器友好，进而改善您的用户体验。我希望你和我一起寻求对浏览器友好！
