---
title: MySQL日志
date: 2024-06-19
description: 总结一下MySQL中的各种日志，包括错误日志、慢查询日志、二进制日志等
categories: MySQL
tag: 
    - 慢查询
mathjax: true
---
# MySQL日志

MySQL 日志包括多种类型，用于记录数据库服务器的各种活动和事件。以下是 MySQL 中常见的几种日志类型：

1. **错误日志（Error Log）**：
错误日志记录了 MySQL 服务器在运行过程中发生的各种错误和警告信息。这些错误可能涉及到数据库启动、运行时错误、崩溃等。错误日志对于排查数据库问题非常重要。
2. **查询日志（Query Log）**：
查询日志记录了 MySQL 服务器收到的所有查询请求，包括 SELECT、INSERT、UPDATE、DELETE 等操作。启用查询日志可能会对性能产生一定的影响，因为每个查询都要被记录下来。
3. **慢查询日志（Slow Query Log）**：
慢查询日志记录了执行时间超过阈值的查询语句。管理员可以设置一个执行时间阈值，通常以秒为单位，超过该阈值的查询会被记录到慢查询日志中。慢查询日志对于优化数据库性能非常有帮助。
4. **二进制日志（Binary Log）**：
二进制日志记录了对数据库执行的所有更改操作，如插入、更新、删除等。二进制日志通常用于数据备份、复制和恢复。它包含了所有更改的精确信息，可以确保在数据库恢复时数据的一致性。
5. **事务日志（Transaction Log）**：
事务日志记录了每个事务的开始和提交（或回滚）等操作。它用于确保数据库的 ACID 特性，即原子性、一致性、隔离性和持久性。

这些日志对于数据库的管理、监控和故障排除都至关重要。管理员可以根据具体需求来启用或禁用这些日志，并根据日志内容来优化数据库性能和确保数据的安全性。

## 慢查询日志

### 1. 查看当前慢查询日志文件路径和设置

通过 MySQL 的配置文件或者直接在 MySQL 命令行中执行以下命令，可以查看慢查询日志的当前设置：

```sql
SHOW VARIABLES LIKE 'slow_query_log';
SHOW VARIABLES LIKE 'slow_query_log_file';
```

- `slow_query_log`：显示是否启用了慢查询日志。如果结果为 `ON`，表示启用了慢查询日志；如果为 `OFF`，表示未启用。
    
    ![Untitled](/pic/MySQL%E6%97%A5%E5%BF%97%20e91bf43e5e27486dac1228b8fbaa1e05/Untitled.png)
    
- `slow_query_log_file`：显示当前慢查询日志文件的路径和名称。
    
    ![Untitled](/pic/MySQL%E6%97%A5%E5%BF%97%20e91bf43e5e27486dac1228b8fbaa1e05/Untitled%201.png)
    

如果是windows系统可能会出现没有文件路径的情况，慢查询日志的地址其实在`C:\ProgramData\MySQL\MySQL Server 8.0\Data\xxxxx-slow.log`

![Untitled](/pic/MySQL%E6%97%A5%E5%BF%97%20e91bf43e5e27486dac1228b8fbaa1e05/Untitled%202.png)

### 2. 查看慢查询阈值设置

可以查看慢查询的阈值设置:

```sql
SHOW VARIABLES LIKE 'long_query_time';
SET  long_query_time=1; //通过SQL修改 
```

- `long_query_time`：显示了在秒为单位的阈值。任何运行时间超过该值的 SQL 查询将被记录到慢查询日志中。默认值通常是 10 秒。
    
    ![Untitled](/pic/MySQL%E6%97%A5%E5%BF%97%20e91bf43e5e27486dac1228b8fbaa1e05/Untitled%203.png)
    

### 3. 修改慢查询日志和参数设置

如果需要修改慢查询日志或者阈值设置，可以在 MySQL 的配置文件中（通常是 `my.cnf` 或 `my.ini`）找到对应的配置项，并进行修改。修改后需要重启 MySQL 服务才能生效。

**可以按照以下步骤操作,打开 MySQL 的慢查询日志：**

1. 打开 MySQL 配置文件 `my.cnf`（或者 `my.ini`，具体名称根据操作系统和安装方式可能有所不同）。
2. 在文件中找到 `[mysqld]` 部分。
3. 添加或修改以下行，以启用慢查询日志并设置相关参数
4. 保存并关闭配置文件。
5. 重启 MySQL 服务，使新的配置生效。
6. 当慢查询发生时，MySQL 将会将相应的查询记录到指定的日志文件中。

```
slow_query_log = ON
slow_query_log_file = /var/log/mysql/mysql-slow.log
long_query_time = 5
```

这会启用慢查询日志，设置慢查询日志文件的路径为 `/var/log/mysql/mysql-slow.log`，并将慢查询的阈值设置为 5 秒。

