## 基于Docker实现Mysql主从同步

### Docker搭建主从服务

1. MySQL Docker Image 安装

   ```sh
   docker pull mysql:8.0
   ```

2. 运行容器

   Master 对外映射端口 3307

   ```sh
   docker run -p 3307:3306 --name master_mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0 
   ```

   Slave 对外映射端口 3308

   ```sh
   docker run -p 3308:3306 --name slave_mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0
   ```

3. 查看正在运行的容器

   ```sh
   docker ps
   ```

   ```tex
   CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
   f1dd97714edd        mysql:8.0           "docker-entrypoint.s…"   44 seconds ago      Up 43 seconds       33060/tcp, 0.0.0.0:3308->3306/tcp   slave_mysql
   c9c213ae2ba1        mysql:8.0           "docker-entrypoint.s…"   50 seconds ago      Up 49 seconds       33060/tcp, 0.0.0.0:3307->3306/tcp   master_mysql
   ```

## 配置

### 主节点

1. 修改ids

   ```sh
   # 进入 master 容器内部
   docker exec -it master_mysql /bin/bash
   
   # 同步时间
   ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
   echo 'Asia/Shanghai' >/etc/timezone
   
   # 更换源
   mv /etc/apt/sources.list /etc/apt/sources.list.bak
   echo "deb http://mirrors.ustc.edu.cn/debian stable main contrib non-free" >> /etc/apt/sources.list
   echo "deb http://mirrors.ustc.edu.cn/debian stable-updates main contrib non-free" >>/etc/apt/sources.list
   
   # 安装vim
   apt-get update
   apt-get install vim -y
   
   # 修改 my.cnf
   vim /etc/mysql/my.cnf
   ```

2. my.cnf 添加内容:

   ```ini
   [mysqld]
   # 同一局域网内唯一ID
   server-id=1
   # 开启二进制日志功能
   log-bin=mysql-bin
   # 开启日志
   # general_log = 1
   # general_log_file = /var/log/mysql/general_sql.log
   # 需要忽略的库，忽略后不同步此库
   binlog-ignore-db=information_schema
   binlog-ignore-db=cluster
   binlog-ignore-db=mysql
   # 需要同步的库
   # binlog-do-db=test
   ```

3. 重启 master 容器

   ```sh
   docker restart master_mysql
   ```

4. 验证server id

   ```mysql
   -- 连接mysql
   mysql -h127.0.0.1 -P3307 -uroot -p123456
   
   show variables like 'server_id';
   ```

   ```tex
   +---------------+-------+
   | Variable_name | Value |
   +---------------+-------+
   | server_id     | 1     |
   +---------------+-------+
   ```

   注意：==如果不修改ID会报错==

   ```verilog
   13117: Fatal error: The slave I/O thread stops because master and slave have equal MySQL server ids; these ids must be different for replication to work (or the --replicate-same-server-id option must be used on slave but this does not always make sense; please check the manual before using it).
   ```

5. 查看 master 的 binlog，此时需要保证Master库不能做任何操作，否则将会引起状态变化

   ```mysql
   show master status;
   ```

   ```tex
   +------------------+----------+--------------+----------------------------------+-------------------+
   | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB                 | Executed_Gtid_Set |
   +------------------+----------+--------------+----------------------------------+-------------------+
   | mysql-bin.000003 |      156 |              | information_schema,cluster,mysql |                   |
   +------------------+----------+--------------+----------------------------------+-------------------+
   ```

6. 创建数据同步用户

   这用户需要授予 `slave REPLICATION SLAVE` ,`REPLICATION CLIENT` 权限，用于同步数据

   ```mysql
   CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
   GRANT REPLICATION SLAVE, REPLICATION CLIENT on *.* TO 'slave'@'%';
   ```

   可用`show grants for slave;` 来验证是否授权成功, 使用`mysql -h127.0.0.1 -P3307 -uslave -p123456` 来验证是否可以登录

