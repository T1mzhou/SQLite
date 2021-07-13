# SQLite解决插入大量数据速度慢的问题

## 背景

在重构公司项目中发现，公司以前代码中将大量数据插入SQLite数据库，速度非常慢，几千条居然需要将近1分钟；

## 解决方法

### 知识背景

事务（Transaction）是一个对数据库执行工作单元。事务（Transaction）是以逻辑顺序完成的工作单位或序列，可以是由用户手动操作完成，也可以是由某种数据库程序自动完成。事务（Transaction）是指一个或多个更改数据库的扩展。例如，如果您正在创建一个记录或者更新一个记录或者从表中删除一个记录，那么您正在该表上执行事务。重要的是要控制事务以确保数据的完整性和处理数据库错误。**实际上，您可以把许多的 SQLite 查询联合成一组，把所有这些放在一起作为事务的一部分进行执行**。

**SQLite中将DML操作的单条sql语句当成一次事务；这意味着：每插入一条数据，就有多少条数据需要去读写磁盘IO。**

## 解决方法

**开启事务，减少读写磁盘IO**

```c
bool is_success = true;
sqlite3_exec(conn, "begin;", 0, 0, 0);  // 开启事务
for (int i = 0; i < 5000; i++)
{

    // 拼接sql
    char* p_text = "T1mzhou"
	char* p_sql = sqlite3_mprintf("INSERT INTO name VALUES('%q') ", p_text);
    // 执行sql
    if (SQLITE_OK != sqlite3_exec(conn, p_sql, 0, 0, &err_msg))
    {
        is_success = false;
        break;
    }
}

if (is_success) 
{
    sqlite3_exec(conn, "commit;", 0, 0, 0);     // 成功，提交事务
}
else
{
   sqlite3_exec(conn, "rollback;", 0, 0, 0);    // 失败，回滚事务
}
```

## 结果

通过开启事务，项目中插入数据的速度大约提升了百倍。