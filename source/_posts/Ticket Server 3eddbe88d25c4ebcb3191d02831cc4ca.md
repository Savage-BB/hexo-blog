---
title: Ticket Server
date: 2024-06-19
description: Flickr 公司的ID生成方案，这是一种成本很小的分布式唯一主键生成方案，主要是用MySQL自增来生成。
categories: 系统设计
tag: 
  - 分布式ID
mathjax: true
---
# Ticket Server

[Ticket Servers: Distributed Unique Primary Keys on the Cheap | code.flickr.com](https://code.flickr.net/2010/02/08/ticket-servers-distributed-unique-primary-keys-on-the-cheap/)

Flickr 公司的ID生成方案比较简单，正如他们描述的，这是一种成本很小的分布式唯一主键生成方案，主要是用MySQL自增来生成。

### REPLACE INTO

来看一下[`REPLACE INTO`](https://dev.mysql.com/doc/refman/8.4/en/replace.html)

> *REPLACE works exactly like INSERT, except that if an old row in the table has the same value as a new row for a PRIMARY KEY or a UNIQUE index, the old row is deleted before the new row is inserted.*
> 

`REPLACE` 和`INSERT` 几乎一样，唯一的区别是新增加的数据行和旧的数据行，如果有相同的主键或唯一索引时，则会删除旧数据行，插入新数据行，此时新的数据行就会有新的自增ID

### Tickets Server

Flickr Ticket服务器只是有一个数据库的独立服务器，每个数据库中有类似Ticket32和Ticket64的表（分别用于提供32位整型主键和64位长整型主键）。

`Tickets64` 表大概像这样

```sql
CREATE TABLE `Tickets64` (
  `id` bigint(20) unsigned NOT NULL auto_increment,
  `stub` char(1) NOT NULL default '',
  PRIMARY KEY  (`id`),
  UNIQUE KEY `stub` (`stub`)
) ENGINE=InnoDB
```

每次需要获取ID时，通过下面的命令就可以获取到一个最新的ID

```sql
REPLACE INTO Tickets64(stub)VALUES('a');
SELECT LAST_INSERT_ID();

+-------------------+
| LAST_INSERT_ID()  | 
+-------------------+
| 72157623227190423 |     
+-------------------+
```

### **SPOFs（Single Point of Failure）**

为了避免单点失败实现“高可用”，可以提供两个或多个MySQL服务器，通过设置起始值和自增步长来获取ID，如果有两台服务器可以这样设置：

```sql
TicketServer1:
auto-increment-increment = 2
auto-increment-offset = 1

TicketServer2:
auto-increment-increment = 2
auto-increment-offset = 2
```

```sql
SHOW VARIABLES LIKE 'AUTO_INC%';  -- 查看increment和offset
SET @@AUTO_INCREMENT_OFFSET=1;    -- 设置ID的起始值
SET @@AUTO_INCREMENT_INCREMENT=2;  -- 设置ID每次增加的ID数
```

两台服务器中如果有一台服务器宕机，那么获取的ID是奇数或者偶数，仍然能保证它们是自增的，对业务不会有影响。

### 总结

正如Flickr团队说的，这种主键生成方案可能并不优雅（It’s not particularly elegant），但它是“cheap”的，没有太复杂的算法和规则，只需要服务器和MySQL就可以做到。这个设计已经成为了Flickr工程师的设计理念—” [dumbest possible thing that will work](http://laughingmeme.org/2009/09/29/try-coding-dear-boy/)”