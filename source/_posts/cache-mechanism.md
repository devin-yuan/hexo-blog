---
title: cache-mechanism
date: 2017-11-02 19:26:33
tags:
---

## 缓存的分类
缓存分为服务端侧（比如：Nginx、Apache）和客户端侧（比如：web brower）。

- 服务端缓存又分为代理服务器缓存和反向代理服务端缓存（也叫网关缓存，比如Nginx反向代理、Squid等），其实广泛使用的CDN也是一种服务端缓存，目的都是让用户的请求走捷径，并且都是缓存图片、文件等静态资源。
- 客户端缓存一般指的是浏览器缓存，目的就是加速各种静态资源的访问，想想现在的大型网站，随便一个页面都是一两百个请求，每天pv都是亿级别，如果没有缓存，用户体验会急剧下降，同时服务器压力和网络带宽都面临严重的考验。

## 浏览器缓存机制详解
浏览器缓存控制机制有两种：HTML Meta标签 和 HTTP头信息。

### HTML Meta标签控制缓存
浏览器缓存机制，其实就是HTTP协议定制的缓存机制（如：Expires; Cache-control等）。但是也有非HTTP协议定义的缓存机制，如使用HTML Meta标签，Web开发者可以在HTML页面的<head>节点中加入<meta>标签，代码如下：

```
<meta http-equiv="Pragma" content="no-cache">
```

上述代码的作用是告诉浏览器当前页面不被缓存，每次访问都需要去服务器拉取。使用上很简单，但只有部分浏览器可以支持，而且所有缓存代理服务器都不支持，因为代理不解析HTML内容本身。而广泛应用的还是HTTP头信息来控制缓存，下面主要介绍HTTP协议定义的缓存机制。

