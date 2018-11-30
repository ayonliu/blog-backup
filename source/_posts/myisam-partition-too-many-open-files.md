---
title: MySQL MyISAM 分区（partition）导致打开文件过多
tags:
- mysql
---

## 问题表现

最近公司一个业务环境的数据库，出现无法连接的情况，重启后过几个小时问题又出现。查看报错信息，如下所示：
```
Out of resources when opening file './xxx.MYD' (Errcode: 24 - Too many open files)
```
问题很明显：打开文件过多，超过了配置的限制。

## 问题分析

查看 table_open_cache, open_files_limit 当前配置的值：
``` bash
mysql> show variables like '%open_%';
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| have_openssl               | YES   |
| innodb_open_files          | 2000  |
| open_files_limit           | 5000  |
| table_open_cache           | 2000  |
| table_open_cache_instances | 16    |
+----------------------------+-------+
5 rows in set (0.01 sec)
```
<!--more-->
查看当前 table 打开的情况：
``` bash
mysql> show global status like 'Open%tables';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| Open_tables   | 500   |
| Opened_tables | 7841  |
+---------------+-------+
2 rows in set (0.01 sec)
```
可以看到 Open_tables 还未达到 table_open_cache 的值，因此随着的表的不断打开，文件描述符也不断的被打开，直到达到 table_open_cache 设定的值才会释放。
对比查看系统中 mysql 打开的文件数量：`lsof -u mysql | wc -l`，持续跟踪可以发现这个数字一直在上升，很快要接近 open_files_limit 的值，最终导致“Too many open files”。

官方文档对此也有描述：[8.4.3.1 How MySQL Opens and Closes Tables](https://dev.mysql.com/doc/refman/5.6/en/table-cache.html)
> Make sure that your operating system can handle the number of open file descriptors implied by the table_open_cache setting. If table_open_cache is set too high, MySQL may run out of file descriptors and refuse connections, fail to perform queries, and be very unreliable.

## 问题解决方法

根据实际情况有2种解决方法：

- 降低 table_open_cache 的数值：
``` bash
mysql> set global table_open_cache=1000;
Query OK, 0 rows affected (0.00 sec)
```

- 加大 open_files_limit 的数值：
修改文件 `/usr/lib/systemd/system/mysqld.service`
```
# Sets open_files_limit
LimitNOFILE = 20480
```
这里要注意不能超过操作系统的打开文件数限制。

## 问题根本原因挖掘
问题的表现是 MySQL 打开文件数量过快过多，最终超过 open_files_limit 的限制，从而导致 MySQL 无法提供服务。
那是什么原因导致 MySQL 打开文件数量过快过多呢？
接合报错信息和 MySQL 打开的文件分析，发现打开某些表时，会同时打开N个文件，这就是问题所在。
- 对于MyISAM表来说，正常每打开一次表都会额外打开一个 MYD文件（MySQL Data），一个MYI文件（MySQL Index），MYI文件可以在不同session间共用；
- 对于使用了分区表（partition）的 MyISAM 表来说，每一次打开表，不管分区是否被使用，都会打开表的所有分区，例如一个表分了10个区，第一次对这个表的访问会打开20个文件，后面每次访问都会再打开10个文件

官方文档对此也有描述：[8.4.3.1 How MySQL Opens and Closes Tables](https://dev.mysql.com/doc/refman/5.6/en/table-cache.html)
> You should also take into account the fact that the MyISAM storage engine needs two file descriptors for each unique open table. For a partitioned MyISAM table, two file descriptors are required for each partition of the opened table. (Note further that when MyISAM opens a partitioned table, it opens every partition of this table, whether or not a given partition is actually used. 

## 结论
MyISAM 的分区（partition）最好别用，真需要用分表，用水平分表或垂直分表代替。
