---
title: Mysql资料整理
tags:
---

查看mysql配置
show variables like '%version%'
show variables like '%isolation%'

查看mysql状态
show status like '%thread%'
show status like '%connection%'

查看mysql客户端连接信息
show full processlist

查看mysql表的详细信息
show table status like '%table_name%'

mysql的逻辑架构
 - 客户端
 - 服务器
 - 存储引擎: InnoDB MyISAM

客户端请求的并发控制
 - 读写锁
 - 锁的粒度: 表锁与行锁

事务
 - ACID特性
 - 隔离级别: 一个事务中所做的修改，对于其他事务的可见性
   - READ UNCOMMITED 未提交读 发生脏读的情况
   - READ COMMITED 提交读 发生不可重复读的情况
   - REPEATABLE READ 可重复读 解决了脏读的问题，还使用MVCC多版本控制来解决幻读的问题
   - SERIALIZABLE 可串行化 事务串行执行
 - 死锁: 多个事务互相等待对方占用的锁
 - 使用事务日志来提高事务的执行效率，不直接改磁盘上的数据，而是记录事务日志，然后在后台慢慢讲数据刷回到磁盘
 - AUTOCOMMIT 如果没有显式开始事务，则自动将每一个sql操作当做事务进行提交

多版本并发控制: 保存数据在某个时间点的快照 乐观锁与悲观锁
  - InnoDB在每行记录中添加额外的两个隐藏的列来实现MVCC

存储引擎
 - InnoDB: 
 - MyISAM