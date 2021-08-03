# 概述

性能是一个老生常谈的话题，那么一个高性能的网站究竟意味着什么呢？

## 高性能的网站

* 能够留住用户
* 提高转化率
* 提升用户体验
* 提高自身价值

### 能够留住用户

这个肯定毋庸置疑的，高性能的网站比低性能的网站更能吸引和留住用户。这个也是基础，有了一定的用户量，才能够实现一定的利益价值。

所以，高性能的网站是一切实现价值的基础。

### 提高转化率

留住用户对于提高转化率至关重要。速度慢的网站对收入有负面影响，而速度快的网站显示可以提高转化率。

### 提升用户体验

在用户体验方面，速度很重要。一项消费者研究表明，对移动速度延迟的压力反应类似于看恐怖电影或解决数学问题，而且比在零售店结账时排队等候的压力更大。

当网站开始加载时，用户会等待一段时间出现内容。在此之前，没有用户体验可言。这种缺乏体验在快速连接上是短暂的。然而，在较慢的连接上，用户被迫等待。随着页面资源慢慢渗入，用户可能会遇到更多问题。

### 提升自身价值

性能是良好用户体验的基础。当网站发布大量代码时，浏览器必须使用用户数以百万计的数据计划才能下载代码。移动设备的 CPU 能力和内存有限。他们经常被我们认为是少量未优化的代码所淹没。这会导致性能不佳，从而导致无响应。

因此，在这种错综复杂的情况下，能够开发或者提升网站的性能也是一件提升自我价值感的事情。

## 提升网站性能

既然性能很重要，那么肯定要想方设法去提升性能。但是不能盲目的去进行，而是要从全局出发，针对性的去提高性能。

Google提出了一项计划**Web Vitals**，旨在根据一些指标，这些指标对于用户体验来讲至关重要。

### 衡量的重要指标

这些指标包括**加载、交互性以及视觉稳定性**，每个指标也被称之为 **core web vitals**

![](../../.gitbook/assets/image%20%2822%29.png)

**LCP（Largest Contentful Paint）**：衡量加载指标，为了提供良好的体验，LCP应在页面首次打开加载后2.5s发生。

**FID（首次输入延迟）：**测量交互性，为了提供良好的体验，FID应为100ms或者更短。

**CLS（累计布局偏移\) ：**测量视觉稳定性**，**为了提供良好的体验，CLS应该保持在0.1或者更少。

除了上述针对每个性能领域的重要指标，还有其他比较重要的指标，也是衡量性能的手段。

**TTFB（Time to First Byte）**：浏览器接收页面内容的第一个字节所需要的时间，也就是服务器响应的时间。

**FCP（First Contentful Paint）**：首次内容绘制，衡量从页面开始加载到页面任何内容展示的时间。

TTFB和FCP在页面加载方面重要的指标，也在诊断LCP的问题时有一定的帮助。

**TTI（交互时间）：**测量从页面加载到可视化呈现、其初始化脚本（若有）已加载并且能够快速可靠的响应用户输入的时间。**实验室指标**。

**TBT（总阻塞时间）**：测量FCP和TTI之间主线程被阻塞足够长的时间以防止输入响应的总时间。**实验室指标**。

上述的性能指标能够了解大部分的网站的性能特征，也是一组通用的性能指标。除此之外，针对额外的一些性能指标，还有一些自定义的性能指标。

### 自定义指标

Web 性能工作组还标准化了较低级别的 API，这些 API 可用于实现您自己的[自定义指标](https://web.dev/custom-metrics/)：

* [用户计时 API](https://w3c.github.io/user-timing/)
* [长任务 API](https://w3c.github.io/longtasks/)
* [元素计时 API](https://wicg.github.io/element-timing/)
* [导航计时API](https://w3c.github.io/navigation-timing/)
* [资源计时 API](https://w3c.github.io/resource-timing/)
* [服务器计时](https://w3c.github.io/server-timing/)

### 衡量指标的工具

有了这些性能指标之后，需要去测试这些指标的阈值，以便去优化对应的指标。

衡量这些指标的工具分为**线上**和**线下。**

**线上**指的是直接可以用一些测试工具去测，比如[Lighthouse](https://github.com/GoogleChrome/lighthouse)、[PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/)、[Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools)、[Search Console](https://search.google.com/search-console/about)、[web.dev 的测量工具](https://web.dev/measure/)、[Web Vitals Chrome 扩展](https://chrome.google.com/webstore/detail/web-vitals/ahfhijdlegdabablpippeagghigmibma)和一个新的 [Chrome UX Report](https://developers.google.com/web/tools/chrome-user-experience-report) API。

**线下**指的是采用一些测试指标的开源库，需要开发者自定义实现代码。比如[web-vitals](https://github.com/GoogleChrome/web-vitals)Javascript库，衡量每个指标类似于调用单个函数。使用[web-vitals](https://github.com/GoogleChrome/web-vitals)，需要将测试的数据汇总以及展示， [Web Vitals Report](https://web-vitals-report.web.app/)  是可以自定义的创建一个web APP，将这些收集到的数据（要传输到 [Google Analytics](https://github.com/GoogleChrome/web-vitals/#send-the-results-to-google-analytics) 的数据）。

至于一些只能在测试环境中使用的指标，也是有一些测试工具的，比如[Chrome 开发者工具](https://developers.google.com/web/tools/chrome-devtools)和[Lighthouse](https://github.com/GoogleChrome/lighthouse)

### 优化指标

有了衡量工具，那么接下来就是优化指标了，下面是针对主要的三个性能指标的优化

* [优化 LCP](https://web.dev/optimize-lcp/)
* [优化 FID](https://web.dev/optimize-fid/)
* [优化 CLS](https://web.dev/optimize-cls/)

