# MySQL auto_increment实现机制

## 一、实现原理

在传统的auto_increment设置中,每次访问auto-increment计数器的时候, INNODB都会加上一个名为AUTO-INC锁直到该语句结束(注意锁只持有到语句结束,不是事务结束).AUTO-INC锁是一个特殊的表级别的锁,用来提升包含auto_increment列的并发插入性能

## 二、使用中的注意事项


## 附录：前置知识点

### MySQL锁机制

[MySQL auto_increment间隙问题](http://www.jianshu.com/p/cca59b515e20)
[MySQL insert锁机制](http://yeshaoting.cn/article/database/mysql%20insert%E9%94%81%E6%9C%BA%E5%88%B6/)