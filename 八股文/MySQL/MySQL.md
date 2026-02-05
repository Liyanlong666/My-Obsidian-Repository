## MySQL 基础
### 0.什么是 MySQL？
MySQL 是一个开源的关系型数据库管理系统，现在隶属于 Oracle 旗下，也是我用得最多的一款关系型数据库。
与此同时，MySQL 也是我们国内使用频率最高的一种数据库，我在本地安装的是最新的 8.0 社区版。
#### 怎么删除/创建一张表和设定主键？
在 MySQL 中，我们可以使用 DROP TABLE 来删除表，使用 CREATE TABLE 来创建表并设定主键。创建表时，可以在定义列的同时设置某一列为主键，如将 id 列设为主键：PRIMARY KEY (id)。

```sql
CREATE TABLE users (
    id INT AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100),
    PRIMARY KEY (id)
);
```
#### 举例用sql实现升序降序
在 SQL 中，可以使用 ORDER BY 子句来对查询结果进行升序或降序排序。默认情况下，查询结果是升序排序，也可以通过 DESC 关键字进行降序排序。
例如，如果我想对 employees 表中的数据按工资升序排序，我会使用：

```sql
SELECT id, name, salary
FROM employees
ORDER BY salary ASC;
```
此外，如果我需要根据多个字段进行排序，例如先按工资降序排列，再按名字升序排列，我可以使用：

```sql
SELECT id, name, salary
FROM employees
ORDER BY salary DESC, name ASC;
```
#### MySQL性能慢的原因有哪些？
可能 SQL 使用了全表扫描，或者未正确使用索引，再或者查询语句过于复杂，如多表 JOIN 或嵌套子查询。
也可能是单表数据量过大，导致查询效率降低。
另外，可以增加一些缓存，如 Redis 来缓存热点数据，减少数据库的访问次数。
### 1. 什么是内连接、外连接、交叉连接、笛卡尔积呢？
MySQL 中的连接是通过两个或多个表之间的列进行关联，从而获取相关联的数据。连接分为内连接、外连接、交叉连接。
①、内连接（inner join）：返回两个表中连接字段匹配的行。如果一个表中的行在另一个表中没有匹配的行，则这些行不会出现在查询结果中。
假设有两个表，Employees 和 Departments。

```sql
SELECT Employees.Name, Departments.DeptName
FROM Employees
INNER JOIN Departments ON Employees.DeptID = Departments.DeptID;
```
这个查询将返回所有员工及其所在部门的信息，但仅限于那些在 Departments 表中有对应部门的员工。
②、外连接（outer join）：不仅返回两个表中匹配的行，还返回左表、右表或两者中未匹配的行。

```sql
SELECT Employees.Name, Departments.DeptName
FROM Employees
LEFT OUTER JOIN Departments ON Employees.DeptID = Departments.DeptID;
```
这个查询将返回所有员工的名字和他们部门的名字，即使某些员工没有分配到部门。
③、交叉连接（cross join）：返回第一个表中的每一行与第二个表中的每一行的组合，这种类型的连接通常用于生成笛卡尔积。

```sql
SELECT Employees.Name, Departments.DeptName
FROM Employees
CROSS JOIN Departments;
```
这个查询将为 Employees 表中的每个员工与 Departments 表中的每个部门生成一个组合。
④、笛卡尔积：数学中的一个概念，例如集合 A={a,b}，集合 B={0,1,2}，那么 A✖️B=`{<a,0>,<a,1>,<a,2>,<b,0>,<b,1>,<b,2>,}`。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的用友面试原题：两张表怎么进行连接
### 2. MySQL 的内连接、左连接、右连接有什么区别？
MySQL 的连接主要分为内连接和外连接，外连接又可以分为左连接和右连接。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-fcdaad5f-c50e-4834-9f9a-0b676cc6be83.jpg](./MySQL.assert/mysql-fcdaad5f-c50e-4834-9f9a-0b676cc6be83.jpg)

①、`inner join` 内连接，在两张表进行连接查询时，只保留两张表中完全匹配的结果集。
只有当两个表中都有匹配的记录时，这些记录才会出现在查询结果中。如果某一方没有匹配的记录，则该记录不会出现在结果集中。
内联可以用来找出两个表中共同的记录，相当于两个数据集的交集。
②、`left join` 返回左表（FROM 子句中指定的表）的所有记录，以及右表中匹配记录的记录。如果右表中没有匹配的记录，则结果中右表的部分会以 NULL 填充。
③、`right join` 刚好与左联相反，返回右表（FROM 子句中指定的表）的所有记录，以及左表中匹配记录的记录。如果左表中没有匹配的记录，则结果中左表的部分会以 NULL 填充。
拿我做过的[技术派实战项目](https://javabetter.cn/zhishixingqiu/paicoding.html)为例。
有三张表，一张文章表 article（主要存文章标题 title） 一张文章详情表 article_detail （主要存文章的内容 content），一张文章评论表 comment（主要存储评论 content） ，可以通过文章 id 关联。
先来看内联：

```sql
SELECT LEFT(a.title, 20) AS ArticleTitle, LEFT(c.content, 20) AS CommentContent
FROM article a
INNER JOIN comment c ON a.id = c.article_id
LIMIT 2;
```
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240308184454.png](./MySQL.assert/mysql-20240308184454.png)

这个查询返回了至少有一条评论的文章标题和评论内容的前 20 个字符，限制结果为前 2 条记录。
再来看左联：

```sql
SELECT LEFT(a.title, 20) AS ArticleTitle, LEFT(c.content, 20) AS CommentContent
FROM article a
LEFT JOIN comment c ON a.id = c.article_id
LIMIT 2;
```
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240308184901.png](./MySQL.assert/mysql-20240308184901.png)

这个查询返回所有文章的标题和文章评论的前 20 个字符，即使某些文章没有评论（这些情况下 CommentContent 为 NULL），限制结果为前 2 条记录。
最后来看右联：

```sql
SELECT LEFT(a.title, 20) AS ArticleTitle, LEFT(c.content, 20) AS CommentContent
FROM comment c
RIGHT JOIN article a ON a.id = c.article_id
LIMIT 2;
```
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240308185525.png](./MySQL.assert/mysql-20240308185525.png)

右联在这种情况下，其实比较别扭，因为可以直接使用左联来实现。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的腾讯 Java 后端实习一面原题：请说说 MySQL 的内联、左联、右联的区别。
### 3.说一下数据库的三大范式？
三大范式的作用是为了减少数据冗余，提高数据完整性。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-16e74a6b-a42a-464e-9b10-0252ee7ecc6e.jpg](./MySQL.assert/mysql-16e74a6b-a42a-464e-9b10-0252ee7ecc6e.jpg)

①、第一范式：确保表的每一列都是不可分割的基本数据单元，比如说用户地址，应该拆分成省、市、区、详细信息等 4 个字段。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240418093235.png](./MySQL.assert/mysql-20240418093235.png)

②、第二范式：要求表中的每一列都和主键直接相关，而不能只与主键的某一部分相关。
比如在一个订单表中，可能会存在订单编号和商品编号。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240418093351.png](./MySQL.assert/mysql-20240418093351.png)

这个订单表中就存在冗余数据，比如说商品名称、单位、商品价格等，应该将其拆分为订单表、订单商品关联表、商品表。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240418093726.png](./MySQL.assert/mysql-20240418093726.png)

③、第三范式：非主键列应该只依赖于主键列，不依赖于其他非主键列。
比如说在设计订单信息表的时候，可以把客户名称、所属公司、联系方式等信息拆分到客户信息表中，然后在订单信息表中用客户编号进行关联。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240418094332.png](./MySQL.assert/mysql-20240418094332.png)

#### 建表的时候考虑哪些问题？
在建表的时候，首先可以考虑表是否符合数据库范式，也就是确保字段不可再分，消除非主键依赖，确保字段仅依赖于主键。
然后在选择字段类型时，尽量选择合适的数据类型。
在字符集上，尽量选择 utf8mb4，这样不仅可以支持中文和英文，还可以支持表情符号等。
当数据量较大时（比如上千万行数据），需要考虑分表。比如订单表，可以采用水平分表的方式来分散存储压力。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动面经同学 13 Java 后端二面面试原题：什么是三大范式，为什么要有三大范式，什么场景下不用遵循三大范式，举一个场景
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的京东面经同学 5 Java 后端技术一面面试原题：建表考虑哪些问题
### 4.varchar 与 char 的区别？
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-40f42d59-a295-4543-8a03-43925da4d6d9.jpg](./MySQL.assert/mysql-40f42d59-a295-4543-8a03-43925da4d6d9.jpg)

**char**：

- char 表示定长字符串，长度是固定的；
- 如果插入数据的长度小于 char 的固定长度时，则用空格填充；
- 因为长度固定，所以存取速度要比 varchar 快很多，甚至能快 50%，但正因为其长度固定，所以会占据多余的空间，是空间换时间的做法；
- 对于 char 来说，最多能存放的字符个数为 255，和编码无关**varchar**：

