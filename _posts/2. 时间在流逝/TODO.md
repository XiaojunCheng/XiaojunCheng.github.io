---
layout: post
title: 时间在流逝之TODO清单
---

# TODO清单

* `leveldb`学习总结
* 消息队列学习总结
* `binlog`同步源码阅读
* 分库分表源码阅读
* `awk`学习
* `vim`学习
* RPC框架学习
	- Dubbo
	- Motan
* `okhttp`组件学习
* `Linux`性能监控
	- `CPU`: `top`, `dstat`
	- `IO`: `iostat`, `dstat`
	- `Network`: `tcpdump`, `dstat`
	- 内存映射: `pmap -x [pid]`
	- 查看系统调用: `dtruss`
* Guava学习
* 正则表达式
* Gitbook使用学习
* 缓存更新问题
* 考驾照
* 考雅思
* 持续学习
	- [ted教育](http://ed.ted.com/) 打开个人眼界
	- [深度博客](http://www.geekonomics10000.com/about)
	- [你知道么？读书这件事也是可以“遗传”的](http://mp.weixin.qq.com/s?__biz=MzAwMDgyMTA3Mg==&mid=2650055816&idx=1&sn=8e19d40084e102a8a9743541ffa21504#rd)
	- [为什么同样都是爱读书你却总是没进步？](http://mp.weixin.qq.com/s?__biz=MzAxNzI4MTMwMw==&mid=2651629760&idx=1&sn=aeb63cdc3cfb261389ac750f58086606&scene=0#wechat_redirect)
	- [李笑来，人人都能学好英语](http://zhibimo.com/read/xiaolai/everyone-can-use-english/index.html)
	- [How To Ask Questions The Smart Way](http://www.catb.org/esr/faqs/smart-questions.html)
	- [How to Ask Better Questions](https://hbr.org/2009/05/real-leaders-ask.html)
	- [十分钟后开始使用英语](http://xiaolai.li/2016/06/11/makecs-appendix01/)
	- [张是之：个人如何通过自学构建经济学的思维方式](http://mp.weixin.qq.com/s?__biz=MzAxNzI4MTMwMw==&mid=2651630017&idx=1&sn=f8cd62f10f8fa04417d25754731a7b19#rd)
	- [认知学习](http://www.yangzhiping.com/info/contact.html)
* [使用 Grafana＋collectd＋InfluxDB 打造现代监控系统](http://www.vpsee.com/2015/03/a-modern-monitoring-system-built-with-grafana-collected-influxdb/)
* 零碎知识学习
	- [Facebook 内部高效工作 PPT 指南](http://www.oschina.net/news/65549/facebook-inner-ppt)
	- [并发之痛 Thread，Goroutine，Actor](http://weibo.com/ttarticle/p/show?id=2309403948698710187414)
[旁观者-郑昀](http://www.cnblogs.com/zhengyun_ustc/)

### Java基础知识
>* CPU缓存，L1，L2，L3和伪共享
	- http://duartes.org/gustavo/blog/post/intel-cpu-caches/
	- http://mechanical-sympathy.blogspot.com/2011/07/false-sharing.html

##### 零碎代码

## 知识盲区
>* `bio` & `nio` & `aio`  
   1. Thrift [`TThreadedSelectorServer`](http://mizhichashao.com/dashu/thrift-book/08.08-TThreadedSelectorServer.md)
   2. [Netty系列之Netty线程模型](http://www.infoq.com/cn/articles/netty-threading-model/)
   3. [ Thrift Java Servers Compared]( https://github.com/m1ch1/mapkeeper/wiki/Thrift-Java-Servers-Compared)
   4. [IO不再神秘](http://2014.54chen.com/blog/2014/03/12/io-demystified/)
   5. [High-Performance Server Architecture](http://pl.atyp.us/content/tech/servers.html)
   6. [An NIO.2 primer, Part 1: The asynchronous channel APIs](http://www.ibm.com/developerworks/java/library/j-nio2-1/index.html)
   7. [An NIO.2 primer, Part 2: The file system APIs](http://www.ibm.com/developerworks/java/library/j-nio2-2/index.html)
   8. [Architecture of a Highly Scalable NIO-Based Server](https://today.java.net/pub/a/today/2007/02/13/architecture-of-highly-scalable-nio-server.html)
   9. [The C10K problem](http://www.kegel.com/c10k.html)
>* `mmap` & `zero copy` & `shared memory`  
   1. 薛笛[`java mmap分析`](http://site.douban.com/161134/widget/articles/8506170/article/18487141/) 

>* 数据结构
   1. `前缀树`、`最大堆`，`最小堆`、`bloom filter`  
   2. `LSM-Tree`（`Log Structured Merge`）
   3. `LIRS`  
      LIRS的基本思想是对访问的数据块进行分类，一部分为hot数据块，一部分为cold数据块。对于hot数据块我们可以分配90％以上的cache给它们。而对于cold数据块给它们分配10%  
      从LIRS算法的描述来看，可以理解为两个LRU队列的组合，利用cold缓冲区来保护Hot缓冲区，提高了进入hot缓冲区的门槛，阻止hot缓冲区频繁地变化  
>* MySQL使用优化
   1. [批量SQL插入性能优化](http://tech.uc.cn/?p=634)
>* 面试问题  
   1. 你们招聘这个岗位的原因是什么
   2. 公司的增长模式是什么
   3. 我需要/能够怎么做才可以直接为公司的增长和收益做贡献
   4. 公司将为我个人技术提升和在公司的重要性提供怎样的机会
   5. 这次面试我还有哪些需要提供的信息

## 知识分享
>* 个人博客搭建：[个人博客](www.selfmindspace.com)
   1. [蒋涛的github个人博客](http://hijiangtao.github.io/)
   2. [Zhe Li's Blog](http://senarukana.github.io/)
   3. 可以参考的高质量博客：[steve losh's blog](http://stevelosh.com/)
   4. 搭建个人博客指南：[beiyuu's blog](http://beiyuu.com/)
   5. [理想的写作环境：Git+Github+Markdown+Jekyll](http://www.yangzhiping.com/tech/writing-space.html)


# 参考资料

### 比较好的`博客`
>* 并发编程网：[http://ifeve.com/](http://ifeve.com/)
>* NoSQL相关：[http://codecapsule.com/](http://codecapsule.com/)
>* 可扩展系统设计：[https://highlyscalable.wordpress.com](https://highlyscalable.wordpress.com)
>* 高质量文章：[http://queue.acm.org/](http://queue.acm.org/)
>* db-readings：[Readings in Databases](https://github.com/rxin/db-readings)
>* 开源代码学习：[https://github.com/codefollower](https://github.com/codefollower)
>* pdf电子书籍下载：[http://www.downloadfreepdf.com/](http://www.downloadfreepdf.com/)
>* http://www.hollischuang.com/
>* http://mindwind.me/archive/
>* https://github.com/mindwind

### 比较好的`视频` & `文章`分享
>* 赵海平：[异步处理在分布式系统中的优化作用](http://www.infoq.com/cn/presentations/optimization-of-asynchronous-processing-in-distributed-systems?utm_source=infoq&utm_medium=videos_homepage&utm_campaign=videos_row3)


### 学习方面
>* `mmap` & `zero copy` & `shared memory` & `MappedByteBuffer` & `DirectByteBuffer`
	1. 文件最后时间只会在进程关闭或者unmap之后才会修改
	2. mmap文件不能超过2G
	3. shared memory 不会持久化
	4. mac下查看mmap使用过程中系统调用  
	   `sudo dtruss -p <pid> > trace.log` 相关系统调用参考`UNIX环境高级编程`
	4. 薛笛[`java mmap分析`](http://site.douban.com/161134/widget/articles/8506170/article/18487141/)  


>* 个人博客
   1. [蒋涛的github个人博客](http://hijiangtao.github.io/)
   2. [Zhe Li's Blog](http://senarukana.github.io/)
   3. 可以参考的高质量博客：[steve losh's blog](http://stevelosh.com/)
   4. 搭建个人博客指南：[beiyuu's blog](http://beiyuu.com/)
   5. [理想的写作环境：Git+Github+Markdown+Jekyll](http://www.yangzhiping.com/tech/writing-space.html)
   6. [jekyll官方文档](http://jekyllrb.com/)

>* [kafka镜像](https://cwiki.apache.org/confluence/display/KAFKA/Kafka+mirroring)
>* https://www.quora.com
>* http://blog.2baxb.me/about-me
>* [http://www.v2ex.com/](http://www.v2ex.com/)
>* [kafka](http://calvin1978.blogcn.com/articles/kafkaio.html)
>* [学习开源软件](https://github.com/zhuangbiaowei/learn-with-open-source)
