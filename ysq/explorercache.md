### 系统缓存

* 浏览器缓存；
* varnish缓存；
* 片段缓存；
* 实体缓存；
* sql缓存；

#### 浏览器缓存

Cache-Control是指明当前资源的有效期，以控制是直接从浏览器缓存取数据还是重新发请求到服务器取数据。


| Cache-directive | 说明 |
| ------------ | ------------- |
| public	| 所有内容都将被缓存 |
| private	| 内容只缓存到私有缓存中 |
| no-cache |	所有内容都不会被缓存 |
| no-store	| 所有内容都不会被缓存到缓存或 Internet 临时文件中 |
| must-revalidation/proxy-revalidation |	如果缓存的内容失效，请求必须发送到服务器/代理以进行重新验证 |
| max-age=xxx (xxx is numeric)	| 缓存的内容将在 xxx 秒后失效, 这个选项只在HTTP 1.1可用, 并如果和| Last-Modified一起使用时, 优先级较高 |

以上概念中，比较重要的是：
* max-age：浏览器可以接收生存期不大于指定时间（以秒为单位）的响应；
* no-cache：浏览器不缓存；


如图：

![cvcs](https://github.com/yangshiqi/wiki/blob/master/imgs/explorer/example.png)




对 cache-directive 值的浏览器响应


|Cache-directive	| 打开一个新的浏览器窗口 | 在原窗口中单击 Enter 按钮 | 刷新 | 单击 Back 按钮|
|----------- | ------------------- | --------------------------|------|------|
|public| 浏览器呈现来自缓存的页面 | 浏览器呈现来自缓存的页面|浏览器重新发送请求到服务器|浏览器呈现来自缓存的页面
|private	| 浏览器重新发送请求到服务器	|第一次，浏览器重新发送请求到服务器；此后，浏览器呈现来自缓存的页面|浏览器重新发送请求到服务器|浏览器呈现来自缓存的页面|
|no-cache/no-store|浏览器重新发送请求到服务器|浏览器重新发送请求到服务器|浏览器重新发送请求到服务器|浏览器重新发送请求到服务器|
|must-revalidation/proxy-revalidation|浏览器重新发送请求到服务器|第一次，浏览器重新发送请求到服务器；此后，浏览器呈现来自缓存的页面|浏览器重新发送请求到服务器|浏览器呈现来自缓存的页面|
|max-age=xxx (xxx is numeric)|在 xxx 秒后，浏览器重新发送请求到服务器|在 xxx 秒后，浏览器重新发送请求到服务器|浏览器重新发送请求到服务器|在 xxx 秒后，浏览器重新发送请求到服务器|


具体参考:[百度百科](http://baike.baidu.com/link?url=4AfVDmdICeLsTN7blG8E-Wa-NUVfx-rg1SWmBrwfAKYujpROFXabJQnklIaPuVjqXeSo9sMKdh7UDf41jNcSzK)

这里还要提下Expires。我一直也对Expires感觉有点晕，感觉和Cache-Control提供类似的功能：Expires是Web服务器响应消息头字段，在响应http请求时告诉浏览器在过期时间前浏览器可以直接从浏览器缓存取数据，而无需再次请求。后查询得知：不过Expires 是HTTP 1.0的东西，现在默认浏览器均默认使用HTTP 1.1，所以它的作用基本忽略。

Cache-Control是基于HTTP1.1的，可以看成是Expires的改进版，提供的功能控制更细致，两者同时出现时，其优先级高于Expires。







#### Last-Modified/If-Modified-Since

Last-Modified/If-Modified-Since要配合Cache-Control使用。

* Last-Modified：web服务器在响应浏览器请求时，告诉浏览器该资源的最后修改时间；
* If-Modified-Since：当资源过期时（使用Cache-Control标识的max-age），发现资源具有Last-Modified声明，则再次向web服务器请求时带上头 If-Modified-Since，表示请求时间。web服务器收到请求后发现有头If-Modified-Since 则与被请求资源的最后修改时间进行比对。若最后修改时间较新，说明资源又被改动过，则响应整片资源内容（写在响应消息包体内），HTTP 200；若最后修改时间较旧，说明资源无新修改，则响应HTTP 304 (不会下载资源内容，节省时间和带宽)，告知浏览器继续使用所保存的cache。

#### Etag/If-None-Match

Etag/If-None-Match也要配合Cache-Control使用。

* Etag：web服务器响应请求时，告诉浏览器当前资源在服务器的唯一标识，此生成规则由服务器来决定；
* If-None-Match：当资源过期时（使用Cache-Control标识的max-age），发现资源具有Etage声明，则再次向web服务器请求时带上头If-None-Match （Etag的值）。web服务器收到请求后发现有头If-None-Match 则与被请求资源的相应校验串进行比对，决定返回200或304。

那么，问题又来了，Last-Modified和Etag看起来又很像，这两者难道也是版本的关系？

确实是。

* 在HTTP 1.1中引入的Etag是为了改进以时间为单位的Last-Modified的不足而生，因为Etag可以控制到秒以内的级别；
* 在经过代理服务器的时候，Last-Modified有可能会被替换，导致和服务器时间不同，缓存策略失败；

Etag是服务器自动生成资源在服务器端的唯一标识符，能够更加准确的控制缓存。同理，Last-Modified与ETag是可以一起使用的，服务器会优先验证ETag，一致的情况下，才会继续比对Last-Modified，最后才决定是否返回304。



![action](https://github.com/yangshiqi/wiki/blob/master/imgs/explorer/explorer-action.png)


#### 总结

first：


![first](https://github.com/yangshiqi/wiki/blob/master/imgs/explorer/explorer-nocache.png)

second:

![second](https://github.com/yangshiqi/wiki/blob/master/imgs/explorer/explorer-cached.png)

参考：[博客](http://www.cnblogs.com/skynet/archive/2012/11/28/2792503.html) [百科](http://baike.baidu.com/link?url=4AfVDmdICeLsTN7blG8E-Wa-NUVfx-rg1SWmBrwfAKYujpROFXabJQnklIaPuVjqXeSo9sMKdh7UDf41jNcSzK)