- varchar 表示可变长字符串，长度是可变的；
- 插入的数据是多长，就按照多长来存储；
- varchar 在存取方面与 char 相反，它存取慢，因为长度不固定，但正因如此，不占据多余的空间，是时间换空间的做法；
- 对于 varchar 来说，最多能存放的字符个数为 65532日常的设计，对于长度相对固定的字符串，可以使用 char，对于长度不确定的，使用 varchar 更合适一些。
### 5.blob 和 text 有什么区别？

- blob 用于存储二进制数据，而 text 用于存储大字符串。
- blob 没有字符集，text 有一个字符集，并且根据字符集的校对规则对值进行排序和比较### 6.DATETIME 和 TIMESTAMP 的异同？
**相同点**：

1. 两个数据类型存储时间的表现格式一致。均为 `YYYY-MM-DD HH:MM:SS`
1. 两个数据类型都包含「日期」和「时间」部分。
1. 两个数据类型都可以存储微秒的小数秒（秒后 6 位小数秒）**区别**：
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-d94e5e1c-2614-4b8b-acdb-efb333032854.jpg](./MySQL.assert/mysql-d94e5e1c-2614-4b8b-acdb-efb333032854.jpg)

DATETIME 和 TIMESTAMP 的区别

1. **日期范围**：DATETIME 的日期范围是 `1000-01-01 00:00:00.000000` 到 `9999-12-31 23:59:59.999999`；TIMESTAMP 的时间范围是`1970-01-01 00:00:01.000000` UTC ` 到 ``2038-01-09 03:14:07.999999 ` UTC
1. **存储空间**：DATETIME 的存储空间为 8 字节；TIMESTAMP 的存储空间为 4 字节
1. **时区相关**：DATETIME 存储时间与时区无关；TIMESTAMP 存储时间与时区有关，显示的值也依赖于时区
1. **默认值**：DATETIME 的默认值为 null；TIMESTAMP 的字段默认不为空(not null)，默认值为当前时间(CURRENT_TIMESTAMP)### 7.in 和 exists 的区别？
MySQL 中的 in 语句是把外表和内表作 hash 连接，而 exists 语句是对外表作 loop 循环，每次 loop 循环再对内表进行查询。我们可能认为 exists 比 in 语句的效率要高，这种说法其实是不准确的，要区分情景：

1. 如果查询的两个表大小相当，那么用 in 和 exists 差别不大。
1. 如果两个表中一个较小，一个是大表，则子查询表大的用 exists，子查询表小的用 in。
1. not in 和 not exists：如果查询语句使用了 not in，那么内外表都进行全表扫描，没有用到索引；而 not extsts 的子查询依然能用到表上的索引。所以无论那个表大，用 not exists 都比 not in 要快。### 8.记录货币用什么字段类型比较好？
货币在数据库中 MySQL 常用 Decimal 和 Numeric 类型表示，这两种类型被 MySQL 实现为同样的类型。他们被用于保存与货币有关的数据。
例如 salary DECIMAL(9,2)，9(precision)代表将被用于存储值的总的小数位数，而 2(scale)代表将被用于存储小数点后的位数。存储在 salary 列中的值的范围是从-9999999.99 到 9999999.99。
DECIMAL 和 NUMERIC 值作为字符串存储，而不是作为二进制浮点数，以便保存那些值的小数精度。
之所以不使用 float 或者 double 的原因：因为 float 和 double 是以二进制存储的，所以有一定的误差。
### 9.怎么存储 emoji?
MySQL 的 utf8 字符集仅支持最多 3 个字节的 UTF-8 字符，但是 emoji 表情（😊）是 4 个字节的 UTF-8 字符，所以在 MySQL 中存储 emoji 表情时，需要使用 utf8mb4 字符集。

```sql
ALTER TABLE mytable CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```
MySQL 8.0 已经默认支持 utf8mb4 字符集，可以通过 `SHOW VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%';` 查看。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240418103116.png](./MySQL.assert/mysql-20240418103116.png)

> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动面经同学 13 Java 后端二面面试原题：mysql 怎么存 emoji，怎么编码
### 10.drop、delete 与 truncate 的区别？
三者都表示删除，但是三者有一些差别：
 | 区别 | delete | truncate | drop | 
|---|---|---|---|
 | 类型 | 属于 DML | 属于 DDL | 属于 DDL | 
 | 回滚 | 可回滚 | 不可回滚 | 不可回滚 | 
 | 删除内容 | 表结构还在，删除表的全部或者一部分数据行 | 表结构还在，删除表中的所有数据 | 从数据库中删除表，所有数据行，索引和权限也会被删除 | 
 | 删除速度 | 删除速度慢，需要逐行删除 | 删除速度快 | 删除速度最快 | 
> **因此，在不再需要一张表的时候，用 drop；在想删除部分数据行时候，用 delete；在保留表而删除所有数据的时候用 truncate。**

### 11.UNION 与 UNION ALL 的区别？

- 如果使用 UNION，会在表链接后筛选掉重复的记录行
- 如果使用 UNION ALL，不会合并重复的记录行
- 从效率上说，UNION ALL 要比 UNION 快很多，如果合并没有刻意要删除重复行，那么就使用 UNION All### 12.count(1)、count(*) 与 count(列名) 的区别？
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-2c754ee2-20c4-4c03-9db0-22c7c9eb7f01.jpg](./MySQL.assert/mysql-2c754ee2-20c4-4c03-9db0-22c7c9eb7f01.jpg)

**执行效果**：
> 
- count(*)包括了所有的列，相当于行数，在统计结果的时候，不会忽略列值为 NULL
- count(1)包括了忽略所有列，用 1 代表代码行，在统计结果的时候，不会忽略列值为 NULL
- count(列名)只包括列名那一列，在统计结果的时候，会忽略列值为空（这里的空不是只空字符串或者 0，而是表示 null）的计数，即某个字段值为 NULL 时，不统计。
**执行速度**：

- 列名为主键，count(列名)会比 count(1)快
- 列名不为主键，count(1)会比 count(列名)快
- 如果表多个列并且没有主键，则 count（1） 的执行效率优于 count（*）
- 如果有主键，则 select count（主键）的执行效率是最优的
- 如果表只有一个字段，则 select count（*）最优。### 13.SQL 查询语句的执行顺序了解吗？
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-47ddea92-cf8f-49c4-ab2e-69a829ff1be2.jpg](./MySQL.assert/mysql-47ddea92-cf8f-49c4-ab2e-69a829ff1be2.jpg)

> 
1. **FROM**：对 FROM 子句中的左表和右表执行笛卡儿积（Cartesianproduct），产生虚拟表 VT1
1. **ON**：对虚拟表 VT1 应用 ON 筛选，只有那些符合的行才被插入虚拟表 VT2 中
1. **JOIN**：如果指定了 OUTER JOIN（如 LEFT OUTER JOIN、RIGHT OUTER JOIN），那么保留表中未匹配的行作为外部行添加到虚拟表 VT2 中，产生虚拟表 VT3。如果 FROM 子句包含两个以上表，则对上一个连接生成的结果表 VT3 和下一个表重复执行步骤 1）～步骤 3），直到处理完所有的表为止
1. **WHERE**：对虚拟表 VT3 应用 WHERE 过滤条件，只有符合的记录才被插入虚拟表 VT4 中
1. **GROUP BY**：根据 GROUP BY 子句中的列，对 VT4 中的记录进行分组操作，产生 VT5
1. **​****CUBE|ROLLUP**：对表 VT5 进行 CUBE 或 ROLLUP 操作，产生表 VT6
2. **HAVING**：对虚拟表 VT6 应用 HAVING 过滤器，只有符合的记录才被插入虚拟表 VT7 中。
1. **SELECT**：第二次执行 SELECT 操作，选择指定的列，插入到虚拟表 VT8 中
1. **DISTINCT**：去除重复数据，产生虚拟表 VT9
1. **ORDER BY**：将虚拟表 VT9 中的记录按照进行排序操作，产生虚拟表 VT10。11）
1. **LIMIT**：取出指定行的记录，产生虚拟表 VT11，并返回给查询用户
### 14.介绍一下 MySQL 的常用命令（补充）
> 2024 年 03 月 13 日增补，可以先向面试官确认一下，“您提到的常用命令是指数据库表的增删改查 SQL 吗？”得到确认答复后可以根据下面这张思维导图作答：

![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240313093551.png](./MySQL.assert/mysql-20240313093551.png)

#### 说说数据库操作命令？
①、**创建数据库**:

```sql
CREATE DATABASE database_name;
```
②、**删除数据库**:

```sql
DROP DATABASE database_name;
```
③、**选择数据库**:

```sql
USE database_name;
```
#### 说说表操作命令？
①、**创建表**:

```sql
CREATE TABLE table_name (
    column1 datatype,
    column2 datatype,
    ...
);
```
②、**删除表**:

```sql
DROP TABLE table_name;
```
③、**显示所有表**:

```sql
SHOW TABLES;
```
④、**查看表结构**:

```sql
DESCRIBE table_name;
```
⑤、**修改表**（添加列）:

```sql
ALTER TABLE table_name ADD column_name datatype;
```
#### 说说 CRUD 命令？
①、**插入数据**:

