mysqldump 

####  导出表结构

```sh
mysqldump -hHost -PPort -uUser -pPassword -d Database TableName
```

#### 导出表结构和数据

```sh
# 直接导出，User 必须有 Table Lock 权限
mysqldump -hHost -PPort -uUser -pPassword --default-character-set=utf8 Database TableName

# mysqldump: Got error: 1044: Access denied for user 'User'@'%' to database 'Database' when doing LOCK TABLES
# User 没有 Table Lock 权限, 当执行mysqldump命令时，是一次性锁定当前库的所有表。而不是锁定当前导出表
mysqldump -hHost -PPort -uUser -pPassword --default-character-set=utf8 --skip-lock-tables Database TableName

# mysqldump: Couldn't execute 'SELECT COLUMN_NAME,                       JSON_EXTRACT(HISTOGRAM, '$."number-of-buckets-specified"')                FROM information_schema.COLUMN_STATISTICS                WHERE SCHEMA_NAME = 'Database' AND TABLE_NAME = 'TableName';': Unknown table 'COLUMN_STATISTICS' in information_schema (1109)
# 禁用新标志
mysqldump -hHost -PPort -uUser -pPassword --default-character-set=utf8 --skip-lock-tables --column-statistics=0 Database TableName
```

#### 表上锁

LOCK TABLES为当前线程锁定表。

如果一个线程获得在一个表上的一个READ锁，该线程和所有其他线程只能从表中读。 如果一个线程获得一个表上的一个WRITE锁，那么只有持锁的线程READ或WRITE表，其他线程被阻止。

```mysql
lock table TableName [READ|WRITE];
```

#### 解锁表

 UNLOCK TABLES释放被当前线程持有的任何锁,当线程发出==另外一个LOCK TABLES==时，或当==服务器的连接被关闭==时，当前线程锁定的所有表会==自动被解锁==。 

```mysql
UNLOCK TABLES;
```

#### 查看上锁表

```mysql
show open table from Database where In_use > 0;
```