你也可以根据需要自定义`long_query_time`参数的值，以适应你的应用程序或系统的性能要求。如果你希望更快地捕获慢查询，可以将`long_query_time`设置为一个较小的值，比如1秒或2秒。请记住，较小的值可能会导致更多的查询被记录，对数据库性能产生一定的影响。因此，需要根据具体情况进行平衡和调整。

在启用慢查询日志后，可能会对 MySQL 的性能产生一定的影响。因此，应根据实际情况来调整 `long_query_time` 参数的值，避免过度记录无关紧要的查询。

### 日志参数

这里我模拟了时间过长的SQL，看一下我们的slow.log（请忽略抽象的SQL，这里完全是为了凑查询时间）

```sql
# Time: 2024-06-20T05:22:47.404762Z
# User@Host: savage[savage] @  [61.162.39.62]  Id:  6793
# Query_time: 2.067045  Lock_time: 0.000013 Rows_sent: 10000  Rows_examined: 232036
SET timestamp=1718860965;
SELECT password FROM user_info ^M
WHERE PASSWORD IN ( SELECT password FROM user_info ^M
WHERE PASSWORD IN ( SELECT password FROM user_info ^M
WHERE PASSWORD IN ( SELECT password FROM user_info ^M
WHERE PASSWORD IN ( SELECT password FROM user_info WHERE username LIKE "高%" ) AND ENABLE =1) AND ENABLE =1 ) AND ENABLE =1) AND ENABLE =1^M
UNION^M
SELECT password FROM user_info ^M
WHERE PASSWORD IN ( SELECT password FROM user_info ^M
WHERE PASSWORD IN ( SELECT password FROM user_info ^M
WHERE PASSWORD IN ( SELECT password FROM user_info ^M
WHERE PASSWORD IN ( SELECT password FROM user_info WHERE username LIKE "高%" ) AND ENABLE =1) AND ENABLE =1 ) AND ENABLE =1) AND ENABLE =1;
```

上面是一条慢SQL的日志，里面有一些参数和具体查询慢的SQL语句，参数的含义如下：

- `Time`：指查询执行完成的时间戳。
- `User@Host`：显示执行查询的用户和主机信息。
- `Id`：连接标识符，表示执行查询的连接 ID。
- `Query_time`：指查询执行所花费的时间，以秒为单位。
- `Lock_time`：指在查询执行过程中等待锁定的时间，以秒为单位。
- `Rows_sent`：指从服务器发送给客户端的行数。
- `Rows_examined`：指在执行查询过程中扫描（或检查）的行数。

## 二进制日志

MySQL的二进制日志（Binary Log）是一种用于记录数据库中对数据进行修改的日志。它包含了对数据库执行的所有更改操作，如插入、更新和删除等。二进制日志以二进制格式记录，可以确保记录的精确性和可靠性。

二进制日志有以下几个主要的作用：

1. **数据恢复**：二进制日志可以用于数据库的点播恢复和增量恢复。通过回放二进制日志中的操作，可以将数据库还原到特定时间点或特定事务的状态，从而实现数据的恢复。
2. **数据库复制**：二进制日志用于数据库的主从复制。主库将自己的二进制日志发送给从库，从库通过解析并执行这些日志，保持与主库的数据同步。
3. **数据审计**：二进制日志可以用于审计数据库中发生的所有更改操作。通过分析二进制日志，可以了解到哪些用户进行了何种操作，以及何时进行的。
4. **数据备份**：二进制日志可用于增量备份。通过备份完整数据后，再备份二进制日志，可以在恢复时将增量的修改应用到已备份的数据上，从而减少数据备份的时间和空间消耗。

MySQL的二进制日志是数据库管理的重要组成部分，它提供了数据恢复、复制和审计的能力，并且在高可用性和灾难恢复方面发挥着关键的作用。

### **开启binlog**

要在MySQL中启用二进制日志（binlog），你需要进行以下步骤：

1. **编辑MySQL配置文件**：
打开MySQL的配置文件（通常是`my.cnf`或`my.ini`），并找到 `[mysqld]` 部分。
2. **启用二进制日志**：
在 `[mysqld]` 部分中添加或修改以下行，以启用二进制日志：
    
    ```
    log_bin = /path/to/binlog_file
    ```
    
    其中 `/path/to/binlog_file` 是你想要存储binlog文件的路径和文件名。例如，可以设置为 `/var/lib/mysql/mysql-bin.log`。
    
3. **设置binlog格式**：
可选的，你可以设置二进制日志的格式。MySQL支持三种格式：STATEMENT、ROW和MIXED。可以使用以下行在 `[mysqld]` 部分中设置binlog格式：
    
    ```
    binlog_format = [format]
    ```
    
    其中 `[format]` 可以是 `STATEMENT`、`ROW` 或 `MIXED`。
    
