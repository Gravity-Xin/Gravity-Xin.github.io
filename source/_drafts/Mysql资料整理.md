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

查看mysql表使用的索引
show index from table_name

增加或删除mysql表的索引
alter table table_name add index index_title(row1, row2...)
alter table table_name drop index index_title

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
   - REPEATABLE READ 可重复读 解决了脏读的问题，还使用MVCC多版本控制来解决幻读的问题  mysql默认的隔离级别
   - SERIALIZABLE 可串行化 事务串行执行
 - 死锁: 多个事务互相等待对方占用的锁
 - 使用事务日志来提高事务的执行效率，不直接改磁盘上的数据，而是记录事务日志，然后在后台慢慢讲数据刷回到磁盘
 - AUTOCOMMIT 如果没有显式开始事务，则自动将每一个sql操作当做事务进行提交

多版本并发控制: 保存数据在某个时间点的快照 乐观锁与悲观锁
  - InnoDB在每行记录中添加额外的两个隐藏的列(创建时间和删除时间)来实现MVCC

存储引擎
 - InnoDB
   - mysql默认事务型引擎
   - 处理大量短期事务
   - 应该优先考虑使用该引擎
   - 支持行锁和表锁
   - 索引文件和数据文件相同
   - 根据主键查询为聚簇索引，索引data域为具体行数据；根据其它字段查询为非聚簇索引，索引data域为主键值，再根据主键的索引找到具体行
 - MyISAM
   - 支持表锁
   - 索引文件和数据文件分离，索引data域为行的地址，为非聚簇索引；主键索引与其它字段的索引格式一样，都是非聚簇索引

索引
 - 会降低增删改的速度，因为需要维护索引
 - 分类
   - 主键索引 PRIMARY KEY
   - 唯一索引 UNIQUE
   - 普通索引 INDEX
   - 组合索引 INDEX
   - 全文索引 FULLTEXT
 - 实现机制
   - B+树
     - 磁盘IO次数为树的高度
     - 多个树节点占用磁盘一页的空间，可以利用磁盘缓存和访问局部性原理提高IO效率
   - 哈希
     - 每次只查询出一行，不支持范围查询
 - 优化
   - 使用explain命令定位性能瓶颈
     - 是否为全表扫描
     - 用了哪些索引
   - 索引的字段应尽量的小，减少单个索引占用的磁盘空间，从而可以增加磁盘上一页中的树节点数量，从而降低B+树的高度
   - 组合索引与最左前缀匹配原理
     - 创建组合索引时，将使用最频繁的列放在左边
     - 使用组合索引查询时，where字句将使用最频繁的列放在最左边
   - 使用区分度高的列作为索引

大表优化
 - 限制查询范围
 - 读写分离
 - 缓存
 - 垂直拆分: 根据表中内容的相关性拆分为几张表
 - 水平拆分: 按照某个字段进行分片