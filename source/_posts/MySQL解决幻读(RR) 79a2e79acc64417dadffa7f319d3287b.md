---
title: MySQL解决幻读(RR)
date: 2024-06-20
description: MySQL在RR事务隔离级别下解决幻读问题
categories: MySQL
mathjax: true
---
# MySQL解决幻读(RR)

## MySQL事务的隔离级别

先查看当前数据库隔离级别

```sql
SELECT @@tx_isolation;   //查看当前会话的事务隔离级别
SELECT @@global.tx_isolation;  //查看全局的事务隔离级别
```

### **Repeatable Read**

![Untitled](/pic/MySQL%E8%A7%A3%E5%86%B3%E5%B9%BB%E8%AF%BB(RR)%2079a2e79acc64417dadffa7f319d3287b/Untitled.png)

当前会话隔离级别是RR，但是它不会产生幻读。可以看一下方的执行过程：

![Untitled](/pic/MySQL%E8%A7%A3%E5%86%B3%E5%B9%BB%E8%AF%BB(RR)%2079a2e79acc64417dadffa7f319d3287b/Untitled%201.png)

当事务A执行了`update`后，事务B执行`insert into` 是会被阻塞，直到事务A提交，如果阻塞等待时间过长还会出现`ERROR`

![Untitled](/pic/MySQL%E8%A7%A3%E5%86%B3%E5%B9%BB%E8%AF%BB(RR)%2079a2e79acc64417dadffa7f319d3287b/Untitled%202.png)

**MySQL是如何解决幻读的呢？**

来看一下间隙锁（Next-key Lock）MySQL InnoDB支持三种行锁定方式：**InnoDB的默认加锁方式是next-key 锁。**

- 行锁（Record Lock）:锁直接加在索引记录上面，锁住的是key。
- 间隙锁（Gap Lock）:锁定索引记录间隙，确保索引记录的间隙不变。间隙锁是针对事务隔离级别为可重复读或以上级别而已的。
- Next-Key Lock ：行锁和间隙锁组合起来就叫Next-Key Lock。

默认情况下，InnoDB工作在可重复读(Repeatable Read)隔离级别下，并且会以Next-Key Lock的方式对数据行进行加锁，这样可以有效防止幻读的发生。Next-Key Lock是行锁和间隙锁的组合，当InnoDB扫描索引记录的时候，会首先对索引记录加上行锁（Record Lock），再对索引记录两边的间隙加上间隙锁（Gap Lock）。加上间隙锁之后，其他事务就不能在这个间隙修改或者插入记录。 read committed隔离级别下

**Gap Lock在InnoDB的唯一作用就是防止其他事务的插入操作，以此防止幻读的发生。**

**Innodb自动使用间隙锁的条件：**

**（1）必须在Repeatable Read级别下**

**（2）检索条件必须有索引（没有索引的话，mysql会全表扫描，那样会锁定整张表所有的记录，包括不存在的记录，此时其他事务不能修改不能删除不能添加）**

[Mysql 间隙锁原理，以及Repeatable Read隔离级别下可以防止幻读原理(百度) - aspirant - 博客园 (cnblogs.com)](https://www.cnblogs.com/aspirant/p/9177978.html)