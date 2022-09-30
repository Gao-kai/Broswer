## CSP（内容安全策略）

### CSP是什么？
CSP全称为Content Secunty Policy，也就是内容安全策略。其主要的目的就是用来防范XSS攻击的。

CSP策略的核心思想十分简单，那就是web开发者在开发页面的时候告诉浏览器，当前页面可以加载执行哪些外部资源，开发者只需要简单的进行配置，它的全部实现和执行都由浏览器自动完成。

CSP策略大大的提高了网页的安全性，是防范XSS攻击的最佳实践。就算攻击者成功在当前页面注入了恶意脚本，只要配置了合适的CSP策略，浏览器在加载执行不在白名单里面的外部资源的时候也不会对恶意脚本进行执行。

除非攻击者还入侵了已经是当前站点白名单站点，否则页面被XSS攻击的概率很低。


### 如何启用CSP策略？
一般情况下开发者开启CSP策略主要有两种方式：
1. 通过HTTP的header：Content Secunty Policy
```js
Content Secunty Policy:default-src 'self' www.baidu.com;
```

2. 通过HTML中head中的meta标签
```html
<meta http-equiv="Content-Security-Policy" content="default-src 'self' www.baidu.com">
```

当一个页面同时配置了http头和meta标签的时候，浏览器会优先以HTTP请求的header字段设置的为主。
如果用户的浏览器已经为当前页面执行了一个CSP策略，那么则会跳过meta的定义。

### CSP策略怎么写？
1. 一个CSP头由多组CSP策略组成，多组CSP策略中间由分号隔开。
2. 每一组CSP策略由一个策略指令搭配一个信任内容来源列表组成。

#### CSP策略指令
+ script-src：定义了页面中Javascript的有效来源。
+ style-src：定义了页面中CSS样式的有效来源。
+ img-src：定义了页面中图片和图标的有效来源。
+ font-src：定义了字体加载的有效来源。
+ connect-src：定义了请求、XMLHttpRequest、WebSocket 和 EventSource 的连接来源。
+ child-src：定义了 web workers 以及嵌套的浏览上下文
+ default-src:定义了那些没有被更精确指令指定的安全策略。这些指令包括上面所有指令。

#### 内容源
和CSP策略指令一一对应的内容源一般分为三类：源列表、关键字和数据。
1. 源列表
源列表就是一个或多个由主机地址(协议域名端口号)组成的信任站点列表
```bash
// 代表当前网页中只对百度和淘宝两个站点的来源信任
Content-Security-Policy: default-src www.baidu.com www.taobao.com;
```

2. 关键字
关键字可以代表特殊的源列表，比如：
+ 'none' 没有信任的第三方站点
+ 'self'：和当前文档同源的站点资源可以信任
+ 'unsafe-inline'：对于内联资源如内联的script标签信任
```bash
// 代表当前网页中只对自己同源的资源进行信任
Content-Security-Policy: default-src 'self'; 
```

3. 数据
+ data: 允许加载data:URI资源
+ mediastream：允许加载 mediastream:URI资源



## Referer Policy
上面主要说了防范XSS攻击所使用的CSP策略，在前端攻击中针对CSRF攻击，HTTP规范还提供了一个很有用的Referer Policy用来防范CSRF攻击。

### Referer Policy是什么？
Referer是一个HTTP的头部字段，主要用来告诉服务端当前请求的来源地址。
一般情况下，对于多数的Ajax请求、图片等资源请求，请求在到达服务端之后，服务端都可以从referer字段中获取到当前来源的详细地址，对于一些不明的来自第三方域名的请求，就可以屏蔽从而防止CSRF攻击的发生。

并且，Referer字段比Origin字段更加准确，因为默认情况下Referer字段中标明的请求来源中包含了发起请求的路径，比如：
```bash
origin: https://mp.weixin.qq.com
referer: https://mp.weixin.qq.com/s?__biz=MzIxMjExNzQxMQ==&mid=2247484096&id
```

### Referer Policy如何开启？
一般情况下开启Referer Policy主要有3种途径：

1. 文档级：通过HTTP中的meta标签为整个文档设置
```html
<meta name="referrer" content="origin">
```

2. 请求级: 通过a标签、link标签、script标签、img标签上的referrerpolicy属性为单次请求设置
```html
<a href="http://example.com" referrerpolicy="origin">
```

3. 补充：也可以在 <a>、<area> 或者 <link> 元素上将 rel 属性设置为 noreferrer
```html
<a href="http://example.com" rel="noreferrer">
```

### Referer Policy如何配置？

HTTP规范规定，通过meta标签或者请求时设置的Referrer-Policy的值可以为以下之一：

1. no-referrer：在该文档的任何请求中Referer请求头字段都不包含来源地址
2. no-referrer-when-downgrade：这是Chrome85以前的默认值，代表从https站点发起http请求的时候，不包含来源地址
3. origin: 只会将文档的源发送，不会发送源的路径等详细信息
4. origin-when-cross-origin：对于跨域请求只会发送origin信息，不会发送完整的URL路径
5. same-origin：对于跨域请求不发送Referer请求字段
6. strict-origin-when-cross-origin：当前默认值。表示同源请求发送完整URL,同等安全级别的情况下发送Origin字段，当安全降级也就是从https到http的时候不发送referer字段。


### 补充
1. 大多数网站会采用默认配置,比如华为云的请求头部就会有Referer Policy：strict-origin-when-cross-origin。代表发起跨域请求的时候只会发送Origin源，不会通过Referer字段发送完整的URL请求路径。

2. Referer Policy可以配置一到多个策略，其主要用来为不支持该头部的浏览器提供了兼容方案。
```js
Referrer-Policy: no-referrer, strict-origin-when-cross-origin
```
以上表示在不支持最新的strict-origin-when-cross-origin策略的浏览器中，使用no-referer策略作为其Referer Policy