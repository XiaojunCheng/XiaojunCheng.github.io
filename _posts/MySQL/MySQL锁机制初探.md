# MySQL锁机制初探

注：本文都是基于InnoDB存储引擎

## 一、预备知识点

- MVCC：基于多版本的并发控制协议
	- [Multi-Version Concurrency Control](https://en.wikipedia.org/wiki/Multiversion_concurrency_control)
- 基于锁的并发控制，Lock-Based Concurrency Control
	- 乐观锁
	- 悲观锁
- 其他
	- unique key
	- 可重入
	- 阻塞 & 非阻塞锁


## 二、MySQL锁机制

### 2.1 乐观锁实现机制

乐观锁优点程序实现，不会存在死锁等问题

#### 适用场景？

### 2.2 悲观锁实现机制

悲观锁是数据库实现，会阻止一切数据库操作


## 三、数据库锁使用场景

### 3.1 实现分布式锁

#### 方法一：基于数据库表

要实现分布式锁，最简单的方式可能就是直接创建一张锁表，然后通过操作该表中的数据来实现了。当我们要锁住某个方法或资源时，我们就在该表中增加一条记录，想要释放锁的时候就删除这条记录。

创建这样一张数据库表：

```
CREATE TABLE `methodLock` (
	`id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
	`method_name` varchar(64) NOT NULL DEFAULT '' COMMENT '锁定的方法名',
	`desc` varchar(1024) NOT NULL DEFAULT '备注信息',
	`update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '保存数据时间，自动生成',
	PRIMARY KEY (`id`),
	UNIQUE KEY `uidx_method_name` (`method_name `) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='锁定中的方法'
```

当我们想要锁住某个方法时，执行以下SQL：

```
insert into methodLock(method_name,desc) values (‘method_name’,‘desc’)
```

可能存在的问题：

- 这把锁强依赖数据库的可用性，数据库是一个单点，一旦数据库挂掉，会导致业务系统不可用
- 这把锁没有失效时间，一旦解锁操作失败，就会导致锁记录一直在数据库中，其他线程无法再获得到锁
- 这把锁只能是非阻塞的，因为数据的insert操作，一旦插入失败就会直接报错。没有获得锁的线程并不会进入排队队列，要想再次获得锁就要再次触发获得锁操作
- 这把锁是非重入的，同一个线程在没有释放锁之前无法再次获得该锁。因为数据中数据已经存在了
- 还有一个问题，就是我们要使用排他锁来进行分布式锁的lock，那么一个排他锁长时间不提交，就会占用数据库连接。一旦类似的连接变得多了，就可能把数据库连接池撑爆
- 这里还可能存在另外一个问题，虽然我们对method_name 使用了唯一索引，并且显示使用for update来使用行级锁。但是，MySql会对查询进行优化，即便在条件中使用了索引字段，但是否使用索引来检索数据是由 MySQL 通过判断不同执行计划的代价来决定的，如果 MySQL 认为全表扫效率更高，比如对一些很小的表，它就不会使用索引，这种情况下 InnoDB 将使用表锁，而不是行锁。如果发生这种情况就悲剧了


数据库实现分布式锁的优缺点：

- 优点
	- 直接借助数据库，容易理解
- 缺点
	- 会有各种各样的问题，在解决问题的过程中会使整个方案变得越来越复杂
	- 操作数据库需要一定的开销，性能问题需要考虑
	- 使用数据库的行级锁并不一定靠谱，尤其是当我们的锁表并不大的时候



## 参考文档

- [分布式锁的几种实现方式~](http://www.hollischuang.com/archives/1716)
- [MySQL 加锁处理分析](http://hedengcheng.com/?p=771)
- [MySQL锁机制](http://www.jianshu.com/p/fb5fe63e15f4)
