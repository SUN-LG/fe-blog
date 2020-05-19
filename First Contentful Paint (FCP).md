# First Contentful Paint (FCP)



## 什么是FCP？

The First Contentful Paint (FCP) 指标是衡量从页面开始加载到屏幕上渲染出任何一部分页面内容的时间。对于这个指标，’content‘代表文字、图片、svg、或者非空白的canvas。

![Naq6S.png](https://cdn.img.wenhairu.com/images/2020/04/10/NaNZs.png)

上面的load timeline中，FCP发生在第二帧，因为那是第一个文本和图像被渲染到屏幕上。

你会注意到，尽管一部分内容已经呈现，但并不是所有的都渲染出来了。这是FCP和LCP之间一个重要的区分点。

> LCP 目的是衡量页面主要原素加载完成。



## 如何测了FCP

你可以使用 [Paint Timing API](https://w3c.github.io/paint-timing/)在JavaScript中测量FCP，下面的例子展示了如何创建[`PerformanceObserver`](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver) 监听整个paint timing，并记录`first-contentful-paint`第一次开始的时间到控制台。

```javascript
// Catch errors since some browsers throw when using the new `type` option.
// https://bugs.webkit.org/show_bug.cgi?id=209216
try {
  // Create the Performance Observer instance.
  const observer = new PerformanceObserver((list) => {
    for (const entry of list.getEntriesByName('first-contentful-paint')) {
      // Log the value of FCP to the console.
      console.log('FCP:', entry.startTime);
      observer.disconnect();
    }
  });

  // Start observing paint entry types.
  observer.observe({
    type: 'paint',
    buffered: true,
  });
} catch (e) {
  // Do nothing if the browser doesn't support this API.
}
```



## 如何提升FCP

了解如何改善特定站点的FCP，你可以运行Lighthouse性能测试，并关注特定的Opportunities和Diagnostics建议。

了解如何整体上改善FPC（任何站点），请参考以下性能指南：

- [消除渲染阻塞资源](https://web.dev/render-blocking-resources/)
- [压缩CSS](https://web.dev/unminified-css/)
- [删除未使用的CSS](https://web.dev/unused-css-rules/)
- [Preconnect to required origins](https://web.dev/uses-rel-preconnect/)
- [缩减服务响应时间(TTFB)](https://web.dev/time-to-first-byte/)
- [避免多次页面重定向](https://web.dev/redirects/)
- [预加载关键资源](https://web.dev/uses-rel-preload/)
- [避免大量的网络负载](https://web.dev/total-byte-weight/)
- [通过有效的缓存策略提供静态服务](https://web.dev/uses-long-cache-ttl/)
- [避免DOM过大](https://web.dev/dom-size/)
- [最小化关键请求深度](https://web.dev/critical-request-chains/)
- [确保文本再webfont加载期间可见](https://web.dev/font-display/)
- [保存最小的请求数和最小传输量](https://web.dev/resource-summary/)

