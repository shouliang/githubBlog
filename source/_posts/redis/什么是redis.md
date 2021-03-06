title: 一.什么是redis
date: 2015-05-07 14:45:23
tags: [Redis]
---

### 1.什么是redis
---
随着互联网的普及，用户数量的快速增长，产生的数据也越来越多，这也对我们的产品提出了新的考验，如何才能构建出高性能，而且扩展性高的应用程序呢？听说Redis是一个不错的选择，那么问题来了，什么是Redis呢？

Redis—— Remote Dictionary Server，它是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API，我们可使用它构建高性能，可扩展的Web应用程序。

Redis是目前最流行的键值对存储数据库，从2010年3月15日起，Redis的开发工作由VMware主持。从2013年5月开始，Redis的开发由Pivotal赞助。

如果你想了解Redis最新的资讯，可以访问 [官方网站]:http://redis.io/

### 2.什么时候使用redis
---
在实际生产环境中，很多公司都曾经使用过这样的架构，使用MySQL进行海量数据存储的，通过Memcached将热点数据加载到cache，加速访问，但随着业务数据量的不断增加，和访问量的持续增长，我们遇到了很多问题：
　　
* MySQL需要不断进行拆库拆表，Memcached也需不断跟着扩容，扩容和维护工作占据大量开发时间。
* Memcached与MySQL数据库数据一致性问题。
* Memcached数据命中率低或down机，大量访问直接穿透到DB，MySQL无法支撑。
* 跨机房cache同步问题。

以上问题都是非常的棘手，不过现在不用担心了，因为我们可以使用redis来完美解决，下面我们来了解下redis的特点，看看redis是如何解决以上问题的。

### 3.Redis特点
---
有那么多相同类型的数据库，为什么要选择redis？

相对于其他的同类型数据库而言，Redis支持更多的数据类型，除了和string外，还支持lists（列表）、sets（集合）和zsets（有序集合）几种数据类型。

这些数据类型都支持push/pop、add/remove及取交集、并集和差集及更丰富的操作，而且这些操作都是原子性的。Redis具备以下特点：

* 异常快速: Redis数据库完全在内存中，因此处理速度非常快，每秒能执行约11万集合，每秒约81000+条记录。
* 数据持久化： redis支持数据持久化，可以将内存中的数据存储到磁盘上，方便在宕机等突发情况下快速恢复。
* 支持丰富的数据类型: 相比许多其他的键值对存储数据库，Redis拥有一套较为丰富的数据类型。
* 数据一致性： 所有Redis操作是原子的，这保证了如果两个客户端同时访问的Redis服务器将获得更新后的值。
* 多功能实用工具： Redis是一个多实用的工具，可以在多个用例如缓存，消息，队列使用(Redis原生支持发布/订阅)，任何短暂的数据，应用程序，如 Web应用程序会话，网页命中计数等。

