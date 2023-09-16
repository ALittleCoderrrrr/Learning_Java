# Learning_Java_09

### MySQL语句

1. **DDL（Data Definition Language）**：数据定义语言，用于定义数据库结构，包括创建、修改、删除表格、索引、视图等数据库对象的操作。常见的DDL命令有`CREATE TABLE`、`ALTER TABLE`、`DROP TABLE`等。DDL命令会自动提交事务。

2. **DML（Data Manipulation Language）**：数据操纵语言，用于查询、插入、更新和删除数据。它不涉及数据库结构的更改，而是处理数据库中的实际数据。常见的DML命令有`SELECT`、`INSERT`、`UPDATE`、`DELETE`等。DML命令通常在事务中使用。

3. **DCL（Data Control Language）**：数据控制语言，用于授权和权限管理。它包括`GRANT`（授予权限）和`REVOKE`（撤销权限）等命令，用于管理数据库用户的访问权限。

4. **TCL（Transaction Control Language）**：事务控制语言，用于管理数据库中的事务。TCL包括`COMMIT`（提交事务）和`ROLLBACK`（回滚事务）等命令，用于确保数据库事务的一致性和可靠性。


### MySQL 日志
https://www.cnblogs.com/semi-sub/p/14225047.html
    -   错误日志 error log:对MySQL启动、运行、关闭进行的记录。

    -   二进制日志 bin log：记录更改数据库的sql语句日志，例如Creat、 Alter、Drop Table、insert、update、delete等，但是select和show不会记录。用于主备机同步，备机重演。
        刷盘时机：随机、每次提交事务、每N次提交事务；
        重新生成bin log时机：MySQL服务启动、flush logs、文件超出大小

    -   慢查询日志 slow query log：执行时间超过long time query（默认10s）的查询语句；用于优化使用。

    -   事务日志 redo log(重做日志)：记录每个页修改了什么内容；当宕机时用于恢复未写入磁盘的数据。为什么不直接将修改的数据刷盘呢？因为一页最少16KB，即便是修改了很少一部分，也要重刷16KB。redo log用于事务级别的恢复，bin log用于数据库级别的恢复。bin log 采用追加方式写入大小无限制，redo log 采用循环方式写入，大小固定。

                undo log(回滚日志)：记录事务对数据的修改，当事务回滚时利用undo log回滚。

    -   中继日志 relay log 复制过程中产生的日志，主要是主动数据库的从库。

    -   DDL日志 (meta data log): DDL（define）语句执行的操作

### MySQL B树 B+树
    B+树真实数据都存在叶子结点，非叶子结点存放的都是键值；
    B树的记录节点只会出现一次，而B+树的键值可能会重复出现。B+树的叶子结点通过双向链表链接。
    B+树由于非叶子结点只存放键值对，导致单个页面可以存放更多的索引，使其树的高度变低，访问所需的IO次数也变少。更适合范围查找，更稳定的查找效率（树高）。
    B树和B+树都是多路平衡的查找树，而红黑树则不是平衡的，而且是二叉树树高还是太高了，不适合做数据库的索引存储结构。
    为什么不用hash表做索引呢？因为他无法实现顺序和范围查找查找。
    
### MySQL索引
    
主键索引：

在 MySQL 的 InnoDB 的表中，当没有显示的指定表的主键时，InnoDB 会自动先检查表中是否有唯一索引且不允许存在 null 值的字段，如果有，则选择该字段为默认的主键，否则 InnoDB 将会自动创建一个 6Byte 的自增主键。

<div style="text-align:center;">
主键索引
<br>
<img src="./day09_picture/0.png"  width="300" height="200">
</div>

二级索引（辅助索引），叶子结点存放的是对应主键，唯一索引，普通索引，前缀索引等索引属于二级索引。
https://mp.weixin.qq.com/s/8qemhRg5MgXs1So5YCv0fQ

### Java 自动拆装箱NPE
    NullPointError，对一个null对象执行自动拆箱会报该错误。
```java
    Integer a = null;
    int b = a;
 ```

### MySQL 是如何实现4种隔离级别的
    - 读未提交 什么同步操作都不做。
    - 读已提交  MVCC
    - 可重复读  MVCC / 锁
    - 串行化    锁

    