### 从节点

1. 修改ids

   ```sh
   # 进入 slave 容器内部
   docker exec -it slave_mysql /bin/bash
   
   # 同步时间
   ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
   echo 'Asia/Shanghai' >/etc/timezone
   
   # 更换源
   mv /etc/apt/sources.list /etc/apt/sources.list.bak
   echo "deb http://mirrors.ustc.edu.cn/debian stable main contrib non-free" >> /etc/apt/sources.list
   echo "deb http://mirrors.ustc.edu.cn/debian stable-updates main contrib non-free" >>/etc/apt/sources.list
   
   # 安装vim
   apt-get update
   apt-get install vim -y
   
   # 修改 my.cnf
   vim /etc/mysql/my.cnf
   ```

2. my.cnf 添加内容:

   ```ini
   [mysqld]
   # 同一局域网内唯一ID
   server-id=2
   # 开启二进制日志功能，以备Slave作为其它Slave的Master时使用
   log-bin=mysql-slave-bin
   # relay_log配置中继日志
   relay_log=mysql-relay-bin
   # 开启日志
   # general_log = 1
   # general_log_file = /var/log/mysql/general_sql.log
   # 需要忽略的库，忽略后不同步此库
   binlog-ignore-db=information_schema
   binlog-ignore-db=cluster
   binlog-ignore-db=mysql
   replicate-ignore-db=mysql
   # 需要复制的库
   # replicate-do-db=ufind_db
   log-slave-updates
   slave-skip-errors=all
   slave-net-timeout=60
   ```

3. 重启 slave 容器

   ```sh
   docker restart slave_mysql
   ```

   

4. 验证server id

   ```sh
   # 连接mysql
   mysql -h127.0.0.1 -P3308 -uroot -p123456
   
   show variables like 'server_id';
   ```

   ```tex
   +---------------+-------+
   | Variable_name | Value |
   +---------------+-------+
   | server_id     | 2     |
   +---------------+-------+
   ```

   

5. 链接主节点

   查询主节点容器的独立IP

   ```sh
   docker inspect --format='{{.NetworkSettings.IPAddress}}'  master_mysql
   
   # 172.17.0.2
   ```
   
   
   
   ```mysql
   -- 链接配置
   CHANGE MASTER TO MASTER_HOST='172.17.0.2', MASTER_PORT=3306,  MASTER_LOG_FILE='mysql-bin.000003', MASTER_LOG_POS=156, MASTER_CONNECT_RETRY=30, MASTER_BIND='';
   
-- 启动
   start slave user='slave' password='123456';
-- 如果 slave 报错:  [ERROR] [MY-010584] [Repl] Slave I/O for channel , 尝试使用 root
   -- start slave user='root' password='123456';
   
   -- 查看 slave 状态，确保: Slave_IO_Running: Connecting, Slave_SQL_Running: Yes
   show slave status\G
   ```
   
   说明:
   
   ```yaml
master_host: Master的地址
   master_port: Master的端口号
   master_user: 用于数据同步的用户
   master_password: 用于同步的用户的密码
   master_log_file: 指定 Slave 从哪个日志文件开始复制数据，即上文中提到的 File 字段的值
   master_log_pos: 从哪个 Position 开始读，即上文中提到的 Position 字段的值
   master_connect_retry: 如果连接失败，重试的时间间隔，单位是秒，默认是60秒
   ```
   
   注意: ==可用`docker logs -f slave_mysql`查看连接错误时的日志==

### 测试

主节点创建数据库，查看从节点是否有同步

主:

```mysql
create database test default character set utf8;
create table test.tbl_test (`user` varchar(64) not null, age int(11) not null) default charset utf8;
insert into test.tbl_test values ('li',22);
```

从：

```mysql
show databases;
show tables from test;
select * from test.tbl_test;
```

```tex
+------+-----+
| user | age |
+------+-----+
| li   |  22 |
+------+-----+
```