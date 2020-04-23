# Transaction ACID

<!-- TOC -->

- [Transaction ACID](#transaction-acid)
    - [What is ACID](#what-is-acid)
    - [Atomicity](#atomicity)
        - [How to](#how-to)
    - [Consistency](#consistency)
        - [How to](#how-to-1)
    - [Isolation](#isolation)
        - [How to](#how-to-2)
            - [方式一：Lock](#方式一lock)
            - [方式二：MVCC](#方式二mvcc)
    - [Durability](#durability)
        - [How to](#how-to-3)

<!-- /TOC -->

## What is ACID

- A(Atomicity): 原子性
- C(Consistency): 一致性
- I(Isolation): 隔离性
- D(Durability): 持久性

## Atomicity

一个事务被视为一个不可分割的最小工作单元，整个事务中的所有操作要么全部提交成功，要么全部失败回滚。
对于一个事务来说，不可能只执行其中的一部分操作。

### How to

实现事务的原子性，首先要保证事务中的操作要么都成功，要么失败都回滚。

以`InnoDB`为例，当执行事务时，会生成对应的`undo log`，如果事务执行失败或者执行了`rollback`，根据`undo log`中的日志将数据回滚到原来的状态。

## Consistency

数据库总是从一个一致性的状态转换到另一个一致性的状态。

### How to

通过事务的原子性、持久性、隔离性来保证数据库的一致性。

- 所有的操作都是原子操作，要么都成功，要么都失败；
- 所有的操作在事务提交之后，都将永久改变数据库中的数据；
- 多个事务之间互相隔离，互不可见；

## Isolation

一个事务所做的修改在最终提交之前，对其他事务是不可见的。

### How to

事务的四种隔离级别：

- Read Uncommited(未提交读): 可能读取到其他未提交事务修改的数据
- Read Commited(提交读): 只能读取到其他事务已提交的数据
- Repeated Read(重复读): 在同一个事务内的查询都和该事务一开始查询的数据保持一致
- Serializable(串行读): 每次读都需要获得表级别共享锁，读写相互阻塞

#### 方式一：Lock

根据锁的类型，可以分为：

- Shared Lock(共享锁)

    共享锁表示对数据进行读操作，多个事务可以同时给一个对象加共享锁。

- Exclusive Lock(排他锁)
    
    排他锁表示对数据进行写操作，如果一个事务给对象加了排他锁，其他事务就不能加任何锁。

根据锁的粒度，可以分为：

- Table Lock(表锁)

    - 常见的DDL(Data Definition Language)语句，如`DELETE TABLE`, `ALTER TABLE`
    - Intention Shared Lock(意向共享锁), Intention Exclusive Lock(意向排他锁)

        在给一行记录加锁前，首先给表加意向锁，表示该事务有意向读或写某行记录；意向锁之间不会发生冲突，真正的冲突是在加行锁时检查。
    - Auto-Inc Lock(自增锁)
    
        在`InnoDB`中，每个含有自增列的表都有一个自增长计数器，在插入的时候会执行`select max(auto_inc_col) from t for update`来获取计数器，并立即释放锁。

- Row Lock(行锁)

    `InnoDB`的行锁是在索引上的索引项加锁实现（Oracle则通过在数据块中对应数据行加锁）。即只有通过索引检索数据，才会使用行锁，否则使用表锁。

    - Record Lock(记录锁): 单行记录上锁
    - Gap Lock(间隙锁): 锁定一个范围，不包括记录本身
    - Next-Key Lock(临键锁): 锁定一个范围，并锁定记录本身
    
    此外还有：
    
    - Insert Intention Lock(插入意向锁)
    
        它的本质还是`Gap Lock`。当多个事务同时插入同一块锁定区间，只要数据行之间不发生冲突（如主键、唯一索引），那么事务之间就不会冲突产生阻塞。
    
#### 方式二：MVCC

MVCC(Multi Version Concurrency Control): 多版本并发控制。好处是读不加锁，读写不冲突，并发性能好。根据数据的隐藏列(数据版本号，删除时间，指向`undo log`的指针...)和`undo log`来实现，在读取数据时，根据隐藏列判断是否回滚，并找到回滚需要的`undo log`。

## Durability

一旦事务提交，则它对数据库的改变是永久性的。

### How to

以`InnoDB`为例，当数据修改时，会在`redo log`中记录操作，事务提交之后，会调用`fsync`将`redo log`数据刷新到磁盘；如果发生宕机，只需要重启后读取`redo log`中的数据进行恢复。

`redo log`采用`WAL(Write-ahead logging)`预写式日志，所有操作先写入日志，再进行更新。