4. **重启MySQL服务**：
保存对配置文件的更改，并重启MySQL服务，以使更改生效。你可以使用适合你的操作系统的命令来重启MySQL服务。

一旦完成上述步骤，MySQL就会启用二进制日志功能，并将binlog文件写入指定的位置。你可以使用先前提到的方法来查看和管理binlog文件，启用二进制日志可能会对数据库性能产生一些影响，因为MySQL需要额外的资源来记录和维护binlog。因此，在启用binlog之前，请确保你的系统具备足够的资源来处理这些额外负载。

**查看或者导出binlog**

要查看MySQL的二进制日志（binlog），可以使用MySQL提供的一些工具和命令。以下是一些查看binlog的方法：

1. **mysqlbinlog命令**：
`mysqlbinlog` 是一个用于解析和显示MySQL二进制日志内容的命令行工具。可以使用以下命令来查看binlog文件的内容：
    
    ```
    mysqlbinlog /path/to/binlog_file
    ```
    
    这将输出binlog文件的内容到控制台，你可以查看其中的SQL语句和相关信息。
    
2. **mysqlbinlog工具的选项**：
`mysqlbinlog` 提供了一些选项，可以用来过滤和格式化输出。例如，你可以使用 `-start-datetime` 和 `-stop-datetime` 选项指定起始和结束时间，只查看特定时间范围内的binlog内容。
3. **MySQL客户端**：
在MySQL客户端中，你也可以使用 `SHOW BINLOG EVENTS` 命令来查看当前正在使用的binlog文件的内容。这个命令会列出binlog中的事件，包括每个事件的时间戳、类型和SQL语句。
4. **MySQL Workbench**：
如果你使用MySQL Workbench作为MySQL数据库管理工具，它也提供了查看binlog的功能。在Workbench中，你可以通过导航到“Server” -> “Data Export” -> “Manage Import/Export” -> “Export to Disk” -> “Export Options” -> “Include Binary Log”来选择导出binlog。

通过以上方法，可以查看MySQL的二进制日志，了解其中记录的数据库更改操作和相关信息。

## 事务日志

MySQL的Transaction Log（事务日志）是一种用于记录数据库中所有修改操作的日志。它包括两个主要组件：redo log（重做日志）和undo log（撤销日志）。

1. **Redo Log（重做日志）**：
Redo log用于恢复事务性更改，以确保事务的持久性。当将数据修改写入到磁盘之前，MySQL会首先将这些修改记录到redo log中。Redo log是在事务提交之前持久化到磁盘的，因此即使在发生系统崩溃或断电的情况下，MySQL也可以使用redo log来重新应用未写入磁盘的事务修改，从而恢复数据库到崩溃之前的状态。
2. **Undo Log（撤销日志）**：
Undo log用于支持事务的回滚和MVCC（多版本并发控制）。在事务执行期间，MySQL会将旧版本的数据记录到undo log中，以便在事务回滚或其他事务需要读取旧版本数据时进行使用。通过undo log，MySQL可以撤销事务的修改，还原到事务开始之前的状态。

事务日志的存在有助于确保数据库的一致性和可靠性。它提供了一种持久化的方式来记录事务修改，并支持数据库的恢复和并发控制机制。通过事务日志，MySQL可以在系统故障或意外中断的情况下保持数据的完整性，并提供事务的持久性。

### **查看**事务**日志**

在 MySQL 中，查看事务日志可以使用以下两种方式：

1. **使用 MySQL 内置命令行工具：**
MySQL 提供了 `mysqlbinlog` 工具，可以用于查看二进制日志中的内容，包括事务日志。以下是使用该工具查看事务日志的示例命令：
    
    ```
    mysqlbinlog /var/lib/mysql/mysql-bin.000001
    ```
    
    这条命令将显示指定二进制日志文件（这里是 `mysql-bin.000001`）中的所有内容，包括所有修改数据的 SQL 语句和其他事务日志相关信息。可以根据需要使用选项来过滤输出结果。
    
2. **使用 MySQL Workbench：**
MySQL Workbench 是一个图形化界面工具，提供了一个方便的方式来查看事务日志。以下是使用该工具查看事务日志的步骤：
    1. 打开 MySQL Workbench 并连接到目标数据库。
    2. 在左侧导航栏中选择“Server Status”。
    3. 在“Server Status”选项卡上，选择“Binary Log”子选项卡。
    4. 在该选项卡上，将显示所有可用的二进制日志文件。可以选择要查看的日志文件并单击“Browse”按钮以查看其内容。

以上两种方法都可以用于查看事务日志。使用 `mysqlbinlog` 命令可以快速查看日志内容，而使用 MySQL Workbench 则提供了更直观方便的图形化界面。