mysql timeout

查一下看有哪些超时

show global variables like '%timeout%';

## connect\_timeout

**手册描述:**  
The number of seconds that the mysqld server waits for a connect packet before responding with Bad handshake. The default value is 10 seconds as of MySQL 5.1.23 and 5 seconds before that.  
Increasing the connect\_timeout value might help if clients frequently encounter errors of the form Lost connection to MySQL server at ‘XXX’, system error: errno.  
**解释：在获取链接时，等待握手的超时时间，只在登录时有效，登录成功这个参数就不管事了。主要是为了防止网络不佳时应用重连导致连接数涨太快，一般默认即可。**  


## delayed\_insert\_timeout

**手册描述：**  
How many seconds an INSERT DELAYED handler thread should wait for INSERT statements before terminating.  
**解释：这是为MyISAM INSERT DELAY设计的超时参数，在INSERT DELAY中止前等待INSERT语句的时间。**  


## innodb\_lock\_wait\_timeout

**手册描述：**  
The timeout in seconds an InnoDB transaction may wait for a row lock before giving up. The default value is 50 seconds. A transaction that tries to access a row that is locked by another InnoDB transaction will hang for at most this many seconds before issuing the following error:

```
ERROR 
1205
(
HY000
):
Lock
 wait timeout exceeded
;
try
 restarting transaction
```

When a lock wait timeout occurs, the current statement is not executed. The current transaction is not rolled back. \(To have the entire transaction roll back, start the server with the –innodb\_rollback\_on\_timeout option, available as of MySQL 5.1.15. See also Section 13.6.12, “InnoDB Error Handling”.\)  
innodb\_lock\_wait\_timeout applies to InnoDB row locks only. A MySQL table lock does not happen inside InnoDB and this timeout does not apply to waits for table locks.  
InnoDB does detect transaction deadlocks in its own lock table immediately and rolls back one transaction. The lock wait timeout value does not apply to such a wait.  
For the built-in InnoDB, this variable can be set only at server startup. For InnoDB Plugin, it can be set at startup or changed at runtime, and has both global and session values.  
**解释：描述很长，简而言之，就是事务遇到锁等待时的Query超时时间。跟死锁不一样，InnoDB一旦检测到死锁立刻就会回滚代价小的那个事务，锁等待是没有死锁的情况下一个事务持有另一个事务需要的锁资源，被回滚的肯定是请求锁的那个Query。**  


## innodb\_rollback\_on\_timeout

**手册描述：**  
In MySQL 5.1, InnoDB rolls back only the last statement on a transaction timeout by default. If –innodb\_rollback\_on\_timeout is specified, a transaction timeout causes InnoDB to abort and roll back the entire transaction \(the same behavior as in MySQL 4.1\). This variable was added in MySQL 5.1.15.  
**解释：这个参数关闭或不存在的话遇到超时只回滚事务最后一个Query，打开的话事务遇到超时就回滚整个事务。**  


## interactive\_timeout/wait\_timeout

**手册描述：**  
The number of seconds the server waits for activity on an interactive connection before closing it. An interactive client is defined as a client that uses the CLIENT\_INTERACTIVE option to mysql\_real\_connect\(\). See also  
**解释：一个持续SLEEP状态的线程多久被关闭。线程每次被使用都会被唤醒为acrivity状态，执行完Query后成为interactive状态，重新开始计时。wait\_timeout不同在于只作用于TCP/IP和Socket链接的线程，意义是一样的。**  


## net\_read\_timeout / net\_write\_timeout

**手册描述：**  
The number of seconds to wait for more data from a connection before aborting the read. Before MySQL 5.1.41, this timeout applies only to TCP/IPconnections, not to connections made through Unix socket files, named pipes, or shared memory. When the server is reading from the client, net\_read\_timeout is the timeout value controlling when to abort. When the server is writing to the client, net\_write\_timeout is the timeout value controlling when to abort. See also slave\_net\_timeout.  
On Linux, the NO\_ALARM build flag affects timeout behavior as indicated in the description of the net\_retry\_count system variable.  
**解释：这个参数只对TCP/IP链接有效，分别是数据库等待接收客户端发送网络包和发送网络包给客户端的超时时间，这是在Activity状态下的线程才有效的参数**  


## slave\_net\_timeout

**手册描述：**  
The number of seconds to wait for more data from the master before the slave considers the connection broken, aborts the read, and tries to reconnect. The first retry occurs immediately after the timeout. The interval between retries is controlled by the MASTER\_CONNECT\_RETRY option for the CHANGE MASTER TO statement or –master-connect-retry option, and the number of reconnection attempts is limited by the –master-retry-count option. The default is 3600 seconds \(one hour\).  
**解释：这是Slave判断主机是否挂掉的超时设置，在设定时间内依然没有获取到Master的回应就人为Master挂掉了**

