title: HTTP缓存
tags: [前端, 缓存, HTTP]
categories: 前端
toc: true
date: 2017-10-14 10:08:03
---

Web缓存分为很多种：数据库缓存，服务器缓存（代理服务器缓存，CDN缓存），浏览器缓存。

而浏览器缓存也分为很多种：HTTP缓存，cookie，localStorage，indexDB等。这篇文章主要聚焦在HTTP缓存的部分。

## 为什么需要缓存？

缓存可以带来很多好处，具体如下：

1. 减少冗余的数据传输，节省带宽。
2. 更快地加载页面，提升了用户体验。
3. 降低了对原始服务器的要求，可以更快地响应。

## 私有缓存和共有缓存

私有缓存即单个用户专享的缓存，比如Web浏览器中的缓存就属于私有缓存。

公有缓存为多个用户共享的缓存，比如共享代理服务器就属于公有缓存，这些服务器又被称为`代理缓存`。

## 浏览器缓存流程

当浏览器请求一项资源时，是否使用缓存的流程图如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/HTTP%E7%BC%93%E5%AD%98.png)

从上图可以看出HTTP缓存处理的整个流程，很多文章从该流程中提取出3个缓存策略，如下：

1. 存储策略：根据HTTP响应头的内容，决定是否缓存资源
2. 过期策略：确定缓存的资源是否过期（新鲜）
3. 协商策略：发送条件请求到原始服务器，判断缓存的过期资源是否可以重用

下面就分别从这3个角度来进行分析。

## 存储策略

服务器可以通过HTTP响应头来指定文档的缓存时间，按照优先级递减的顺序，服务器可以：

- 附加一个Cache-Control: no-store首部到响应中去
- 附加一个Cache-Control: no-cache首部到响应中去
- 附加一个Cache-Control: must-revalidate首部到响应中去
- 附加一个Cache-Control: max-age首部到响应中去
- 附加一个Expires日期首部到响应中去
- 不附加过期信息， 让缓存确定自己的过期日期

### no-Store

{% codeblock lang:javascript %}
Cache-Control: no-store
{% endcodeblock %}

以上响应头会禁止缓存对响应进行复制，缓存通常会像非缓存代理服务器一样，向客户端转发一条`no-store`的响应，然后删除对象。

### no-Cache

{% codeblock lang:javascript %}
Pragma: no-cache
Cache-Control: no-cache
{% endcodeblock %}

收到`no-cache`的响应后，并非不缓存资源，而是将资源缓存，但是在与原始服务器进行新鲜度再验证之前，缓存不能将其提供给客户端使用。

而`Pragma: no-cache`首部只是为了兼容HTTP/1.0+，除了向下兼容HTTP/1.0的程序外，其他情况，我们的应用程序应该使用`Cache-Control: no-cache`。

### max-age

{% codeblock lang:javascript %}
Cache-Control: max-age=3600
Cache-Control: s-maxage=3600
{% endcodeblock %}

其中`max-age`表明了改资源保持新鲜状态的秒数，在过了max-age指定的时间后，该资源过期。

其中`s-maxage`与max-age首部作用相同，不过其仅适用于共享（公有）缓存。

### Expires

{% codeblock lang:javascript %}
Expires: Fri, 05 Jul 2017, 02:00:00 GMT
{% endcodeblock %}

不推荐使用这个首部，它指定的是实际的过期日期而非秒数。由于众多的服务器以及客户端的时钟可能不同步，或者不正确，所以该字段没有什么太大意义。

### must-revalidate

{% codeblock lang:javascript %}
Cache-Control: must-revalidate
{% endcodeblock %}

缓存服务器可以被配置，使其对客户端提供一些过期的资源，以提高性能。如果原始服务器希望严格遵守过期信息，可以在响应头加上`must-revalidate`的字段。

它告诉缓存，在事先没有跟原始服务器进行再验证的情况下，不能对客户端提供这个资源的陈旧副本。

