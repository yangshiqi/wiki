### 缓存知识科普 -- varnish缓存

根据一般应用级缓存，可以大致分成以下几类：

* 浏览器缓存；
* varnish缓存；
* 片段缓存；
* 实体缓存；
* sql缓存；

本篇介绍第二节：varnish缓存。前一篇请参考：[《浏览器缓存》](https://github.com/yangshiqi/wiki/blob/master/ysq/explorercache.md)。

介绍varnish前先介绍一下反向代理（Reverse Proxy）的概念：

> 在计算机网络中，反向代理是代理服务器的一种。它根据客户端的请求，从后端的服务器上获取资源，然后再将这些资源返回给客户端。与前向代理不同，前向代理作为一个媒介将互联网上获取的资源返回给相关联的客户端，而反向代理是在服务器端作为代理使用，而不是客户端。-- 引自[wikipedia](http://zh.wikipedia.org/wiki/%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86)

简单来说如图：

![reverse proxy](http://upload.wikimedia.org/wikipedia/commons/thumb/6/67/Reverse_proxy_h2g2bob.svg/280px-Reverse_proxy_h2g2bob.svg.png)

用户从外部访问到的是这台代理服务器，然后由它来转发请求到后端的服务器，从而起到负载均衡的作用。

结合我们的实际，示意图如下：

![ours](https://github.com/yangshiqi/wiki/blob/master/imgs/varnish/varnish-1.png)

可以看到，根据我们配置的规则，节选：

![rules](https://github.com/yangshiqi/wiki/blob/master/imgs/varnish/varnish-2.png)

在命中这些规则后，会优先去varnish中查找是否有内容且没有过期（上图中的set beresp.ttl = 60 m 可以理解为缓存60分钟），如果有内容，则直接返回数据，就不需要再访问后边的服务器集群获取数据了，这也就是我们常说的“页面缓存”概念。我们知道，用户请求一个[页面](http://www.haodf.com/hospital/DE4rIxMvCogR-EoxrURJ3U3.htm)，是需要webapp层->service层->db层等调用顺序的，且service还会多次调用，还有可能涉及到搜索引擎等等各种操作，开销是非常大的。如果配置的页面缓存，则直接命中返回了，后端没有任何开销。

varnish内部的简单原理示意如下：

![simple](https://github.com/yangshiqi/wiki/blob/master/imgs/varnish/simple.jpg)

简单解释：在收到请求(recevie)，检查当前配置的规则，如果请求在规则中，则检查缓存区(Lookup)，命中(Hit)的话就直接返回了(Deliver)。如果没有命中(Miss)，则根据服务器集群的配置，转发到后端服务器集群处理(Fetch)，然后保存结果，返回请求(Deliver)。如果当前的请求没有配置缓存策略(Pass)，则直接Fetch后端，然后直接Deliver结果。

Pipe和Pass有类似，在这可以先忽略，具体的差异可以参考：[pass or pipe](http://yeelone.blog.51cto.com/1476571/772369)。

看到这，相信大多数都可以理解varnish的一般工作原理和我们所谓的“页面缓存”的概念了。当然，varnish在目前的开发和测试环境中是没有部署的，这点是需要特别指出来打架要注意的，被缓存的页面是当做html处理、并且直接返回的。这代表着这些被缓存的页面不是每次请求来是，都会执行后台的php逻辑的，在这些页面上如果要实现动态功能（比如：判断登录、收藏、访问量等），就需要单独实现ajax异步请求了。


进一步的：

![simple](https://github.com/yangshiqi/wiki/blob/master/imgs/varnish/big.jpg)