```sql
INSERT INTO table_name (column1, column2, ...) VALUES (value1, value2, ...);
```
②、**查询数据**:

```sql
SELECT column_names FROM table_name WHERE condition;
```
③、**更新数据**:

```sql
UPDATE table_name SET column1 = value1, column2 = value2 WHERE condition;
```
④、**删除数据**:

```sql
DELETE FROM table_name WHERE condition;
```
#### 说说索引和约束的创建修改命令？
①、**创建索引**:

```sql
CREATE INDEX index_name ON table_name (column_name);
```
②、**添加主键约束**:

```sql
ALTER TABLE table_name ADD PRIMARY KEY (column_name);
```
③、**添加外键约束**:

```sql
ALTER TABLE table_name ADD CONSTRAINT fk_name FOREIGN KEY (column_name) REFERENCES parent_table (parent_column_name);
```
#### 说说用户和权限管理的命令？
①、**创建用户**:

```sql
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
```
②、**授予权限**:

```sql
GRANT ALL PRIVILEGES ON database_name.table_name TO 'username'@'host';
```
③、**撤销权限**:

```sql
REVOKE ALL PRIVILEGES ON database_name.table_name FROM 'username'@'host';
```
④、**删除用户**:

```sql
DROP USER 'username'@'host';
```
#### 说说事务控制的命令？
①、**开始事务**:

```sql
START TRANSACTION;
```
②、**提交事务**:

```sql
COMMIT;
```
③、**回滚事务**:

```sql
ROLLBACK;
```
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的用友金融一面原题：介绍一下 MySQL 的常用命令
### 15.MySQL bin 目录下的可执行文件了解吗（补充）
> 2024 年 03 月 13 日增补

推荐阅读：[MySQL bin 目录下的一些可执行文件](https://javabetter.cn/mysql/bin.html)

- mysql：客户端程序，用于连接 MySQL 服务器
- mysqldump：一个非常实用的 MySQL 数据库备份工具，用于创建一个或多个 MySQL 数据库级别的 SQL 转储文件，包括数据库的表结构和数据。对数据备份、迁移或恢复非常重要。
- mysqladmin：mysql 后面加上 admin 就表明这是一个 MySQL 的管理工具，它可以用来执行一些管理操作，比如说创建数据库、删除数据库、查看 MySQL 服务器的状态等。
- mysqlcheck：mysqlcheck 是 MySQL 提供的一个命令行工具，用于检查、修复、分析和优化数据库表，对数据库的维护和性能优化非常有用。
- mysqlimport：用于从文本文件中导入数据到数据库表中，非常适合用于批量导入数据。
- mysqlshow：用于显示 MySQL 数据库服务器中的数据库、表、列等信息。
- mysqlbinlog：用于查看 MySQL 二进制日志文件的内容，可以用于恢复数据、查看数据变更等。### 16.MySQL 第 3-10 条记录怎么查？（补充）
> 2024 年 03 月 30 日增补

在 MySQL 中，要查询第 3 到第 10 条记录，可以使用 limit 语句，结合偏移量 offset 和行数 row_count 来实现。
limit 语句用于限制查询结果的数量，偏移量表示从哪条记录开始，行数表示返回的记录数量。

```sql
SELECT * FROM table_name LIMIT 2, 8;
```

- 2：偏移量，表示跳过前两条记录，从第三条记录开始。
- 8：行数，表示从偏移量开始，返回 8 条记录。偏移量是从 0 开始的，即第一条记录的偏移量是 0；如果想从第 3 条记录开始，偏移量就应该是 2。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的美团面经同学 16 暑期实习一面面试原题：MySQL 第 3-10 条记录怎么查？
### 17.用过哪些 MySQL 函数？（补充）
> 2024 年 04 月 12 日增补

MySQL 支持很多内置函数，包括执行计算、格式转换、日期处理等。我说一些自己常用的（~~挑一些自己熟悉的~~）。
#### 用过哪些字符串函数来处理文本？

- `CONCAT()`: 连接两个或多个字符串。
- `LENGTH()`: 返回字符串的长度。
- `SUBSTRING()`: 从字符串中提取子字符串。
- `REPLACE()`: 替换字符串中的某部分。
- `LOWER()` 和 `UPPER()`: 分别将字符串转换为小写或大写。
- `TRIM()`: 去除字符串两侧的空格或其他指定字符。
```sql
-- 连接字符串
SELECT CONCAT('沉默', ' ', '王二') AS concatenated_string;

-- 获取字符串长度
SELECT LENGTH('沉默 王二') AS string_length;

-- 提取子字符串
SELECT SUBSTRING('沉默 王二', 1, 5) AS substring;

-- 替换字符串内容
SELECT REPLACE('沉默 王二', '王二', 'MySQL') AS replaced_string;

-- 字符串转小写
SELECT LOWER('HELLO WORLD') AS lower_case;

-- 字符串转大写
SELECT UPPER('hello world') AS upper_case;

-- 去除字符串两侧的空格
SELECT TRIM('  沉默 王二  ') AS trimmed_string;
```
#### 用过哪些数值函数？

- `ABS()`: 返回一个数的绝对值。
- `CEILING()`: 返回大于或等于给定数值的最小整数。
- `FLOOR()`: 返回小于或等于给定数值的最大整数。
- `ROUND()`: 四舍五入到指定的小数位数。
- `MOD()`: 返回除法操作的余数。
```sql
-- 返回绝对值
SELECT ABS(-123) AS absolute_value;

-- 向上取整
SELECT CEILING(123.45) AS ceiling_value;

-- 向下取整
SELECT FLOOR(123.45) AS floor_value;

-- 四舍五入
SELECT ROUND(123.4567, 2) AS rounded_value;

-- 余数
SELECT MOD(10, 3) AS modulus;
```
#### 用过哪些日期和时间函数？

- `NOW()`: 返回当前的日期和时间。
- `CURDATE()`: 返回当前的日期。
- `CURTIME()`: 返回当前的时间。
- `DATE_ADD()` 和 `DATE_SUB()`: 在日期上加上或减去指定的时间间隔。
- `DATEDIFF()`: 返回两个日期之间的天数。
- `DAY()`, `MONTH()`, `YEAR()`: 分别返回日期的日、月、年部分。
```sql
-- 返回当前日期和时间
SELECT NOW() AS current_date_time;

-- 返回当前日期
SELECT CURDATE() AS current_date;

-- 返回当前时间
SELECT CURTIME() AS current_time;

-- 在日期上添加天数
SELECT DATE_ADD(CURDATE(), INTERVAL 10 DAY) AS date_in_future;

-- 计算两个日期之间的天数
SELECT DATEDIFF('2024-12-31', '2024-01-01') AS days_difference;

-- 返回日期的年份
SELECT YEAR(CURDATE()) AS current_year;
```
#### 用过哪些汇总函数？

- `SUM()`: 计算数值列的总和。
- `AVG()`: 计算数值列的平均值。
- `COUNT()`: 计算某列的行数。
- `MAX()` 和 `MIN()`: 分别返回列中的最大值和最小值。
- `GROUP_CONCAT()`: 将多个行值连接为一个字符串。
```sql
-- 创建一个表并插入数据进行聚合查询
CREATE TABLE sales (
    product_id INT,
    sales_amount DECIMAL(10, 2)
);

INSERT INTO sales (product_id, sales_amount) VALUES (1, 100.00);
INSERT INTO sales (product_id, sales_amount) VALUES (1, 150.00);
INSERT INTO sales (product_id, sales_amount) VALUES (2, 200.00);

-- 计算总和
SELECT SUM(sales_amount) AS total_sales FROM sales;

-- 计算平均值
SELECT AVG(sales_amount) AS average_sales FROM sales;

-- 计算总行数
SELECT COUNT(*) AS total_entries FROM sales;

-- 最大值和最小值
SELECT MAX(sales_amount) AS max_sale, MIN(sales_amount) AS min_sale FROM sales;
```
#### 用过哪些逻辑函数？
> 
- `IF()`: 如果条件为真，则返回一个值；否则返回另一个值。
- `CASE`: 根据一系列条件返回值。
- `COALESCE()`: 返回参数列表中的第一个非 NULL 值。

```sql
-- IF函数
SELECT IF(1 > 0, 'True', 'False') AS simple_if;

-- CASE表达式
SELECT CASE WHEN 1 > 0 THEN 'True' ELSE 'False' END AS case_expression;

-- COALESCE函数
SELECT COALESCE(NULL, NULL, 'First Non-Null Value', 'Second Non-Null Value') AS first_non_null;
```
#### 用过哪些格式化函数？

- `FORMAT()`: 格式化数字为格式化的字符串，通常用于货币显示。
```sql
-- 格式化数字
SELECT FORMAT(1234567.8945, 2) AS formatted_number;
```
#### 用过哪些类型转换函数？

- `CAST()`: 将一个值转换为指定的数据类型。
- `CONVERT()`: 类似于`CAST()`，用于类型转换。
```sql
-- CAST函数
SELECT CAST('2024-01-01' AS DATE) AS casted_date;

-- CONVERT函数
SELECT CONVERT('123', SIGNED INTEGER) AS converted_number;
```
### 18.说说 SQL 的隐式数据类型转换？（补充）
> 2024 年 04 月 25 日增补

在 SQL 中，当不同数据类型的值进行运算或比较时，会发生隐式数据类型转换。
比如说，当一个整数和一个浮点数相加时，整数会被转换为浮点数，然后再进行相加。

```sql
SELECT 1 + 1.0; -- 结果为 2.0
```
比如说，当一个字符串和一个整数相加时，字符串会被转换为整数，然后再进行相加。

```sql
SELECT '1' + 1; -- 结果为 2
```
数据类型隐式转换会导致意想不到的结果，所以要尽量避免隐式转换。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240425111246.png](./MySQL.assert/mysql-20240425111246.png)

