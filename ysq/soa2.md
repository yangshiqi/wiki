## SOA落地浅谈（下）

时间过去有点久了，[上一篇](https://github.com/yangshiqi/wiki/blob/master/ysq/soa.md) 关于这个话题，似乎感兴趣的人不多么，没有收到任何反馈。。。

不过，想了想还是先把这个话题写完吧，算是对前一阵子思考的总结。请大家尽可能简单回顾一下 [上一篇](https://github.com/yangshiqi/wiki/blob/master/ysq/soa.md) 提到的一些原则和概念，这样能保持一个思维的连贯性。

#### 目录
1. 我们系统的现状
2. 不得不说的DTO
3. 理想中和现实中的SOA


#### 我们系统的现状

对于我们现在的系统来说，由service直接吐出实体开发起来很爽：web层可以直接访问对象内部数据和方法，可以级联加载，不断循环关联对象。但是，又必须注意到，对象的很多方法是在web层不能使用的，因为其不在一个service的业务事务中，修改其内部数据是不能被固化的，如果想修改，必须要写一个client方法放在实体方法中，然后转调回service处理（此处一不小心就可能会出现循环远程调用）。

对于service中的封装，我们要求大家要按照领域驱动设计（Domain-Driven Design）的原则，尽可能把业务逻辑放到相关实体中（以实体为核心建模），而服务层只是承载业务用例--即实体间的调用而不涉及相关实体内部逻辑；

这样，对于对象的使用，就有了比较矛盾的地方：一方面，实体自由穿梭在web层和service层；另一方面，实体上有这样大量的业务逻辑方法，web层是不能使用的。实体上的逻辑，既有业务层的，又有展现层的，“超级实体”的出现也就不足为奇了。

由于早期团队人少，开发工作都是一个人从前（web层）到后（service层）一股脑写过来，直接使用实体确实减少了中间数据传输环节，提高了开发效率。

但是，实体在各个web系统中游走，各种级联加载随意调用，这样做另外一个很大的弊端，就是牺牲了逻辑隔离性。因为大家对“封装业务逻辑”的认识不一致性和书写程序的随意性，大规模产生代码后，很难保障业务逻辑不会散落在web层（而其后DAL的出现又加剧了这个要素）。如果实体逻辑或者字段含义发生变化需要重构，就有可能需要从实体、service到web层全部修改，因为期间缺少了缓冲层。

此时，DTO被一些人一再提起了。




#### 不得不说的DTO

什么是DTO？还是引入wikipedia: 

> Data transfer object (DTO) is an object that carries data between processes. The motivation for its use has to do with the fact that communication between processes is usually done resorting to remote interfaces (e.g. web services), where each call is an expensive operation. Because the majority of the cost of each call is related to the round-trip time between the client and the server, one way of reducing the number of calls is to use an object (the DTO) that aggregates the data that would have been transferred by the several calls, but that is served by one call only.

很清晰，百度百科里边的内容就别看了，会误解。其中也提到了区别：

> The difference between data transfer objects and business objects or data access objects is that a DTO does not have any behavior except for storage and retrieval of its own data (accessors and mutators). DTOs are simple objects that should not contain any business logic that would require testing.This pattern is often incorrectly used outside of remote interfaces. This has triggered a response from its author where he reiterates that the whole purpose of DTOs is to shift data in expensive remote calls.

慢慢看，都能看懂，[链接](http://en.wikipedia.org/wiki/Data_transfer_object)

概括一下：

1. DTO是为了跨进程传输而发明的；
2. DTO是为了减少远程调用的巨大开销而封装的对象；
3. DTO是只是传输对象，不要承载额外的业务逻辑；

DTO是一个典型的用空间换时间的概念：因为需要结合业务用例，考虑传输数据的通用性，而降低消费层（如：web，app等）和服务层的调用次数，减少服务器建联的时间消耗（即定义中提到的round-trip）。我们可以提供在前台和后台之间引入DTO而降低两者之间的耦合性，搭建中间层来起到缓冲、隔离的作用。

同时，DTO的另外一个好处就是开发间的“按约定开发”。消费层的开发者根据业务实际需要去设计需要的DTOs，服务层的开发者再根据这个约定，去实现并填充这些DTOs。比如对于web开发来说，前端开发者甚至可以拿到service返回的json对象直接做数据输出。--这只是理论上的优点，随着现代前端交互的复杂性，nodejs的诞生也是为了给前端开发者一个同语言(javascript)的缓冲层，以隔离前端变化。当然，一个好的接口设计也需要对业务逻辑和开发的深入了解，需要一定的预判性和平衡性，这是个哲学话题，在此不表。但是，不可否认，在开发跨系统调用的需求时，设计接口是非常重要的一个环节，希望看到这里的同学们记住这一点，以后在实际工作中慢慢体会。

#### 理想中和现实中的SOA

由于我也没有实际设计过比当前我们的系统更复杂的系统了，所以很多概念都是参考别的公司公布的论文或者架构介绍的文章或者别人的分享体会的，所以在此，我把接下来要说的形容成：我理想中的SOA。同时，也不考虑实际情况中小团队人员问题。实践证明，小团队不适合直接搞SOA，反而容易得不偿失，待系统做大，人员变多，再转身来不迟。

我理想中的SOA应该是：
1. 契约先行的；
2. 跨语言的；
3. 隔离变化的；
4. 接口及调用相对稳定的；
5. 无状态的；
6. 高效的；
7. 可能还有其它我没想到的。。。

即调用栈类似于：
1. 消费层：函数型/面向对象型等前端逻辑->服务封装层->SOA通讯层；
2. 服务层：SOA通讯层->facade层->service层->实体层->数据层；

这里解释一下消费层服务封装层和服务层的facade，这两个是目前在我们系统中没有的：

前端服务封装层：前端是否可以组织自己的service，来完全隔离对于服务层的依赖？？？这个是写到此处的时候想到的，可以留作开放性讨论问题，以后大家讨论一下。有点类似于我们喜欢的集成层，只不过是放在了消费层，视角不同罢了。

facade：在实际开发过程中，会遇到一个业务用例，需要服务层多个service组合完成的情况。目前，我们直接在一个service代码中，new另外一个service，然后调用相关代码。而此时，引入facade层，就可以解决这个尴尬的情况。


SOA通讯层，则可以通过类似于thrift等代码生成工具，自动在服务层和消费层直接生成调用代码进行调用；或者自己实现方法，比如采用http传输json等。这层应该是很薄，很通用的一层，可以完全忽视任何业务逻辑。

如下简单示意图，帮助说明我的意思：

![CDN](https://github.com/yangshiqi/wiki/blob/master/imgs/soa.png)


当然，现实中的SOA需要考虑的因素将会更多。比如，负责消费端（展示端）的程序员往往不关注业务逻辑，而只是关注页面逻辑，他们对业务逻辑的深入思考度平均来说不如服务端的程序员，所以，设计出来的接口往往需要多次修改且不容易稳定。DTO设计的随意性，导致业务稍微变化，就要从下到上一通修改，框架中的缓冲层起不到缓冲的作用，就变成累赘，让人觉得增加工作量，没用。

服务层的分离也是不容易划清楚边界的地方，业务复杂的系统，往往对象错综交错，越到展现层，就越是复杂。从这种网状结构逐步划分系统边界，梳理流程就是一个艰难的过程。

#### 总结

从上一篇宏观介绍了SOA的定义和设计原则，以及本篇介绍的神器DTO，至此，可能大家看的也是一头雾水，到底我们要怎么做？怎么做才是能够解决当前我们系统问题的王道？

这个问题我在思考，希望通过这两篇文章，能够让不同程度的你获得一些感觉，从而在接下来我们的讨论和实施环节中，大家能够在概念上尽可能统一一些。拆分和梳理势在必行。对于我们系统的改造，更多的还是需要我们通盘去思考，我们的系统，从职责，从宏观角度来看，到底怎么做才是更适合我们当前和可预见未来的？我们的抽象是否是对的？系统边界呢？

reload--我十分喜欢这个词，它能够帮助我们刷新自己，用过去的经验填满弹药，再一次全力开火。
