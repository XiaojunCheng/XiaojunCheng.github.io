# Linux内存优化

## 一、基本概念

### `Page Cache` vs `Buffer Cache`

Page Cache针对文件系统，VFS文件层cache，ext3文件格式下page cache大小为4k

linux kernel 2.4之后Buffer cache 和 page cache归并到一起了，都使用Radix tree进行管理

### `dirty pages` 脏页

需要被写到存储介质而目前存储在page cache上的数据，会被os周期性刷到存储介质上

### `writeback`

Page cache —> Storage

`Linux 2.6.31`

后台pdflush进程

`Linux 2.6.32`

后新的更有效的writeback mechanism（provide threads for eache device）

flush-MAJOR

- [Page Cache](http://calvin1978.blogcn.com/articles/kafkaio.html)
- [Linux 2.6.32](https://kernelnewbies.org/Linux_2_6_32#head-72c3f91947738f1ea52f9ed21a89876730418a61)
- [Page Cache](https://en.wikipedia.org/wiki/Page_cache)
- [Linux Page Cache Basics](https://www.thomas-krenn.com/en/wiki/Linux_Page_Cache_Basics)
- [Improving Linux performance by preserving Buffer Cache State](http://insights.oetiker.ch/linux/fadvise.html)
- [Flushing out pdflush](http://lwn.net/Articles/326552/)
- [In defense of per-BDI writeback](http://lwn.net/Articles/354851/)
- [Linux Writeback机制分析](http://blog.csdn.net/myarrow/article/details/8918944)

page cache & mmap
1、kernel如何管理虚拟内存
2、file & memory
3、虚拟地址 virtual address
     必须已page size为单位，内存则只能以页为单位

测试Page Cache

```
wfischer@pc:~$ dd if=/dev/zero of=testfile.txt bs=1M count=10
10+0 records in
10+0 records out
10485760 bytes (10 MB) copied, 0,0121043 s, 866 MB/s
wfischer@pc:~$ cat /proc/meminfo | grep Dirty
Dirty:             10260 kB
wfischer@pc:~$ sync
wfischer@pc:~$ cat /proc/meminfo | grep Dirty
Dirty:                 0 kB
```

`Up to Version 2.6.31 of the Kernel: pdflush`

Up to and including the 2.6.31 version of the Linux kernel, the pdflush threads ensured that dirty pages were periodically written to the underlying storage device.

`As of Version 2.6.32: per-backing-device based writeback`

Since pdflush had several performance disadvantages, Jens Axboe developed a new, more effective writeback mechanism for Linux Kernel version 2.6.32. [2]

This approach provides threads for each device, as the following example of a computer with an SSD (/dev/sda) and a hard disk (/dev/sdb) shows.

```
root@pc:~# ls -l /dev/sda
brw-rw---- 1 root disk 8, 0 2011-09-01 10:36 /dev/sda
root@pc:~# ls -l /dev/sdb
brw-rw---- 1 root disk 8, 16 2011-09-01 10:36 /dev/sdb
root@pc:~# ps -eaf | grep -i flush
root       935     2  0 10:36 ?        00:00:00 [flush-8:0]
root       936     2  0 10:36 ?        00:00:00 [flush-8:16]
```

read ahead（preloads）

知识点

- 共享内存实现机制
- 内存对齐
- mmap的实现原理与机制[x]
    - 注意：当一个进程使用mmap方式映射内存并写映射数据
    - mmap需要一次CPU拷贝操作，另外映射操作也是一个开销很大的虚拟存储操作，这种操作需要通过更改页表以及冲刷 TLB （使得 TLB 的内容无效）来维持存储的一致性。但是，因为映射通常适用于较大范围，所以对于相同长度的数据来说，映射所带来的开销远远低于 CPU 拷贝所带来的开销
- zero copy技术[x]
    - 为了避免内核区域的一次拷贝，需要一个支持聚合操作的网络接口
    - 注意事项
        - 移植性
        - The second difference is Linux doesn't implement vectored transfers
    - 适用于应用程序地址空间不需要对所访问数据进行处理的情况
- copy on write技术
- 内存管理：http://dongxicheng.org/os/linux-memory-management-basic/

参考资料

- Efficient data transfer through zero copy：https://www.ibm.com/developerworks/linux/library/j-zerocopy/
- Zero Copy：http://www.linuxjournal.com/article/6345