可以通过显式转换来规避这种情况。

```sql
SELECT CAST('1' AS SIGNED INTEGER) + 1; -- 结果为 2
```
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的小公司面经合集同学 1 Java 后端面试原题：说说 SQL 的隐式数据类型转换？
### 19. 说说 SQL 的语法树解析？（补充）
> 2024 年 09 月 19 日增补

语法树（或抽象语法树，AST）是 SQL 解析过程中的中间表示，它使用树形结构表示 SQL 语句的层次和逻辑。语法树由节点（Node）组成，每个节点表示 SQL 语句中的一个语法元素。

- **根节点**：通常是 SQL 语句的主要操作，例如 SELECT、INSERT、UPDATE、DELETE 等。
- **内部节点**：表示语句中的操作符、子查询、连接操作等。例如，WHERE 子句、JOIN 操作等。
- **叶子节点**：表示具体的标识符、常量、列名、表名等。例如，users 表、id 列、常量 1 等。以一个简单的 SQL 查询语句为例：

```sql
SELECT name, age FROM users WHERE age > 18;
```
这个查询语句的语法树可以表示为：

```sql
          SELECT
         /      \
     Columns     FROM
    /      \      |
  name      age  users
               |
             WHERE
               |
            age > 18
```
根节点：SELECT，表示这是一个查询操作。
子节点：Columns 和 FROM。

- Columns 子树表示查询的列，包含两个叶子节点：name 和 age。
- FROM 子树表示查询的数据源，包含一个叶子节点：users 表。条件节点：WHERE 子树表示查询条件，包含条件表达式 `age > 18`。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动面经同学 21  抖音商城一面面试原题：sql的语法树解析
## 数据库架构
### 20.说说 MySQL 的基础架构？
MySQL 的架构大致可以分为三层，从上到下依次是：连接层、服务层、和存储引擎层。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-77626fdb-d2b0-4256-a483-d1c60e68d8ec.jpg](./MySQL.assert/mysql-77626fdb-d2b0-4256-a483-d1c60e68d8ec.jpg)

①、连接层主要负责客户端连接的管理，包括验证用户身份、权限校验、连接管理等。可以通过数据库连接池来提升连接的处理效率。
②、服务层是 MySQL 的核心，主要负责查询解析、优化、执行等操作。在这一层，SQL 语句会经过解析、优化器优化，然后转发到存储引擎执行，并返回结果。这一层包含查询解析器、优化器、执行计划生成器、缓存（如查询缓存）、日志模块等。
③、存储引擎层负责数据的实际存储和提取，是 MySQL 架构中与数据交互最直接的层。MySQL 支持多种存储引擎，如 InnoDB、MyISAM、Memory 等。
#### binlog写入在哪一层？
binlog 在服务层，负责记录 SQL 语句的变化。它记录了所有对数据库进行更改的操作，用于数据恢复、主从复制等。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动面经同学 21  抖音商城一面面试原题：mysql分为几层？binlog写入在哪一层
### 21.一条查询语句如何执行？
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240415102041.png](./MySQL.assert/mysql-20240415102041.png)

> 第一步，客户端发送 SQL 查询语句到 MySQL 服务器。
第二步，MySQL 服务器的连接器开始处理这个请求，跟客户端建立连接、获取权限、管理连接。
~~第三步（MySQL 8.0 以后已经干掉了），连接建立后，MySQL 服务器的查询缓存组件会检查是否有缓存的查询结果。如果有，直接返回给客户端；如果没有，进入下一步。~~
第三步，解析器对 SQL 语句进行解析，检查语句是否符合 SQL 语法规则，确保引用的数据库、表和列都是存在的，并处理 SQL 语句中的名称解析和权限验证。
第四步，优化器负责确定 SQL 语句的执行计划，这包括选择使用哪些索引，以及决定表之间的连接顺序等。
第五步，执行器会调用存储引擎的 API 来进行数据的读写。
第六步，MySQL 的存储引擎是插件式的，不同的存储引擎在细节上面有很大不同。例如，InnoDB 是支持事务的，而 MyISAM 是不支持的。之后，会将执行结果返回给客户端
第七步，客户端接收到查询结果，完成这次查询请求。

### 22.一条更新语句怎么执行的？
更新语句的执行是 Server 层和引擎层配合完成，数据除了要写入表中，还要记录相应的日志。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-812fb038-39de-4204-ac9f-93d8b7448a18.jpg](./MySQL.assert/mysql-812fb038-39de-4204-ac9f-93d8b7448a18.jpg)


1. 执行器先找引擎获取 ID=2 这一行。ID 是主键，存储引擎检索数据，找到这一行。如果 ID=2 这一行所在的数据页本来就在内存中，就直接返回给执行器；否则，需要先从磁盘读入内存，然后再返回。
1. 执行器拿到引擎给的行数据，把这个值加上 1，比如原来是 N，现在就是 N+1，得到新的一行数据，再调用引擎接口写入这行新数据。> 
1. 引擎将这行新数据更新到内存中，同时将这个更新操作记录到 redo log 里面，此时 redo log 处于 prepare 状态。然后告知执行器执行完成了，随时可以提交事务。
1. 执行器生成这个操作的 binlog，并把 binlog 写入磁盘。
1. 执行器调用引擎的提交事务接口，引擎把刚刚写入的 redo log 改成提交（commit）状态，更新完成。
从上图可以看出，MySQL 在执行更新语句的时候，在服务层进行语句的解析和执行，在引擎层进行数据的提取和存储；同时在服务层对 binlog 进行写入，在 InnoDB 内进行 redo log 的写入。
不仅如此，在对 redo log 写入时有两个阶段的提交，一是 binlog 写入之前`prepare`状态的写入，二是 binlog 写入之后`commit`状态的写入。
### 23.说说 MySQL 的数据存储形式（补充）
> 2024 年 04 月 26 日增补

MySQL 是以表的形式存储数据的，而表空间的结构则由段、区、页、行组成。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240515110034.png](./MySQL.assert/mysql-20240515110034.png)

①、段（Segment）：表空间由多个段组成，常见的段有数据段、索引段、回滚段等。
创建索引时会创建两个段，数据段和索引段，数据段用来存储叶子阶段中的数据；索引段用来存储非叶子节点的数据。
回滚段包含了事务执行过程中用于数据回滚的旧数据。
②、区（Extent）：段由一个或多个区组成，区是一组连续的页，通常包含 64 个连续的页，也就是 1M 的数据。
使用区而非单独的页进行数据分配可以优化磁盘操作，减少磁盘寻道时间，特别是在大量数据进行读写时。
③、页（Page）：页是 InnoDB 存储数据的基本单元，标准大小为 16 KB，索引树上的一个节点就是一个页。
也就意味着数据库每次读写都是以 16 KB 为单位的，一次最少从磁盘中读取 16KB 的数据到内存，一次最少写入 16KB 的数据到磁盘。
④、行（Row）：InnoDB 采用行存储方式，意味着数据按照行进行组织和管理，行数据可能有多个格式，比如说 COMPACT、REDUNDANT、DYNAMIC 等。
MySQL 8.0 默认的行格式是 DYNAMIC，由COMPACT 演变而来，意味着这些数据如果超过了页内联存储的限制，则会被存储在溢出页中。
可以通过 `show table status like '%article%'` 查看行格式。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240515123301.png](./MySQL.assert/mysql-20240515123301.png)

## 存储引擎
### 24.MySQL 有哪些常见存储引擎？
MySQL 支持多种存储引擎，常见的有 MyISAM、InnoDB、MEMORY 等。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240408073338.png](./MySQL.assert/mysql-20240408073338.png)

我来做一个表格对比：
 | 功能 | InnoDB | MyISAM | MEMORY | 
|---|---|---|---|
 | 支持事务 | Yes | No | No | 
 | 支持全文索引 | Yes | Yes | No | 
 | 支持 B+树索引 | Yes | Yes | Yes | 
 | 支持哈希索引 | Yes | No | Yes | 
 | 支持外键 | Yes | No | No | 
除此之外，我还了解到：
①、MySQL 5.5 之前，默认存储引擎是 MyISAM，5.5 之后是 InnoDB。
②、InnoDB 支持的哈希索引是自适应的，不能人为干预。
③、InnoDB 从 MySQL 5.6 开始，支持全文索引。
④、InnoDB 的最小表空间略小于 10M，最大表空间取决于页面大小（page size）。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240408074630.png](./MySQL.assert/mysql-20240408074630.png)

