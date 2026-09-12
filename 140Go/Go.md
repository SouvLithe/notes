# File 1
转go主要是写后端，

写后端的基础知识储备有`http 协议，mysql ，redis，rpc`

该方案是一个未完成的方案，需要大家互相提供资源。

第一步先学这个， 看3遍到5遍，全部掌握

[https://golang.google.cn/tour/welcome/1](https://golang.google.cn/tour/welcome/1)

 
[Go 语言设计与实现](https://draveness.me/golang/#go-%e8%af%ad%e8%a8%80%e8%ae%be%e8%ae%a1%e4%b8%8e%e5%ae%9e%e7%8e%b0)

[https://draveness.me/golang/](https://draveness.me/golang/)

这个是go的八股文，面试主要问这个

**深入****Go****底层原理，重写****Redis****中间件实战**

[https://pan.baidu.com/s/1TL_16GzEShW2zsL5nrys7w](https://pan.baidu.com/s/1TL_16GzEShW2zsL5nrys7w)  提取码：skii

[https://geektutu.com/](https://geektutu.com/)

这个可以研究研究

链接: ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABQAAAAPCAMAAADTRh9nAAAAilBMVEUAAAAArUEArUEArUEArUEArUEArUEArUEArUEArUEArUEArUEIsEYArUEArUEArUEArUEArUEArUEArUEet1cArUEArUEArUEArUEArUH///8Fr0Sq5MA6wGz4/frq+PAsvGISs04juVtfzIi16MjZ8+N305oLsUlpz5BOxnvg9eh71Z296s656Mts52KcAAAAGXRSTlMAarTu0Azk9xOXq2TYwBADpVXvgtxBhSB52zgb+gAAAI5JREFUeF5lyNcSgyAUBcCrooAlvR2w97T//70kA5aZ7OPSj3BC5nGP7SJBlpAcEy5tB1gp4/9sipPJZLl7nrkm5XxZ/sTF5B5WXbTA2aTPy74fgap7VAi2ZEidKtXoV54BR7Ku/LtFqkbgIGhyg26VSgE4NNsm0O9uAFxa2TAMNcB8WvNdAKG9hRNHc3wA3z4P8zYBe+AAAAAASUVORK5CYII=) 

[https://pan.baidu.com/s/15449lWT80wQESEdZRMPTag?pwd=qpqm](https://pan.baidu.com/s/15449lWT80wQESEdZRMPTag?pwd=qpqm)  提取码: qpqm 这个是群友提供的极客时间的培训课程，可以参考

目前主要推荐这三个资源，多了也看不完。

我个人使用go的一点体会，这玩意语法是相当的丑陋，编译器的后端优化，垃圾回收算法这块其实是一坨狗屎。 但是好处是高并发不需要你个人去操控，换言之高并发比较简单，对比java 来说，首先是轻，而且编译型语言速度也有保证。 学go的话，建议先把c掌握，如果你完成了cs61abc 的话，掌握go的语法也就3天到4天。

面试这块，我看了看互联网上的面试，主要是问http库的实现模式，还有go的并发库的设计，主要是select、 `goruntime` 、以及channel这块。特别厉害的面试官会问一些垃圾回收算法（比较少见）。

如果是想面试月收入20k以上的岗位，你需要掌握的知识还有比如grpc的实现原理。Gin的实现原理，radix树等一系列。

面试这块，我个人的意见是你要有一个自己的项目，同时强调自己对源码的理解，能清楚的说清楚一个框架的运行原理，或者一个库的实现原理。最好是自己掌握主动权，不要让面试官带着你走。祝大家找到好工作。

该文档是1.1 版本，完成时间是是2022 /11/29

# 背景
首先，了解Golang有哪些知识点，对整理路线有个把握 顺序：右上角开始顺时针45度走一圈 ![](https://cdn.nlark.com/yuque/0/2023/png/32825554/1685844531270-d6f50842-77b7-4699-b844-5d49aa142a54.png#averageHue=%23f4f4f7&clientId=ubf411074-bc8d-4&from=paste&id=u81d11ec1&originHeight=1414&originWidth=1986&originalType=url&ratio=1.375&rotation=0&showTitle=false&status=done&style=none&taskId=u34124b8d-c576-4874-92eb-7faea918ce4&title=) 相关重点知识牛哥都精选了几个还不错的资料，牛哥严格把控质量与数量，少即是多，选择真正有效，说到点子上的资料。

Golang高效、学习成本低，面试点可控，通常来说，跟随这个指引，可以在2-3周内可以将Golang掌握到不错的程度。

## 学习指引

大家看的过程中，遇到问题及时反馈，如果有更好的资料也及时同步，持续共建。

|模块|内容|资料|说明|
|---|---|---|---|
|基础语法||||

| 基础语法

|

- [Golang入门(2):一天学完GO的基本语法 - 掘金](https://juejin.cn/post/6844904117450571790)
    
- [Go语言快速入门](https://mp.weixin.qq.com/s?__biz=Mzg5ODU2ODczMQ==&mid=2247484132&idx=1&sn=ea8e3fd37b9d351e6aca8054f5e87c40&chksm=c061c590f7164c86628664f1009b088a3f4ea703765b49ee3939856870dc8b6f1db661d54174&token=1386439580&lang=zh_CN#rd)（牛牛自产，用例跑完掌握基本语法） | 选得这两个，都是上手极快的教程。 快速搭建好环境、跑完用例 | | 数据结构
    

| 数据结构

|

- [[Golang]-1 Slice与数组的区别](https://www.cnblogs.com/feily/p/14134062.html)（比较简洁，且有总结，入门可看）
    
- [【Go】深入剖析slice和array](https://zhuanlan.zhihu.com/p/54780689)（详细、有实验，缺点是不简洁，耐心好的可看）
    
- [Go Map底层实现原理](https://zhuanlan.zhihu.com/p/495998623)（Map，很容易被考察到）
    
- [Golang实现栈和队列](https://hunterhug.github.io/goa.c/#/algorithm/stack_queues)（其它数据结构的实现有兴趣也可以看看）
    
- [为什么 Go map 和 slice 是非线程安全的？](https://segmentfault.com/a/1190000040716956) | 数据结构必须掌握，是基础也是面试重点 Map底层实现 Slice和Array的区别 实现栈和队列 哪些结构是线程安全的？ | | 并发编程
    

| 协程

|

- [协程使用](https://www.cnblogs.com/sparkdev/p/10930168.html)（会用即可）
    
- [干货 | 进程、线程、协程 10 张图讲明白了！](https://zhuanlan.zhihu.com/p/337978321)（理解协程的意义） | 初步了解协程 深刻理解为什么要有协程 | | | context
    

| [Go Concurrency Patterns: Context](https://go.dev/blog/context) (官方文档） [深度解密Go语言之context](https://zhuanlan.zhihu.com/p/68792989) [【Go】Context 源码解析](https://juejin.cn/post/7081870299653734436)（看点源码分析） | 理解Context的使用场景； 熟悉Context的接口； **实践：使用Context传递traceId** | | | channel

|

- [Go Channel 详解](https://www.runoob.com/w3cnote/go-channel-intro.html)
    
- 《Go语言学习笔记》第七章、第二部第4节 | 有缓冲无缓冲，分别解决什么问题？ Channel底层原理。 | | | sync
    

|

- [同步原语与锁](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-sync-primitives/)
    
- [Golang sync.WaitGroup的用法](https://studygolang.com/articles/12972)(WaitGroup很常用） | **实践：并发100个协程请求，阻塞等待所有协程完成，** **并拿到所有子协程处理结果。** | | web编程
    

| http

|

- [golang net/http包使用](https://www.kancloud.cn/digest/batu-go/153529)
    
- [Go 的 http 包](https://learnku.com/docs/build-web-application-with-golang/034-gos-http-package-detailed-solution/3171)（了解http大概做了啥事） | 用http起服务 **实践：给http服务提供中间件功能** | | | seelog
    

|

- [Golang seelog 使用入门简介](https://blog.csdn.net/u010649766/article/details/79261173)
    
- [Go中seelog日志包](https://www.jianshu.com/p/033754491393) | 初期会使用即可； 后期理解原理，考虑性能，和其它框架对比； | | | | | | | gin
    

| [golang框架-web框架之gin](https://juejin.cn/post/6844903938093744142) [gin框架httprouter路由实现原理](https://juejin.cn/post/7121614553649004575) [性能比较](https://colobu.com/2016/03/23/Go-HTTP-request-router-and-web-framework-benchmark/) | 使用gin快速搭建web服务 理解gin路由、中间件原理 **实践：使用gin中间件实现打印输出包的功能** | | | gorm

| [GORM 指南](https://gorm.io/zh_CN/docs/index.html)(gorm官网，快速启动，时常翻阅） [golang orm对比](https://segmentfault.com/a/1190000015606291)（能说出1，2个点即可） | 和xorm进行对比 **实践：和MySQL交互，实现简单的crud** | | | redigo |

- [使用redigo](https://cloud.tencent.com/developer/article/1381778)（抓大放小，一目了然） | **实践：和Redis交互，实现基本结构的操作** | | 原理深入
    

| GMP

|

- [Golang 调度器 GMP 原理与调度全分析](https://learnku.com/articles/41728)
    
- 《Go语言学习笔记》第七章、第二部第3节 | 掌握比较宏观的模型概念； 理解一些细节场景； | | | 内存结构 |
    
- 《Go语言学习笔记》第二部第1节 | 有个大致的理解，属于强化项 | | | 垃圾回收 |
    
- 《Go语言学习笔记》第二部第2节 | 有个大致的理解，属于强化项 | | 微服务框架 | go-zero
    

|

- [开源go微服务框架对比](https://zhuanlan.zhihu.com/p/488233067)
    
- [go-zero.dev](https://go-zero.dev/cn/)（文档很清晰了）
    

| 大概了解是啥 学会使用 熟练后研究原理 | | | kratos |

- [简介 | Kratos](https://go-kratos.dev/docs/) （文档很清晰了）
    

| 大概了解是啥 学会使用 熟练后研究原理 |