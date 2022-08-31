# 在google浏览器遇到的问题
## 遇到google浏览器使用axios下载s3存储桶的文件的cros问题

https://stackoverflow.com/questions/68953079/axios-cors-not-working-on-chrome-deployed-site
在这个链接可以看到解决方法


## 重现该问题的步骤如下：

打开在 Chromium 核心上运行的浏览器。其中使用最广泛的是 Chromium、Google Chrome 和 Microsoft Edge。

我将使用 Google Chrome 来演示它。

打开一个随机网站。例如——这个。

在浏览器 devtools 中打开控制台。

从具有 CORS 规范的不同主机中选择图像 url。重要的是来自不同的主机，并且不返回Access-Control-Allow-Origin: *标头，因此我们可以触发 CORS 检查。

几乎所有 s3 托管的映像都会发生这种情况。

我的例子是这个网址：https ://chrome-cors-testing.s3.eu-central-1.amazonaws.com/hacksoft.svg

使用 JavaScript 加载图像：
``` 
const url = "https://chrome-cors-testing.s3.eu-central-1.amazonaws.com/hacksoft.svg";
const image = new Image();
image.src = url;
```
再次加载图像，但这次添加Access-Control-Allow-Origin: *标题。通过添加图像属性来做到这一点.crossOrigin = "Anonymous"
```js
const corsImage = new Image();
corsImage.crossOrigin = "Anonymous";
corsImage.src = url;
```
结果出现跨域问题

请注意，我们第二次尝试加载图像时 - Chrome 返回一个 CORS 错误而不是响应对象。

## 原因

那么，为什么当使用 CORS 标头访问 url 时，谷歌浏览器会抛出错误呢？

好吧，首先，您应该知道网站为什么使用 CORS 策略。有一篇很好的文章解释了这一点https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS。

让我们探索浏览器如何获取图像和资源。
它发送带有特定标头的图像的 GET 请求。然后它会下载图像，然后将其缓存以供进一步使用。
在加载任何图像之前，它首先检查缓存，看看它是否已经在某个时候下载了它。如果它在那里找到图像 - 浏览器不会发送图像的 GET 请求，而只是从缓存中获取图像并将其返回给您。当您经常访问同一网站时，这可以节省加载时间和网络数据。

我们在这里遇到的问题与 Chromium 缓存图像的方式有关，并且在基于不同引擎的浏览器中似乎不会发生：

问题来自 Chromium 缓存图像的方式。
缓存初始图像的方式是 - 没有 CORS 标头。因此，下次当我们想要使用 CORS 标头获取图像时，Chromium 会尝试从缓存中提供图像。

问题是我们第一次获取图像时没有 CORS 标头（当您浏览网站并看到在<img>标签中呈现的图像时可能会发生这种情况）。

而且由于图像最初没有 CORS 标头，现在有了它们 - Chromium 返回一个 CORS 错误。

这是 Chromium 中的一个众所周知的问题，并且已在 Chromium 错误跟踪软件中进行了描述：https ://bugs.chromium.org/p/chromium/issues/detail?id=409090

然而，致力于 Chromium 的开发团队将此问题标记为WontFix(Closed)因为这可能是 Chromium 引擎的预期行为。

## 解决方案
为了解决这个问题，我们可以在获取所需图像时简单地在 url 中添加一个虚拟 GET 参数。这将迫使浏览器不使用之前缓存的图像，而是发送一个新的 GET 请求来获取图像，因为 URL 现在与 Chromium 缓存的不同。

```javascript
const corsImageModified = new Image();
corsImageModified.crossOrigin = "Anonymous";
corsImageModified.src = url + "?not-from-cache-please";
```

您添加的 GET 参数无关紧要，只要生成的 URL 与初始（缓存）图像 URL 不同。

只需添加一个虚拟 GET 参数，您将获得所需的相同图像，但这次 Chromium 将为其发送一个新请求，其中包含 CORS 标头。

该解决方案的另一个优点是它不会打扰所有其他浏览器。因此，它将修复您的用户在 Chrome、Edge 和 Chromium 中遇到的错误，而不会影响您所有其他用户的体验。