#### 如何切换 MySQL 的数据引擎？
可以通过 alter table 语句来切换 MySQL 的数据引擎。

```sql
ALTER TABLE your_table_name ENGINE=InnoDB;
```
不过不建议，应该提前设计好到底用哪一种存储引擎。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动面经同学 1 Java 后端技术一面面试原题：MySQL 支持哪些存储引擎?
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的用友面试原题：innodb 引擎和 hash 引擎有什么区别
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的国企零碎面经同学 9 面试原题：MySQL 的存储引擎
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的京东同学 4 云实习面试原题：mysql的数据引擎有哪些, 区别(innodb,MyISAM,Memory)
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的阿里系面经同学 19 饿了么面试原题：存储引擎介绍
### 25.那存储引擎应该怎么选择？
> 
- 大多数情况下，使用默认的 InnoDB 就对了，InnoDB 可以提供事务、行级锁、外键、B+ 树索引等能力。
- MyISAM 适合读更多的场景。
- MEMORY 适合临时表，数据量不大的情况。由于数据都存放在内存，所以速度非常快。
### 26.InnoDB 和 MyISAM 主要有什么区别？
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-b7aa040e-a3a7-4133-8c43-baccc3c8d012.jpg](./MySQL.assert/mysql-b7aa040e-a3a7-4133-8c43-baccc3c8d012.jpg)

InnoDB 和 MyISAM 之间的区别主要表现在存储结构、事务支持、最小锁粒度、索引类型、主键必需、表的具体行数、外键支持等方面。
**①、存储结构**：

- MyISAM：用三种格式的文件来存储，.frm 文件存储表的定义；.MYD 存储数据；.MYI 存储索引。
- InnoDB：用两种格式的文件来存储，.frm 文件存储表的定义；.ibd 存储数据和索引。**②、事务支持**：

- MyISAM：不支持事务。
- InnoDB：支持事务。**③、最小锁粒度**：

- MyISAM：表级锁，高并发中写操作存在性能瓶颈。
- InnoDB：行级锁，并发写入性能高。**④、索引类型**：
MyISAM 为非聚簇索引，索引和数据分开存储，索引保存的是数据文件的指针。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240403130104.png](./MySQL.assert/mysql-20240403130104.png)

InnoDB 为聚簇索引，索引和数据不分开。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240403130508.png](./MySQL.assert/mysql-20240403130508.png)

**⑤、外键支持**：MyISAM 不支持外键；InnoDB 支持外键。
**⑥、主键必需**：MyISAM 表可以没有主键；InnoDB 表必须有主键。
**⑦、表的具体行数**：MyISAM 表的具体行数存储在表的属性中，查询时直接返回；InnoDB 表的具体行数需要扫描整个表才能返回。
### 27. InnoDB 的 Buffer Pool了解吗？（补充）
> Buffer Pool 是 InnoDB 存储引擎中的一个内存缓冲区，它会将数据以页（page）的单位保存在内存中，当查询请求需要读取数据时，优先从 Buffer Pool 获取数据，避免直接访问磁盘。

![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20241104200915.png](./MySQL.assert/mysql-20241104200915.png)

也就是说，即便我们只访问了一行数据的一个字段，InnoDB 也会将整个数据页加载到 Buffer Pool 中，以便后续的查询。
修改数据时，也会先在缓存页面中修改。当数据页被修改后，会在 Buffer Pool 中变为脏页。
脏页不会立刻写回到磁盘。
InnoDB 会定期将这些脏页刷新到磁盘，保证数据的一致性。通常采用改良的 LRU 算法来管理缓存页，也就是将最近最少使用的数据移出缓存，为新数据腾出空间。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20241104202752.png](./MySQL.assert/mysql-20241104202752.png)

Buffer Pool 能够显著减少对磁盘的访问，从而提升数据库的读写性能。
> 在调优方面，我们可以设置合理的 Buffer Pool 大小（通常为物理内存的 70%），并配置多个 Buffer Pool 实例（通过 innodb_buffer_pool_instances）来提升并发能力。此外，还可以通过调整刷新策略参数，比如 innodb_flush_log_at_trx_commit，来平衡性能和数据持久性。

## 高可用/性能
### 65.数据库读写分离了解吗？
读写分离的基本原理是将数据库读写操作分散到不同的节点上，下面是基本架构图：
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-31df767c-db05-4de4-a05b-a45bcf76c1bf.jpg](./MySQL.assert/mysql-31df767c-db05-4de4-a05b-a45bcf76c1bf.jpg)

读写分离的基本实现是:

- 数据库服务器搭建主从集群，一主一从、一主多从都可以。
- 数据库主机负责读写操作，从机只负责读操作。
- 数据库主机通过复制将数据同步到从机，每台数据库服务器都存储了所有的业务数据。
- 业务服务器将写操作发给数据库主机，将读操作发给数据库从机。### 66.读写分离的分配怎么实现呢？
将读写操作区分开来，然后访问不同的数据库服务器，一般有两种方式：程序代码封装和中间件封装。

1. 程序代码封装程序代码封装指在代码中抽象一个数据访问层（所以有的文章也称这种方式为 "中间层封装" ） ，实现读写操作分离和数据库服务器连接的管理。例如，基于 Hibernate 进行简单封装，就可以实现读写分离：
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-771eb01f-3f1a-4437-8e1b-affe4de36ec3.jpg](./MySQL.assert/mysql-771eb01f-3f1a-4437-8e1b-affe4de36ec3.jpg)

目前开源的实现方案中，淘宝的 TDDL （Taobao Distributed Data Layer, 外号：头都大了）是比较有名的。

1. 中间件封装中间件封装指的是独立一套系统出来，实现读写操作分离和数据库服务器连接的管理。中间件对业务服务器提供 SQL 兼容的协议，业务服务器无须自己进行读写分离。
对于业务服务器来说，访问中间件和访问数据库没有区别，事实上在业务服务器看来，中间件就是一个数据库服务器。
其基本架构是：
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-f2313613-25bd-4065-8f63-969a4b5757a7.jpg](./MySQL.assert/mysql-f2313613-25bd-4065-8f63-969a4b5757a7.jpg)

### 67.主从复制原理了解吗？
MySQL 的主从复制（Master-Slave Replication）是一种数据同步机制，用于将数据从一个主数据库（master）复制到一个或多个从数据库（slave）。
广泛用于数据备份、灾难恢复和数据分析等场景。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-1bfbfcb5-2392-4f98-be1b-a66204da09e5.jpg](./MySQL.assert/mysql-1bfbfcb5-2392-4f98-be1b-a66204da09e5.jpg)

复制过程的主要步骤有：

- 在主服务器上，所有修改数据的语句（如 INSERT、UPDATE、DELETE）会被记录到二进制日志中。
- 主服务器上的一个线程（二进制日志转储线程）负责读取二进制日志的内容并发送给从服务器。
- 从服务器接收到二进制日志数据后，会将这些数据写入自己的中继日志（Relay Log）。中继日志是从服务器上的一个本地存储。
- 从服务器上有一个 SQL 线程会读取中继日志，并在本地数据库上执行，从而将更改应用到从数据库中，完成同步。> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的腾讯云智面经同学 16 一面面试原题：MySQL 的主从复制过程
### 68.主从同步延迟怎么处理？
**主从同步延迟的原因**
一个服务器开放Ｎ个链接给客户端来连接的，这样有会有大并发的更新操作, 但是从服务器的里面读取 binlog 的线程仅有一个，当某个 SQL 在从服务器上执行的时间稍长 或者由于某个 SQL 要进行锁表就会导致，主服务器的 SQL 大量积压，未被同步到从服务器里。这就导致了主从不一致， 也就是主从延迟。
**主从同步延迟的解决办法**
解决主从复制延迟有几种常见的方法:

1. 写操作后的读操作指定发给数据库主服务器例如，注册账号完成后，登录时读取账号的读操作也发给数据库主服务器。这种方式和业务强绑定，对业务的侵入和影响较大，如果哪个新来的程序员不知道这样写代码，就会导致一个 bug。

1. 读从机失败后再读一次主机这就是通常所说的 "二次读取" ，二次读取和业务无绑定，只需要对底层数据库访问的 API 进行封装即可，实现代价较小，不足之处在于如果有很多二次读取，将大大增加主机的读操作压力。例如，黑客暴力破解账号，会导致大量的二次读取操作，主机可能顶不住读操作的压力从而崩溃。

1. 关键业务读写操作全部指向主机，非关键业务采用读写分离例如，对于一个用户管理系统来说，注册 + 登录的业务读写操作全部访问主机，用户的介绍、爰好、等级等业务，可以采用读写分离，因为即使用户改了自己的自我介绍，在查询时却看到了自我介绍还是旧的，业务影响与不能登录相比就小很多，还可以忍受。
### 69.你们一般是怎么分库的呢？
分库分表是为了解决单库单表数据量过大导致数据库性能下降的一种解决方案。
分库的策略有两种：
①、垂直分库：按照业务模块将不同的表拆分到不同的库中，例如，用户表、订单表、商品表等分到不同的库中。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-2a43af18-617b-4502-b66a-894c2ff4c6c3.jpg](./MySQL.assert/mysql-2a43af18-617b-4502-b66a-894c2ff4c6c3.jpg)

