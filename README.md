# docker-mysql-master-slave

- - - 

说明一下，这里是在MySQL5.7.28这个镜像中测试通过，测试为两台台式机。

1. 首先通过仓库的`docker-compose.yml`文件启动MySQL镜像

2. 如果是从机，需要使用MySQL配置文件中，相应的配置文件

3. 在启动的时候，第一次请保证data目录为空目录，否则`docker-compose.yml`中的MySQL默认`root`密码失效

4. 连接MySQL相应的服务，在要作为从机的服务上执行。

```sql
# 新建同步帐号,用于连接MySQL主服务，所以应该在主服务上执行
grant replication slave on *.* to 'sync'@'%' identified by 'sync';  
# 刷新权限缓存
flush privileges 

# 在主服务上执行
show master status;

# 配置和主服务关联，在从机上执行，binlog日志和logpos，从上一步结果中获取
change master to master_host='${host}',master_port='${port}',master_user= '${master_user}' ,master_password='${master_password}',master_log_file='${master_log_file}',master_log_pos='${master_log_pos}';
# 启动从服务
start slave
# 查看服务状态
show slave status\G

# example
change master to master_host='192.168.100.37',master_port=3308,master_user='sync',master_password='sync',master_log_file='mysql-bin.000003',master_log_pos=591;

```

== 如果是双主结构，只需要在配置关联哪里，在两台服务器相互关联为对方的主服务，就好了 ==

### 其中一方down机处理方法

需要先`reset slave` 然后`start slave`才能恢复正常

### 锁表

同步之前，最好是保证主数据库处于只读状态
```sql
#锁定数据库所有表，只能读，不能写
FLUSH TABLES WITH READ LOCK;
#解锁,直接关闭客户端session，也可以解锁。
unlock tables;

```

