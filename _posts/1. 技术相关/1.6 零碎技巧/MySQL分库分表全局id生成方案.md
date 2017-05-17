# MySQL分库分表全局id生成方案

## 一、背景

## 二、现有解决方案

包括实现分析及优缺点比较

## 三、感想



## 附录一：前置知识点

### unique key使用

### replace into

使用说明：

```
replace into是insert into的增强版,在向表中插入数据时，我们经常会遇到这样的情况：
1. 首先判断数据是否存在
2. 如果不存在，则插入；
3. 如果存在，则更新
```

和insert into区别：

```
replace into首先尝试插入数据到表中
1. 如果发现表中已经有此行数据（根据主键或者唯一索引判断）则先删除此行数据，然后插入新的数据
2. 否则直接插入新数据
```

[使用陷阱 - master/slave](https://blog.xupeng.me/2013/10/11/mysql-replace-into-trap/)

### insert ... on duplicate key update

[MySQL官方文档](https://dev.mysql.com/doc/refman/5.7/en/insert-on-duplicate.html)

### mysqlid

是个啥？

## 附录二：参考资料

[twitter解决方案](https://github.com/twitter/snowflake)
[flickr解决方案](http://code.flickr.net/2010/02/08/ticket-servers-distributed-unique-primary-keys-on-the-cheap/)


    