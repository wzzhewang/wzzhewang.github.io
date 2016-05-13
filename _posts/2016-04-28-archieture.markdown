---
layout: post
title: 数据库－事务详解
category: Meta
author: 王哲

excerpt: 数据库－事务详解

---
![image](/resources/img/transaction_1.jpg)
然后说说修改事务隔离级别的方法：

1.全局修改，修改mysql.ini配置文件，在最后加上

1 #可选参数有：READ-UNCOMMITTED, READ-COMMITTED, REPEATABLE-READ, SERIALIZABLE.
2 [mysqld]
3 transaction-isolation = REPEATABLE-READ
这里全局默认是REPEATABLE-READ,其实MySQL本来默认也是这个级别

2.对当前session修改，在登录mysql客户端后，执行命令：

![image](/resources/img/transaction_2.jpg)

要记住mysql有一个autocommit参数，默认是on，他的作用是每一条单独的查询都是一个事务，并且自动开始，自动提交（执行完以后就自动结束了，如果你要适用select for update，而不手动调用 start transaction，这个for update的行锁机制等于没用，因为行锁在自动提交后就释放了），所以事务隔离级别和锁机制即使你不显式调用start transaction，这种机制在单独的一条查询语句中也是适用的，分析锁的运作的时候一定要注意这一点

#再来说说锁机制：

共享锁：由读表操作加上的锁，加锁后其他用户只能获取该表或行的共享锁，不能获取排它锁，也就是说只能读不能写

排它锁：由写表操作加上的锁，加锁后其他用户不能获取该表或行的任何锁，典型是mysql事务中

    start transaction;
    select * from user where userId = 1 for update;

执行完这句以后

　　1）当其他事务想要获取共享锁，比如事务隔离级别为SERIALIZABLE的事务，执行
     
     select * from user;

 　　将会被挂起，因为SERIALIZABLE的select语句需要获取共享锁

　　2）当其他事务执行

    select * from user where userId = 1 for update;
    update user set userAge = 100 where userId = 1; 

　　也会被挂起，因为for update会获取这一行数据的排它锁，需要等到前一个事务释放该排它锁才可以继续进行


锁的范围:

行锁: 对某行记录加上锁

表锁: 对整个表加上锁

这样组合起来就有,行级共享锁,表级共享锁,行级排他锁,表级排他锁

 

#下面来说说不同的事务隔离级别的实例效果，例子使用InnoDB，开启两个客户端A，B，在A中修改事务隔离级别，在B中开启事务并修改数据，然后在A中的事务查看B的事务修改效果：

 

1.READ-UNCOMMITTED(读取未提交内容)级别

　　1）A修改事务级别并开始事务，对user表做一次查询

　　　![image](/resources/img/transaction_3.jpg)

 
　　2)B更新一条记录

　　　
![image](/resources/img/transaction_4.jpg)
 

　　3）此时B事务还未提交，A在事务内做一次查询，发现查询结果已经改变

![image](/resources/img/transaction_5.jpg)

 

　　4）B进行事务回滚

　![image](/resources/img/transaction_6.jpg)　　

 

　　5）A再做一次查询，查询结果又变回去了

　　　
![image](/resources/img/transaction_7.jpg)
 

　　6）A表对user表数据进行修改

　![image](/resources/img/transaction_8.jpg)　　

 

　　7）B表重新开始事务后，对user表记录进行修改，修改被挂起，直至超时，但是对另一条数据的修改成功，说明A的修改对user表的数据行加行共享锁(因为可以使用select)

　![image](/resources/img/transaction_9.jpg)　　

 

　　可以看出READ-UNCOMMITTED隔离级别，当两个事务同时进行时，即使事务没有提交，所做的修改也会对事务内的查询做出影响，这种级别显然很不安全。但是在表对某行进行修改时，会对该行加上行共享锁

 

2. READ-COMMITTED（读取提交内容）

　　1）设置A的事务隔离级别，并进入事务做一次查询

　　　![image](/resources/img/transaction_10.jpg)

 

　　2）B开始事务，并对记录进行修改

　　　![image](/resources/img/transaction_11.jpg)

 

　　3）A再对user表进行查询，发现记录没有受到影响

　　　![image](/resources/img/transaction_12.jpg)

 

　　4）B提交事务

　　　![image](/resources/img/transaction_13.jpg)

 

　　5）A再对user表查询，发现记录被修改

　　　![image](/resources/img/transaction_14.jpg)

 

　　6）A对user表进行修改

　　　![image](/resources/img/transaction_15.jpg)

 

　　7）B重新开始事务，并对user表同一条进行修改，发现修改被挂起，直到超时，但对另一条记录修改，却是成功，说明A的修改对user表加上了行共享锁(因为可以select)

　　　![image](/resources/img/transaction_16.jpg)
　　　
     ![image](/resources/img/transaction_17.jpg)
　　　

 

　　READ-COMMITTED事务隔离级别，只有在事务提交后，才会对另一个事务产生影响，并且在对表进行修改时，会对表数据行加上行共享锁

 

3. REPEATABLE-READ(可重读)

　　1）A设置事务隔离级别，进入事务后查询一次

　　　![image](/resources/img/transaction1_1.jpg)

 

　　2）B开始事务，并对user表进行修改

　　　![image](/resources/img/transaction1_2.jpg)

 

　　3）A查看user表数据，数据未发生改变

　　　![image](/resources/img/transaction1_3.jpg)

 

　　4）B提交事务

　　　![image](/resources/img/transaction1_4.jpg)

 

　　5）A再进行一次查询，结果还是没有变化

　　　
![image](/resources/img/transaction1_5.jpg)
 

　　6）A提交事务后，再查看结果，结果已经更新

　　　![image](/resources/img/transaction1_6.jpg)

 

　　7）A重新开始事务，并对user表进行修改
![image](/resources/img/transaction1_7.jpg)
　　　

　　　

　　8）B表重新开始事务，并对user表进行修改，修改被挂起，直到超时，对另一条记录修改却成功，说明A对表进行修改时加了行共享锁(可以select)

　　　![image](/resources/img/transaction1_8.jpg)

　　　![image](/resources/img/transaction1_9.jpg)

 

　　REPEATABLE-READ事务隔离级别，当两个事务同时进行时，其中一个事务修改数据对另一个事务不会造成影响，即使修改的事务已经提交也不会对另一个事务造成影响。

　　在事务中对某条记录修改，会对记录加上行共享锁，直到事务结束才会释放。

 

4.SERIERLIZED(可串行化)

　　1）修改A的事务隔离级别，并作一次查询

　　　![image](/resources/img/transaction2_1.jpg)

　　2）B对表进行查询，正常得出结果，可知对user表的查询是可以进行的

　　　![image](/resources/img/transaction2_2.jpg)

 

　　3）B开始事务，并对记录做修改，因为A事务未提交，所以B的修改处于等待状态，等待A事务结束，最后超时，说明A在对user表做查询操作后，对表加上了共享锁

　　　![image](/resources/img/transaction2_3.jpg)

 

　　SERIALIZABLE事务隔离级别最严厉，在进行查询时就会对表或行加上共享锁，其他事务对该表将只能进行读操作，而不能进行写操作。

 