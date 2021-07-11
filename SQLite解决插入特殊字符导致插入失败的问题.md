# SQLite解决插入特殊字符导致插入失败的问题

## 背景

在使用SQLite数据库时，我们经常会遇到插入的数据里面有单引号之类的特殊字符，如果不能正确处理，会导致插入数据失败。

## 解决方法

### 方法一

#### 	对特殊字符进行转义

```sqlite
INSERT INTO time VALUES('5 O''clock');
# 插入的数据是
5 O'clock
```

####    缺点

* 需要每次添加转义字符，单条或数据量比较小的时候比较方便；
* 比如处理json文本数据时,数据量比较大需要每次去遍历数据，然后转义，不方便也比较麻烦；

### 方法二

#### 	使用SQLite内置的格式化字符串

```c
char* p_text = "T1mzhou"
char* p_sql = sqlite3_mprintf("INSERT INTO name VALUES('%q') ", p_text);
sqlite3_exece(db, p_sql, 0, 0, 0);
sqlite3_free(p_sql);
```

