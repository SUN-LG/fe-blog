# Largest Contentful Paint (LCP)

Largest Contentful Paint (LCP)是一个重要的，用来衡量[加载速度感知]()的，以用户为中心的指标。因为它再加载时间轴上，标记出了页面主要内容加载完毕的时间点。快速的 LCP有助于用户确信页面时有用的。

一直以来，测量页面主要内容加载完成并对用户可见的速度，对web开发者是一项挑战。

旧的标准（比如Load和DOMContentLoaded）不好，因为它不一定对应着用户在屏幕上看到的内容。以及更新的以用户为中心的性能指标例如FCP仅捕获了加载体验的最开始。If a page shows a splash screen or displays a loading indicator, this moment is not very relevant to the user。

过去我们推荐性能指标的比如 [First Meaningful Paint (FMP)](https://web.dev/first-meaningful-paint/)和 Speed Index (SI)，有助于在初始渲染（FCP）后捕获更多的加载体验。但是这些指标过于复杂，难以解释，并且经常是错误的-这意味着他们仍然无法识别页面主要内容何时加载完毕。

有些时候越简单越好。基于[W3C Web Performance Working Group](https://www.w3.org/webperf/)的讨论和Google的研究，我们发现衡量页面主要内容加载完成的更准确的方法是，寻找最大元素被渲染出来的时刻。

## 什么是LCP

Largest Contentful Paint (LCP) 指标，代表可视窗口中最大的元素或内容被渲染出来的时间。

## 考虑哪些元素？

在目前的[Largest Contentful Paint API](https://wicg.github.io/largest-contentful-paint/)中规定，以下的几种元素被认为是LCP中最大的元素：

- `<img>`元素

- `<image>`元素中的`<svg>`元素

- `<video>`元素（设置了poster image）

- 通过`url()`设置了背景的元素

- 包含文本节点或内联了子文本元素的块级元素

注意，将元素限制在此有限的范围内是有意的，以便一开始就保持简单。随着更多研究的进行，将来会增加更多的元素。



## 如何确定元素的大小？

LCP中最大的元素的大小通常是指用户在可视窗口看到的大小。如果元素延伸到视口之外，或者任何元素被裁剪或出现不可见的溢出，那些部分不计入元素大小。

对于已从固有尺寸调整大小的image元素，它的大小是可见大小或固有大小中较小的那个。例如，缩小到远小于固有尺寸的图片，它的大小是现在显示的大小，而拉伸或延展成更大的图片则以固有尺寸作为元素大小。

对于文本元素，仅考虑其文本节点的大小



## LCP什么时候会被报告？

网页通常分阶段加载，结果，页面上最大的元素可能发生变化。

为了应对这种潜在的变化，



## 如何处理元素布局和大小的更改？

为了保持计算和派发新`PerformanceEntry`低开销，元素大小或位置发生变化不会生产新的LCP candidates。只考虑原始固有尺寸和视口中的位置发生变化。

这意味着最初在屏幕外渲染然后过渡到屏幕上的图像并不会被reported。同样意味着开始在视口渲染的图像被挤出屏幕，仍然会以初始或视口中的尺寸被reported。

但是，如果某个元素已从DOM中删除或图像资源发生变化，则该元素不被考虑。

### 例如：

以下是几个受欢迎的网站，LCP发生的时刻：

![lcp-cnn-filmstrip](https://cdn.img.wenhairu.com/images/2020/04/10/NaYjR.png)

![lcp-techcrunch-filmstrip](https://cdn.img.wenhairu.com/images/2020/04/10/NaWYA.png)

在上面两个时间轴，最大的元素随着内容加载不断改变。第一个例子中，新内容插入到DOM中后，最大是元素改变了。第二个例子中，页面布局发生改变之前最大的内容被移除出了视口。

通常情况下，延迟加载的内容大于已经在页面上的内容，但也不一定。下面两个例子展示了页面完全加载完之前最大的元素已经渲染出来了。

![lcp-instagram-filmstrip](
https://cdn.img.wenhairu.com/images/2020/04/10/NaDyT.png)

![lcp-google-filmstrip](https://cdn.img.wenhairu.com/images/2020/04/10/NaEXG.png)

第一个例子中，Instagram的logo相对较早的加载完了，直到其他内容逐步加载完毕，logo依然是最大元素。在Google search页的例子中，最大的元素是文字段落，它在其他图片或logo加载完成之前就被渲染了。由于所有的单个的图片都小于这个段落，它一直是页面加载过程中最大的元素。

## 使用JavaScript测量LCP

你可以再JavaScript中使用 [Largest Contentful Paint API](https://wicg.github.io/largest-contentful-paint/)测量LCP。下面的例子展示了如何创建一个 [`PerformanceObserver`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver)监听 `largest-contentful-paint`并记录LCP的值到控制台。

```javascript
// Catch errors since some browsers throw when using the new `type` option.
// https://bugs.webkit.org/show_bug.cgi?id=209216
try {
  // Create a variable to hold the latest LCP value (since it can change).
  let lcp;

  // Create the PerformanceObserver instance.
  const po = new PerformanceObserver((entryList) => {
    const entries = entryList.getEntries();
    const lastEntry = entries[entries.length - 1];

    // Update `lcp` to the latest value, using `renderTime` if it's available,
    // otherwise using `loadTime`. (Note: `renderTime` may not be available if
    // the element is an image and it's loaded cross-origin without the
    // `Timing-Allow-Origin` header.)
    lcp = lastEntry.renderTime || lastEntry.loadTime;
  });

  // Observe entries of type `largest-contentful-paint`, including buffered
  // entries, i.e. entries that occurred before calling `observe()`.
  po.observe({type: 'largest-contentful-paint', buffered: true});

  // Send the latest LCP value to your analytics server once the user
  // leaves the tab.
  addEventListener('visibilitychange', function fn() {
    if (lcp && document.visibilityState === 'hidden') {
      console.log('LCP:', lcp);
      removeEventListener('visibilitychange', fn, true);
    }
  }, true);
} catch (e) {
  // Do nothing if the browser doesn't support this API.
}
```



## 好的LCP分数是多少？

为了提供良好的用户体验，网站应努力将LCP限制在2.5秒内。

## 如何改善LCP

LCP主要受三个因素影响：

- 服务器响应时间
- CSS阻塞时间
- 资源加载时间

另外，如果你的网站是客户端渲染，最大的元素是通过JavaScript生成并加入到DOM中，你的脚本解析、编译和执行时间同样是影响LCP的因素之一。

你可以参考以下指南来优化以上因素：

- [Apply instant loading with the PRPL pattern](https://web.dev/apply-instant-loading-with-prpl)
- [优化关键渲染路径](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/)
- [优化CSS](https://web.dev/fast#optimize-your-css)
- [优化图片](https://web.dev/fast#optimize-your-images)
- [优化web Fonts](https://web.dev/fast#optimize-web-fonts)
- [优化JavaScript](https://web.dev/fast#optimize-your-javascript)