这意味着缓存必须对该项资源进行新鲜度检查，原始服务器不可用时，就返回一条`504 Gateway Timeout`错误。

### 试探性过期

如果响应中没有`Cache-Control: max-age`首部，也没有`Expires`首部，缓存可以计算出一个试探性最大使用期。

如果响应中包含了`Last-Modified`字段，那么就可以使用如下算法来得到过期时间：

{% codeblock lang:javascript %}
收到响应时间 + (Date - Last-Modified)*10%
{% endcodeblock %}

这种算法的理念如下：

- 如果已缓存资源最后一次修改距离当前时间很远，它可能是一份稳定的文档，不太会突然发生变化，因此将其继续保存在缓存中比较合适。
- 如果已缓存文件最近被修改过，说明它很可能频繁地发生变化，因此应该假设它很快会过期，并且尽早与原始服务器进行再验证。

## 过期策略

在缓存文档过期之前，缓存可以直接向客户端提供这些资源副本，而无需与服务器联系。但是一旦资源过期，缓存就必须与服务器进行核对，询问文档是否被修改过：

- 如果文档没有被修改，原始服务器会返回`304 Not Modified`,并且根据响应首部更新资源的缓存时间。
- 如果文档更新了，那就需要将更新资源发送给缓存，响应`200`

那么如何缓存如何判断某项资源是否过期呢？当然是根据`存储策略`中提到的`Expires`和`Cache-Control`字段了。因为Cache-Control用的是相对时间，而非绝对时间，所以我们更加倾向于使用较新的Cache-Control首部。

## 协商策略

缓存经过`过期策略`对资源进行判定后，如果资源没有过期，可直接将其提供给客户端。

但是如果资源过期了，就需要向原始服务器发起`再验证(条件请求)`, 以决定已过期副本是否可以直接提供给客户端使用。

HTTP定义了几个用于条件请求的首部，其中最有用的就是下面提到的`If-Modified-Since`和`If-None-Match`。

### Last-Modified & If-Modified-Since

当缓存从原始服务器请求来一项资源时，响应头中会带有`Last-Modified`字段，说明了该资源最后一次被修改的时间。

这个字段的值，会在发送条件请求的时候，通过`If-Modified-Since`传入到原始服务器。原始服务器将`If-Modifeid-Since`字段的值和资源的最新修改时间对比，如果相同，返回`403 Not Modified`，如果不同，返回修改后的资源。

使用`If-Modified-Since`来进行条件请求，有如下弊端：

1. 有些文档可能会周期性的重写（比如打包），这样实际内容没变，但是最后修改日期会发生变化。
2. 有些服务器提供的文档会在亚秒间隙发生变化，对这些服务器来说，以1s为力度的修改日期可能就不够用了。

而`ETag`可以解决这两个问题。

### ETag & If-None-Match

ETag即`Entity Tag`，即实体标签，它是原始服务器附加到文档上的任意标签。比如文档的序列号或版本名称，或者文档内容的指纹信息(散列值)等。

原始服务器在发送该资源时，通过`ETag`响应头将Etag传递给缓存。

Etag的值，会在发送条件请求的时候，通过`If-None-Match`请求头传入原始服务器，原始服务器则根据`If-None-Match`的值来决定返回304或者200。

### 以上两种验证方式应该在什么情况下使用？

- 如果原始服务器响应了ETag，HTTP/1.1客户端发送条件请求时就必须使用If-None-Match。
- 如果原始服务器只响应了Last-Modified，客户端就可以使用If-Modified-Since。
- 如果原始服务器同时响应了ETag和Last-Modified，客户端发送条件请求时应该同时传入If-None-Match和If-Modified-Since。
- 如果HTTP/1.1缓存或服务器收到的请求既带有If-Modified-Since，又含有If-None-Match，那么只有这两个条件都满足时，才能返回304 Not Modified。

以上就是关于HTTP缓存的基本总结，还有待进一步实践之。
