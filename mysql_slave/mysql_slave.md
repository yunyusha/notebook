## 一、主数据库master修改

### 1、修改mysql配置文件
```
# 日志文件名
log-bin = mysql-bin
# 主数据库端ID号
server-id = 1
```
### 2、重启mysql，创建同步账户
```
# 创建slave帐号slave_account，密码123456
mysql>create user 'slave_account'@'%' identified by '123456';

# 配置slave账号权限
mysql>grant replication slave on *.* to 'slave_account'@'%';

# 更新数据库权限
mysql>flush privileges;

```

### 3、查询master的状态
```
mysql>show master status;
+---------------+----------+--------------+------------------+-------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+---------------+----------+--------------+------------------+-------------------+
| binlog.000002 |     1301 |              |                  |                   |
+---------------+----------+--------------+------------------+-------------------+
```
执行后，不再操作主数据库，防止数据库状态值变化

## 二、修改数据库slave

### 1、修改mysql配置文件
```
# 主数据库端ID号
server-id = 2
```

### 2、执行同步命令

```
# 执行同步命令，设置主数据库ip，同步帐号密码，同步位置
mysql>change master to master_host='主数据库ip',master_pord='主数据端口号',master_user='slave_account',master_password='123456',master_log_file='mysql-bin.000009',master_log_pos=196;

# 开启同步功能
mysql>start slave;
```

### 3、检查是否配置成功

```
show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 175.24.75.110
                  Master_User: slave_account
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000001
          Read_Master_Log_Pos: 155
               Relay_Log_File: 73bc8f9266ca-relay-bin.000004
                Relay_Log_Pos: 323
        Relay_Master_Log_File: mysql-bin.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB:
          Replicate_Ignore_DB:
```
检查
```
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```

## 三、其他相关参数
#### 1、master端：
 ```
 # 不同步哪些数据库
binlog-ignore-db = mysql
binlog-ignore-db = test
binlog-ignore-db = information_schema

# 只同步哪些数据库，除此之外，其他不同步
binlog-do-db = game

# 日志保留时间
expire_logs_days = 10

# 控制binlog的写入频率。每执行多少次事务写入一次
# 这个参数性能消耗很大，但可减小MySQL崩溃造成的损失
sync_binlog = 5

# 日志格式，建议mixed
# statement 保存SQL语句
# row 保存影响记录数据
# mixed 前面两种的结合
binlog_format = mixed
```
#### 2、slave端
```
# 停止主从同步
mysql> stop slave;

# 连接断开时，重新连接超时时间
mysql> change master to master_connect_retry=50;

# 开启主从同步
mysql> start slave;
```
