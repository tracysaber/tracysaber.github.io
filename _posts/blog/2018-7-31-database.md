---
layout: post
title: 数据库原理基础
categories: Blog
description: 没有靠谱描述
keywords: database
---
数据库原理基础

# 数据库设计

|Sno	  |Sname	|Sdept	  |Mname	|Cname	   | Grade  |
| :-----: | :------:| :-----: | :-----: | :-----: | :-----: |
|1	      |学生-1	|学院-1	  |院长-1	|课程-1	   |90      |
|2	      |学生-2	|学院-2	  |院长-2	|课程-2	   |80      |
|2	      |学生-2	|学院-2	  |院长-2	|课程-1	   |100     |
|3	      |学生-3	|学院-2	  |院长-2	|课程-2	   |95      |

## 异常
不符合范式的关系，会产生很多异常，主要有以下四种异常：
* 冗余数据：例如 学生-2 出现了两次。
* 修改异常：修改了一个记录中的信息，但是另一个记录中相同的信息却没有被修改。
* 删除异常：删除一个信息，那么也会丢失其它信息。例如如果删除了 课程-1，需要删除第一行和第三行，那么学生-1 的信息就会丢失。
* 插入异常，例如想要插入一个学生的信息，如果这个学生还没选课，那么就无法插入。

## 范式(Normal Form)
范式理论是为了解决以上提到四种异常。
高级别范式的依赖于低级别的范式，1NF 是最低级别的范式。
### 第一范式
属性不可分。
### 第二范式
每个非主属性完全函数依赖于键码。
可以通过分解来满足。（消除部分依赖）
### 第三范式
非主属性不传递函数依赖于键码。
### BC范式
所有属性都不传递函数依赖于键码。

# 数据库事务

## 事务
满足ACID特性的一组操作
* Automicity 原子性，最小的操作单元，不可分割
* Consistency 一致性，事务执行前后都是一致的
* Isolation 隔离性，事务的修改提交前，对其他事务都是不可见的
* Duraibility 持久性，修改后会永久保存到数据库中

## 并发一致性
### 丢失修改
T1,T2都对同一个数据进行修改，先修改的更新像是丢失了一样
### 读脏数据
T1修改某个数据后撤回，导致T2读到了脏数据
### 不可重复读
T2读取一个数据后T1进行了修改，T2再也读不到相同的数据了
### 幻影读
T1读取某个范围的数据后，T2向这个范围内插入了新数据

## 封锁

## 隔离级别

|   	   |脏读	 |不可重复读 |幻影读   |
| :-----:  | :------:| :-----:   | :-----: |
|未提交读  |YES	     |YES	     |YES	   |
|提交读	   |NO	     |YES	     |YES	   |
|可重复读  |NO	     |NO	     |YES	   |
|可串行化  |NO	     |NO	     |NO	   |

MYSQL默认的隔离级别是可重复读

# MySQL引擎介绍
Mysql在V5.1之前默认存储引擎是MyISAM；在此之后默认存储引擎是InnoDB。
## InnoDB引擎
Innodb引擎提供了对数据库ACID事务的支持，并且实现了SQL标准的四种隔离级别。该引擎还提供了行级锁和外键约束，它的设计目标是处理大容量数据库系统，它本身其实就是基于MySQL后台的完整数据库系统，MySQL运行时Innodb会在内存中建立缓冲池，用于缓冲数据和索引。但是该引擎不支持FULLTEXT类型的索引，而且它没有保存表的行数，当SELECT COUNT(*) FROM TABLE时需要扫描全表。当需要使用数据库事务时，该引擎当然是首选。由于锁的粒度更小，写操作不会锁定全表，所以在并发较高时，使用Innodb引擎会提升效率。但是使用行级锁也不是绝对的，如果在执行一个SQL语句时MySQL不能确定要扫描的范围，InnoDB表同样会锁全表。

### 聚集索引和辅助索引
数据库中的 B+ 树索引可以分为聚集索引（clustered index）和辅助索引（secondary index），它们之间的最大区别就是，聚集索引中存放着一条行记录的全部信息，而辅助索引中只包含索引列和一个用于查找对应行记录的『书签』。
#### 聚集索引
InnoDB 存储引擎中的表都是使用索引组织的，也就是按照键的顺序存放；聚集索引就是按照表中主键的顺序构建一颗 B+ 树，并在**叶节点**中存放表中的**行记录数据**。

聚集索引与表的物理存储方式有着非常密切的关系，所有正常的表应该有且仅有一个聚集索引（绝大多数情况下都是主键），表中的所有行记录数据都是按照聚集索引的顺序存放的。
#### 辅助索引
数据库将所有的非聚集索引都划分为辅助索引，但是这个概念对我们理解辅助索引并没有什么帮助；辅助索引也是通过 B+ 树实现的，但是它的叶节点并不包含行记录的全部数据，仅包含索引中的所有键和一个用于查找对应行记录的『书签』，在 InnoDB 中这个书签就是当前记录的主键。

辅助索引的存在并不会影响聚集索引，因为聚集索引构成的 B+ 树是数据实际存储的形式，而辅助索引只用于加速数据的查找，所以一张表上往往有多个辅助索引以此来提升数据库的性能。

### 锁
我们都知道锁的种类一般分为乐观锁和悲观锁两种，InnoDB 存储引擎中使用的就是悲观锁，而按照锁的粒度划分，也可以分成行锁和表锁。


## MyISAM引擎
MyISAM没有提供对数据库事务的支持，也不支持行级锁和外键，因此当INSERT(插入)或UPDATE(更新)数据时即写操作需要锁定整个表，效率便会低一些。不过和Innodb不同，MyISAM中存储了表的行数，于是SELECT COUNT(*) FROM TABLE时只需要直接读取已经保存好的值而不需要进行全表扫描。如果表的读操作远远多于写操作且不需要数据库事务的支持，那么MyISAM也是很好的选择。