### HTTP头信息控制缓存
#### 浏览器请求流程
- 浏览器第一次请求流程图：
![image](http://static.oschina.net/uploads/space/2015/0119/015343_psx2_568818.png?_=4807408)

- 浏览器再次请求时：
![image](http://static.oschina.net/uploads/space/2015/0119/015353_P04w_568818.png?_=4807408)

#### 几个重要概念解释
- Expires策略：Expires是Web服务器响应消息头字段，在响应http请求时告诉浏览器在过期时间前浏览器可以直接从浏览器缓存取数据，而无需再次请求。不过Expires是HTTP 1.0的东西，现在浏览器均默认使用HTTP 1.1，所以它的作用基本忽略。Expires的一个缺点就是，返回的到期是服务端的时间，这样存在一个问题，如果客户端的时间与服务器的时间相差很大，那么误差就很大，所以在HTTP 1.1版本开始，使用“Cache-Control: max-age=秒”替代。
- Cache-Control策略：Cache-Control与Expires的作用一致，都是指明当前资源的有效期，控制浏览器是否直接从浏览器缓存取数据还是重新发请求到服务器取数据。只不过Cache-Control的选择更多，设置更细致，如果同时设置的话，其优先级高于Expires。设置列表如下：

属性 | 解释
---|---
Public | 指示响应可以被任何缓存区缓存。
Private | 指示对于单个用户的的整个或部分响应消息，不能被共享缓存处理。这允许服务器仅仅描述当前用户的部分响应消息，此响应消息对于其他用户的请求无效。
no-cache | 指示请求或响应消息不能缓存。
no-store | 用于防止重要的信息被无意的发布。在请求消息中发送将使得请求和响应消息都不适用缓存，完全不存下来。
max-age | 指示客户端可以接收生存期不大于指定时间（以秒为单位）的响应。
min-fresh | 指示客户端可以接收响应时间小于当前时间加上指定时间的响应。
max-stale | 指示客户端可以接收超出超时时间的响应消息。

- Last-Modified/If-Modified-Since: Last-Modified/If-Modified-Since要配合Cache-Control使用。


属性 | 解释
---|---
Last-Modified | 标识这个响应资源的最后修改时间。web服务器在响应请求时，告诉浏览器资源的最后修改时间。
If-Modified-Since | 当资源过期时（使用Cache-Control标识的max-age），发现资源具有Last-Modified声明，则再次向web服务器请求时带上头 If-Modified-Since，表示请求时间。web服务器收到请求后发现有头If-Modified-Since 则与被请求资源的最后修改时间进行比对。若最后修改时间较新，说明资源又被改动过，则响应整片资源内容（写在响应消息包体内），HTTP 200；若最后修改时间较旧，说明资源无新修改，则响应HTTP 304 (无需包体，节省浏览)，告知浏览器继续使用所保存的cache。

- Etag/If-None-Match：Etag/If-None-Match也要配合Cache-Control使用。


属性 | 解释
---|---
Etag | web服务器响应请求时，告诉浏览器当前资源在服务器的唯一标识（生成规则由服务器决定）。Apache中，ETag的值，默认是对文件的索引节（INode），大小（Size）和最后修改时间（MTime）进行Hash后得到的。
If-None-Match | 当资源过期时（使用Cache-Control标识的max-age），发现资源具有Etage声明，则再次向web服务器请求时带上头If-None-Match （Etag的值）。web服务器收到请求后发现有头If-None-Match 则与被请求资源的相应校验串进行比对，决定返回200或304。

- 既生Last-Modified何生Etag？你可能会觉得使用Last-Modified已经足以让浏览器知道本地的缓存副本是否足够新，为什么还需要Etag（实体标识）呢？HTTP1.1中Etag的出现主要是为了解决几个Last-Modified比较难解决的问题：**Last-Modified标注的最后修改只能精确到秒级，如果某些文件在1秒钟以内，被修改多次的话，它将不能准确标注文件的修改时间
如果某些文件会被定期生成，当有时内容并没有任何变化，但Last-Modified却改变了，导致文件没法使用缓存
有可能存在服务器没有准确获取文件修改时间，或者与代理服务器时间不一致等情形**
- Etag是服务器自动生成或者由开发者生成的对应资源在服务器端的唯一标识符，能够更加准确的控制缓存。Last-Modified与Etag一起使用时，服务器会优先验证Etag。

总结以下几种状态码的区别：

名称 | 解释
---|---
200状态 | 当浏览器本地有没缓存或者下一层失效时，或者用户强制刷新时，浏览器直接去服务器下载最新数据。
304状态 | 这一层由Last-Modified/Etag控制。当下一层失效时或者用户刷新时，浏览器就会发送请求给服务器，如果服务端没有变化，则返回304给浏览器。
200状态（from cache） | 这一层由expires/cache-control控制。expires（http 1.0版有效）是绝对时间。cache-control（http 1.1版有效）是相对时间。两者都存在时，cache-control覆盖expores，只要没有失效，浏览器只访问自己的缓存。

## 用户行为与缓存
浏览器缓存行为与用户的行为有关，总结如下：

用户操作 | Expries/Cache-control | Last-Modified/Etag
---|---|---
地址栏回车 | 有效 | 有效
页面链接跳转 | 有效 | 有效
新开窗口 | 有效 | 有效
前进、后退 | 有效 | 有效
F5刷新 | 无效（BR重置max-age=0） | 有效
Crtl+F5刷新 | 无效（重置CC=no-cache） | 无效（请求头丢弃该选项）

## 相关参考
1. 浏览器缓存机制：http://www.cnblogs.com/skynet/archive/2012/11/28/2792503.html
2. Web开发人员需知的Web缓存知识：http://www.oschina.net/news/41397/web-cache-knowledge
3. 浏览器缓存详解:expires,cache-control,last-modified,etag详细说明：http://blog.csdn.net/eroswang/article/details/8302191
4. 在浏览器地址栏按回车、F5、Ctrl+F5刷新网页的区别：http://blog.csdn.net/yui/article/details/6584401
5.  Cache Control与ETag：https://blog.othree.net/log/2012/12/22/cache-control-and-etag/
6.  缓存的故事：http://segmentfault.com/blog/animabear/1190000000375344
7.  Google的PageSpeed网站优化理论中提到使用Etag可以减少服务器负担：https://developers.google.com/speed/docs/pss/AddEtags
8.  yahoo的Yslow法则中则提示谨慎设置Etag：http://developer.yahoo.com/performance/rules.html#etags