②、水平分库：按照一定的策略将一个表中的数据拆分到多个库中，例如，按照用户 id 的 hash 值将用户表拆分到不同的库中。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-debe0fb1-d7f7-4ef2-8c99-13c9377138b6.jpg](./MySQL.assert/mysql-debe0fb1-d7f7-4ef2-8c99-13c9377138b6.jpg)

> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的快手面经同学 7 Java 后端技术一面面试原题：分库分表了解吗
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的华为面经同学 8 技术二面面试原题：说说分库分表的准则
### 70.那你们是怎么分表的？
在[技术派实战项目](https://javabetter.cn/zhishixingqiu/paicoding.html)中，我们将文章的基本信息和文章详情做了垂直分表处理，因为文章的详情会占用比较大的空间，并且更新频繁，而文章的基本信息占用的空间比较小，且更新频率较低。
垂直拆分可以减轻单表的查询和更新压力。
当单表数据增量过快，比如说单表超过 500 万条数据，就可以考虑水平分表了。比如说我们可以将文章表拆分成多个表，如 article_0、article_9999、article_19999 等。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-7cba6ce0-c8bb-4f51-9c3b-e5a44e724c79.jpg](./MySQL.assert/mysql-7cba6ce0-c8bb-4f51-9c3b-e5a44e724c79.jpg)


> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的快手面经同学 7 Java 后端技术一面面试原题：分库分表了解吗
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的华为面经同学 8 技术二面面试原题：说说分库分表的准则
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的腾讯面经同学 29 Java 后端一面原题：表存满了之后怎么扩表？
### 71.分片策略有哪几种？
常见的分片策略有三种，分别是范围分片、Hash 分片和配置路由分片。
范围分片是根据某个字段的值范围进行分库分表。这种方式适用于分片键具有顺序性或连续性的场景。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-b3882ca3-1d04-44e2-9015-7e6c867255a0.jpg](./MySQL.assert/mysql-b3882ca3-1d04-44e2-9015-7e6c867255a0.jpg)

比如说将 user_id 作为分片键：

- 1 ~ 10000 → db1.user_1
- 10001 ~ 20000 → db2.user_2Hash 分片是指通过对分片键的值进行哈希运算，将数据均匀分布到多个分片中。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-e01e7757-c337-48c8-95db-2f7cfd2bc036.jpg](./MySQL.assert/mysql-e01e7757-c337-48c8-95db-2f7cfd2bc036.jpg)

假如我们一开始规划好了 4 个数据表，那么路由算法可以简单地通过取模来实现：

```sql
public String getTableNameByHash(long userId) {
    int tableIndex = (int) (userId % 4);
    return "user_" + tableIndex;
}
```
配置路由分片是通过路由配置来确定数据应该存储在哪个表，适用于分片键不规律的场景。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-fcd34332-d38d-455a-875d-d4afd37cac72.jpg](./MySQL.assert/mysql-fcd34332-d38d-455a-875d-d4afd37cac72.jpg)

比如说我们可以通过 order_router 表来确定订单数据存储在哪个表中：
 | order_id | table_id | 
|---|---|
 | xxxx | table_1 | 
 | yyyy | table_2 | 
 | zzzz | table_3 | 
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的腾讯面经同学 24 面试原题：项目中的水平分表是怎么做的？分片键具体是怎么设置的？
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的腾讯面经同学 29 Java 后端一面原题：分库分表具体的分片策略是怎么做的？
### 72.不停机扩容怎么实现？
实际上，不停机扩容，实操起来是个非常麻烦而且很有风险的操作，当然，面试回答起来就简单很多。

- **第一阶段：在线双写，查询走老库**
1. 建立好新的库表结构，数据写入久库的同时，也写入拆分的新库
1. 数据迁移，使用数据迁移程序，将旧库中的历史数据迁移到新库
1. 使用定时任务，新旧库的数据对比，把差异补齐![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-2d4d94c9-e816-47fc-93dd-a835b1318099.jpg](./MySQL.assert/mysql-2d4d94c9-e816-47fc-93dd-a835b1318099.jpg)


- **第二阶段：在线双写，查询走新库**
1. 完成了历史数据的同步和校验
1. 把对数据的读切换到新库![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-5cf01486-72c1-4eab-9f6e-a19c31569f46.jpg](./MySQL.assert/mysql-5cf01486-72c1-4eab-9f6e-a19c31569f46.jpg)


- **第三阶段：旧库下线**
1. 旧库不再写入新的数据
1. 经过一段时间，确定旧库没有请求之后，就可以下线老库![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-a122d6d5-fff2-4ccd-8ddb-a9282eb2e2da.jpg](./MySQL.assert/mysql-a122d6d5-fff2-4ccd-8ddb-a9282eb2e2da.jpg)

### 73.怎么分库分表？
如果表的字段过多，我会按字段的访问频率或功能相关性拆分成多个表，减少单表宽度。
如果单表数据量过大，我会按 ID 或时间将数据分到多张表中。比如订单表可以按 `user_id % N` 进行水平分表。
如果业务模块较多，我会将不同模块的表分布到不同的数据库中，比如用户相关的表放在一个库，订单相关的表放在另一个库。
如果单库数据量过大，我会按用户 ID 范围或哈希值将数据分布到多个库中。
#### 常用的分库分表中间件有哪些？
常用的分库分表中间件有 Sharding-JDBC 和 Mycat。
①、Sharding-JDBC 最初由当当开源，后来贡献给了 Apache，主要在 Java 的 JDBC 层提供额外的服务。无需额外部署和依赖，可理解为增强版的 JDBC 驱动，完全兼容 JDBC 和各种 ORM 框架。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20241207120214.png](./MySQL.assert/mysql-20241207120214.png)

②、Mycat 是由阿里巴巴的一款产品 Cobar 衍生而来，可以把它看作一个数据库代理，其核心功能是分表分库，即将一个大表切片为多个小表，一个大库切片成多个小库。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20241207121845.png](./MySQL.assert/mysql-20241207121845.png)

