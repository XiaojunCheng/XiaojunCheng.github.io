# MySQL select for update使用说明

MySQL InnoDB默认Row-Level Lock，所以只有「明确」地指定主键，MySQL 才会执行Row lock (只锁住被选取的数据) ，否则MySQL 将会执行Table Lock (将整个数据表单给锁住)

lock in share mode适用于两张表存在业务关系时的一致性要求，for  update适用于操作同一张表时的一致性要求

[innodb-locking-reads](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking-reads.html)