推荐阅读：[mycat 介绍](https://yanxizhu.com/index.php/archives/113/)
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的得物面经同学 9 面试题目原题：Mysql有很大的数据量怎么办？怎么分表分库？
### 74.你觉得分库分表会带来什么问题呢？
从分库的角度来讲：

- **事务的问题**使用关系型数据库，有很大一点在于它保证事务完整性。
而分库之后单机事务就用不上了，必须使用分布式事务来解决。

- **跨库 JOIN 问题**在一个库中的时候我们还可以利用 JOIN 来连表查询，而跨库了之后就无法使用 JOIN 了。
此时的解决方案就是**在业务代码中进行关联**，也就是先把一个表的数据查出来，然后通过得到的结果再去查另一张表，然后利用代码来关联得到最终的结果。
这种方式实现起来稍微比较复杂，不过也是可以接受的。
还有可以**适当的冗余一些字段**。比如以前的表就存储一个关联 ID，但是业务时常要求返回对应的 Name 或者其他字段。这时候就可以把这些字段冗余到当前表中，来去除需要关联的操作。
还有一种方式就是**数据异构**，通过 binlog 同步等方式，把需要跨库 join 的数据异构到 ES 等存储结构中，通过 ES 进行查询。
从分表的角度来看：

- **跨节点的 count,order by,group by 以及聚合函数问题**只能由业务代码来实现或者用中间件将各表中的数据汇总、排序、分页然后返回。

- **数据迁移，容量规划，扩容等问题**数据的迁移，容量如何规划，未来是否可能再次需要扩容，等等，都是需要考虑的问题。

- **ID 问题**数据库表被切分后，不能再依赖数据库自身的主键生成机制，所以需要一些手段来保证全局主键唯一。

1. 还是自增，只不过自增步长设置一下。比如现在有三张表，步长设置为 3，三张表 ID 初始值分别是 1、2、3。这样第一张表的 ID 增长是 1、4、7。第二张表是 2、5、8。第三张表是 3、6、9，这样就不会重复了。
1. UUID，这种最简单，但是不连续的主键插入会导致严重的页分裂，性能比较差。
1. 分布式 ID，比较出名的就是 Twitter 开源的 sonwflake 雪花算法#### id 是怎么生成的？
[技术派](https://javabetter.cn/zhishixingqiu/paicoding.html)项目中，我们在雪花算法 Snowflake 的基础上实现了一套自定义的 ID 生成方案，通过更改时间戳单位、ID 长度、workId 与 dataCenterId 的分配比例，ID 生成的延迟降低了 20%；同时满足了社区在高并发环境下 ID 的唯一性和可追溯性。
![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20241223150915.png](./MySQL.assert/mysql-20241223150915.png)

#### 雪花算法具体是怎么实现的？
雪花算法是 Twitter 开源的分布式 ID 生成算法，其核心思想是：使用一个 64 位的数字来作为全局唯一 ID。

- 第 1 位是符号位，永远是 0，表示正数。
- 接下来的 41 位是时间戳，记录的是当前时间戳减去一个固定的开始时间戳，可以使用 69 年。
- 然后是 10 位的工作机器 ID。
- 最后是 12 位的序列号，每毫秒最多可生成 4096 个 ID。![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20241223150351.png](./MySQL.assert/mysql-20241223150351.png)

大致的实现代码如下所示：

```sql
public class SnowflakeIdGenerator {
    private long datacenterId = 1L; // 数据中心ID
    private long machineId = 1L; // 机器ID
    private long sequence = 0L; // 序列号

    private long lastTimestamp = -1L;

    public synchronized long nextId() {
        long timestamp = System.currentTimeMillis();
        if (timestamp == lastTimestamp) {
            sequence = (sequence + 1) & 4095;
            if (sequence == 0) {
                while (timestamp == lastTimestamp) {
                    timestamp = System.currentTimeMillis();
                }
            }
        } else {
            sequence = 0;
        }

        lastTimestamp = timestamp;

        return ((timestamp - 1609459200000L) << 22) | (datacenterId << 17) | (machineId << 12) | sequence;
    }
}
```
除了雪花算法，还有百度 UidGenerator、美团 Leaf 等开源的分布式 ID 生成方案。
> 
1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的腾讯面经同学 29 Java 后端一面原题：id是怎么生成的？（分布式自增主键）
 运维75.百万级别以上的数据如何删除？关于索引：由于索引需要额外的维护成本，因为索引文件是单独存在的文件,所以当我们对数据的增加,修改,删除,都会产生额外的对索引文件的操作,这些操作需要消耗额外的 IO,会降低增/改/删的执行效率。 所以，在我们删除数据库百万级别数据的时候，查询 MySQL 官方手册得知删除数据的速度和创建的索引数量是成正比的。 1. 所以我们想要删除百万数据的时候可以先删除索引 2. 然后删除其中无用数据 3. 删除完成后重新创建索引创建索引也非常快 76.百万千万级大表如何添加字段？当线上的数据库数据量到达几百万、上千万的时候，加一个字段就没那么简单，因为可能会长时间锁表。 大表添加字段，通常有这些做法： - 通过中间表转换过去 创建一个临时的新表，把旧表的结构完全复制过去，添加字段，再把旧表数据复制过去，删除旧表，新表命名为旧表的名称，这种方式可能回丢掉一些数据。 - 用 pt-online-schema-change `pt-online-schema-change`是 percona 公司开发的一个工具，它可以在线修改表结构，它的原理也是通过中间表。 - 先在从库添加 再进行主从切换 如果一张表数据量大且是热表（读写特别频繁），则可以考虑先在从库添加，再进行主从切换，切换后再将其他几个节点上添加字段。 77.MySQL cpu 飙升的话，要怎么处理呢？排查过程： （1）使用 top 命令观察，确定是 mysqld 导致还是其他原因。 （2）如果是 mysqld 导致的，show processlist，查看 session 情况，确定是不是有消耗资源的 sql 在运行。 （3）找出消耗高的 sql，看看执行计划是否准确， 索引是否缺失，数据量是否太大。 处理： （1）kill 掉这些线程 (同时观察 cpu 使用率是否下降)， （2）进行相应的调整 (比如说加索引、改 sql、改内存参数) （3）重新跑这些 SQL。 其他情况： 也有可能是每个 sql 消耗资源并不多，但是突然之间，有大量的 session 连进来导致 cpu 飙升，这种情况就需要跟应用一起来分析为何连接数会激增，再做出相应的调整，比如说限制连接数等 SQL 题78.一张表：id，name，age，sex，class，sql 语句：所有年龄为 18 的人的名字？找到每个班年龄大于 18 有多少人？找到每个班年龄排前两名的人？（补充）这是一道 SQL 题，主要考察 SQL 的基本语法。建议大家直接在本地建表，然后实操一下。 2024 年 04 月 11 日增补。第一步，建表： CREATE TABLE students ( id INT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(50), age INT, sex CHAR(1), class VARCHAR(50) ); 第二步，插入数据： INSERT INTO students (name, age, sex, class) VALUES ('沉默王二', 18, '女', '三年二班'), ('沉默王一', 20, '男', '三年二班'), ('沉默王三', 19, '男', '三年三班'), ('沉默王四', 17, '男', '三年三班'), ('沉默王五', 20, '女', '三年四班'), ('沉默王六', 21, '男', '三年四班'), ('沉默王七', 18, '女', '三年四班'); ①、所有年龄为 18 的人的名字 SELECT name FROM students WHERE age = 18; 这条 SQL 语句从表中选择`age`等于 18 的所有记录，并返回这些记录的`name`字段。 ![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240410105325.png](./MySQL.assert/mysql-20240410105325.png)
 ②、找到每个班年龄大于 18 有多少人 SELECT class, COUNT(*) AS number_of_students FROM students WHERE age > 18 GROUP BY class; 这条 SQL 语句先筛选出年龄大于 18 的记录，然后按`class`分组，并计算每个班的学生数。 ![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240410105512.png](./MySQL.assert/mysql-20240410105512.png)
 ③、找到每个班年龄排前两名的人 这个查询稍微复杂一些，需要使用子查询和`COUNT`函数。 SELECT a.class, a.name, a.age FROM students a WHERE ( SELECT COUNT(DISTINCT b.age) FROM students b WHERE b.class = a.class AND b.age > a.age ) < 2 ORDER BY a.class, a.age DESC; 这条 SQL 语句首先从`students`表中选择`class`、`name`和`age`字段，然后使用子查询计算每个班级中年龄排前两名的学生。 ![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240410105951.png](./MySQL.assert/mysql-20240410105951.png)
 1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的奇安信面经同学 1 Java 技术一面面试原题：一张表：id，name，age，sex，class，sql 语句：所有年龄为 18 的人的名字？找到每个班年龄大于 18 有多少人？找到每个班年龄排前两名的人？ 79.有一个查询需求，MySQL 中有两个表，一个表 1000W 数据，另一个表只有几千数据，要做一个关联查询，如何优化如果 orders 表是大表（比如 1000 万条记录），而 users 表是相对较小的表（比如几千条记录）。 **①、为关联字段建立索引**，确保两个表中用于 JOIN 操作的字段都有索引。这是最基本的优化策略，避免数据库进行全表扫描，可以大幅度减少查找匹配行的时间。 CREATE INDEX idx_user_id ON users(user_id); CREATE INDEX idx_user_id ON orders(user_id); **②、小表驱动大表**，在执行 JOIN 操作时，先过滤小表中的数据，这样可以减少后续与大表进行 JOIN 时需要处理的数据量，从而提高查询效率。 SELECT u.*, o.* FROM ( SELECT user_id FROM users WHERE some_condition -- 这里是对小表进行过滤的条件 ) AS filtered_users JOIN orders o ON filtered_users.user_id = o.user_id WHERE o.some_order_condition; -- 如果需要，可以进一步过滤大表 1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的京东面经同学 1 Java 技术一面面试原题：有一个查询需求，MySQL 中有两个表，一个表 1000W 数据，另一个表只有几千数据，要做一个关联查询，如何优化 80.新建一个表结构，创建索引，将百万或千万级的数据使用 insert 导入该表，新建一个表结构，将百万或千万级的数据使用 isnert 导入该表，再创建索引，这两种效率哪个高呢？或者说用时短呢？talk is cheap，show me the code。 先创建一个表，然后创建索引，然后执行插入语句，来看看执行时间（100 万数据在我本机上执行时间比较长，我们就用 10 万条数据来测试）。 CREATE TABLE test_table ( id BIGINT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255) NOT NULL, email VARCHAR(255) NOT NULL, created_at DATETIME NOT NULL ); CREATE INDEX idx_name ON test_table(name); DELIMITER // CREATE PROCEDURE insert_data() BEGIN DECLARE i INT DEFAULT 0; WHILE i < 1000000 DO INSERT INTO test_table(name, email, created_at) VALUES (CONCAT('wanger',i), CONCAT('email', i, '@example.com'), NOW()); SET i = i + 1; END WHILE; END // DELIMITER ; CALL insert_data(); 这是一个完整的测试过程，通过存储过程来执行插入操作，然后查看总的执行时间。 在实际的开发工作中，可能涉及到持久层框架，还有批量插入。 ![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240412083019.png](./MySQL.assert/mysql-20240412083019.png)
 总的时间 13.93+0.01+0.01+0.01=13.96 秒。 接下来，我们再创建一个表，然后执行插入操作，最后再创建索引。 CREATE TABLE test_table_no_index ( id BIGINT AUTO_INCREMENT PRIMARY KEY, name VARCHAR(255) NOT NULL, email VARCHAR(255) NOT NULL, created_at DATETIME NOT NULL ); DELIMITER // CREATE PROCEDURE insert_data_no_index() BEGIN DECLARE i INT DEFAULT 0; WHILE i < 1000000 DO INSERT INTO test_table_no_index(name, email, created_at) VALUES (CONCAT('wanger', i), CONCAT('email', i, '@example.com'), NOW()); SET i = i + 1; END WHILE; END // DELIMITER ; CALL insert_data_no_index(); CREATE INDEX idx_name_no_index ON test_table_no_index(name); 来看一下总的时间，0.01+0.00+13.08+0.18=13.27 秒。 ![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240412083312.png](./MySQL.assert/mysql-20240412083312.png)
 先插入数据再创建索引的方式（13.27 秒）比先创建索引再插入数据（13.96 秒）要快一些。这个结果虽然显示时间差异不是非常大，但它确实反映了数据库处理大量数据插入时的性能特点。 - **先插入数据再创建索引**：在没有索引的情况下插入数据，数据库不需要在每次插入时更新索引，这会减少插入操作的开销。之后一次性创建索引通常比逐条记录更新索引更快。 - **先创建索引再插入数据**：这种情况下，数据库需要在每次插入新记录时维护索引结构，随着数据量的增加，索引的维护可能会导致额外的性能开销。 数据库是先建立索引还是先插入数据？在 InnoDB 中，如果表定义了主键，那么主键索引就是聚簇索引。如果没有明确指定主键，InnoDB 会自动选择一个唯一索引作为聚簇索引。如果表没有任何唯一索引，InnoDB 将自动生成一个隐藏的行 ID 作为聚簇索引。 这意味着当插入新数据时，InnoDB 首先将数据插入到聚簇索引中。这一步骤实质上是创建索引的一部分，因为数据存放在索引结构中。 对于非主键的其他索引（次级索引），在插入数据到聚簇索引后，InnoDB 还需要更新表的所有次级索引。这些索引中的每一个都包含指向聚簇索引记录的指针。 所以在 InnoDB 中，数据插入和索引创建（更新）是密不可分的。从数据库的视角看，插入操作包括向聚簇索引添加记录和更新所有相关的次级索引。这些步骤在一个事务中原子地执行，以确保数据的一致性和完整性。 1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的农业银行面经同学 7 Java 后端面试原题：数据库是先建立索引还是先插入数据 81.什么是深分页，select * from tbn limit 1000000000 这个有什么问题，如果表大或者表小分别什么问题深分页指的是在分页查询中，请求的页数非常大，例如请求第 100 万页的数据。在这种情况下，数据库需要跳过前 999999 页的数据，会消耗大量的 CPU 和 I/O 资源，导致查询性能下降。 `select * from tbn limit 1000000000` 正是一个深分页查询。 - 如果表的数据量非常大，那么这个查询可能会消耗大量的内存和 CPU 资源，甚至可能导致数据库崩溃。 - 如果表的数据量非常小，比如说只有 100 条，那就会返回这前 100 条，虽然没什么性能影响，但这个查询本身没什么意义。 1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动面经同学 13 Java 后端二面面试原题：什么是深分页，select \* from tbn limit 1000000000 这个有什么问题，如果表大或者表小分别什么问题 82.一个表（name, sex,age,id），select age,id,name from tblname where name='paicoding';怎么建索引索引的建立应当基于查询中的过滤条件（WHERE 子句）以及查询的选择列（SELECT 子句）。 由于查询条件是`name='paicoding'`，所以应当为`name`字段建立索引。 CREATE INDEX idx_name ON tblname(name); 查询中选择了`age`、`id`和`name`字段，如果这三列经常一起使用，可以考虑建立包含这些字段的联合索引。可以将查询条件中的字段放在联合索引的首位，这样查询时可以利用索引覆盖，直接从索引中获取数据，而不需要再去查找数据行。 CREATE INDEX idx_name_age_id ON tblname (name, age, id); 表字段id（主键）age name select name,age from 表 where name like(A%) and age =30会不会走索引？可以创建组合索引 (name, age)，这可以利用 name 和 age 的双重条件来高效地进行查询。 1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的字节跳动面经同学 13 Java 后端二面面试原题：一个表（name, sex,age,id），select age,id,name from tblname where name='paicoding';怎么建索引 83.SQL 题：一个学生成绩表，字段有学生姓名、班级、成绩，求各班前十名这是一个典型的 SQL 题，主要考察 SQL 的基本语法和分组查询。 第一步，建表： CREATE TABLE student_scores ( student_name VARCHAR(100), class VARCHAR(50), score INT ); 第二步，插入数据： INSERT INTO student_scores (student_name, class, score) VALUES ('沉默王二', '三年二班', 88), ('沉默王三', '三年二班', 92), ('沉默王四', '三年二班', 87), ('沉默王五', '三年二班', 85), ('沉默王六', '三年二班', 90), ('沉默王七', '三年二班', 95), ('沉默王八', '三年二班', 82), ('沉默王九', '三年二班', 78), ('沉默王十', '三年二班', 91), ('沉默王十一', '三年二班', 79), ('沉默王十二', '三年三班', 84), ('沉默王十三', '三年三班', 81), ('沉默王十四', '三年三班', 90), ('沉默王十五', '三年三班', 88), ('沉默王十六', '三年三班', 87), ('沉默王十七', '三年三班', 93), ('沉默王十八', '三年三班', 89), ('沉默王十九', '三年三班', 85), ('沉默王二十', '三年三班', 92), ('沉默王二十一', '三年三班', 84); 第三步，查询各班前十名： SET @cur_class = NULL, @cur_rank = 0; SELECT student_name, class, score FROM ( SELECT student_name, class, score, @cur_rank := IF(@cur_class = class, @cur_rank + 1, 1) AS rank, @cur_class := class FROM student_scores ORDER BY class, score DESC ) AS ranked WHERE ranked.rank <= 10; 使用 `@cur_class` 和 `@cur_rank` 来跟踪当前行的班级和排名。 在 SELECT 语句中，通过检查当前班级（`@cur_class`）是否与上一行相同来决定排名。如果相同，则增加排名；如果不同，则重置排名为 1。 然后通过 ORDER BY 子句确保在计算排名前按班级和分数排序。 ![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/mysql-20240423113508.png](./MySQL.assert/mysql-20240423113508.png)
 1. [Java 面试指南（付费）](https://javabetter.cn/zhishixingqiu/mianshi.html)收录的腾讯云智面经同学 16 一面面试原题：SQL 题：一个学生成绩表，字段有学生姓名、班级、成绩，求各班前十名 --- 图文详解 84 道 MySQL 面试高频题，这次吊打面试官，我觉得稳了（手动 dog）。整理：沉默王二，戳[转载链接](https://mp.weixin.qq.com/s/JFjFs_7xduCmHOegbJ-Gbg)，作者：三分恶，戳[原文链接](https://mp.weixin.qq.com/s/zSTyZ-8CFalwAYSB0PN6wA)。 *没有什么使我停留——除了目的，纵然岸旁有玫瑰、有绿荫、有宁静的港湾，我是不系之舟*。 **系列内容**： - [面渣逆袭 Java SE 篇 👍](https://javabetter.cn/sidebar/sanfene/javase.html) - [面渣逆袭 Java 集合框架篇 👍](https://javabetter.cn/sidebar/sanfene/javathread.html) - [面渣逆袭 Java 并发编程篇 👍](https://javabetter.cn/sidebar/sanfene/collection.html) - [面渣逆袭 JVM 篇 👍](https://javabetter.cn/sidebar/sanfene/jvm.html) - [面渣逆袭 Spring 篇 👍](https://javabetter.cn/sidebar/sanfene/spring.html) - [面渣逆袭 Redis 篇 👍](https://javabetter.cn/sidebar/sanfene/redis.html) - [面渣逆袭 MyBatis 篇 👍](https://javabetter.cn/sidebar/sanfene/mybatis.html) - [面渣逆袭 MySQL 篇 👍](https://javabetter.cn/sidebar/sanfene/mysql.html) - [面渣逆袭操作系统篇 👍](https://javabetter.cn/sidebar/sanfene/os.html) - [面渣逆袭计算机网络篇 👍](https://javabetter.cn/sidebar/sanfene/network.html) - [面渣逆袭 RocketMQ 篇 👍](https://javabetter.cn/sidebar/sanfene/rocketmq.html) - [面渣逆袭分布式篇 👍](https://javabetter.cn/sidebar/sanfene/fenbushi.html) - [面渣逆袭微服务篇 👍](https://javabetter.cn/sidebar/sanfene/weifuwu.html) - [面渣逆袭设计模式篇 👍](https://javabetter.cn/sidebar/sanfene/shejimoshi.html) --- GitHub 上标星 10000+ 的开源知识库《[二哥的 Java 进阶之路](https://github.com/itwanger/toBeBetterJavaer)》第一版 PDF 终于来了！包括 Java 基础语法、数组&字符串、OOP、集合框架、Java IO、异常处理、Java 新特性、网络编程、NIO、并发编程、JVM 等等，共计 32 万余字，500+张手绘图，可以说是通俗易懂、风趣幽默……详情戳：[太赞了，GitHub 上标星 10000+ 的 Java 教程](https://javabetter.cn/overview/) 微信搜 **沉默王二** 或扫描下方二维码关注二哥的原创公众号沉默王二，回复 **222** 即可免费领取。 ![C:\Users\Administrator\Desktop\YuqueExportToMarkdown\八股文\MySQL/MySQL.assert/gongzhonghao.png](./MySQL.assert/gongzhonghao.png)

