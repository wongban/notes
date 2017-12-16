# Oracle Database 11g SQL开发指南
`SQL(Structred Query Language)结构化查询语言`
`DBMS(Database Management System)数据库管理系统`

## 简介
SQL语句分为5类：

* **查询语句** 用于检索数据库表中存储的行。使用SELECT语句编写查询语句
* **数据操纵语言（Data  Manipulation Language，DML）** 修改表内容
    - **INSERT** 向表中添加行
    - **UPDATE** 修改行的内容
    - **DELETE** 删除行
* **数据定义语言（Data Definition Language，DDL）** 定义构成数据库的数据结构
    - **CREATE** 创建数据库结构。例如，CREATE TABLE创建一个表；CREATE USER创建一个数据库用户。
    - **ALTER** 修改数据库结构。例如，ALTER TABLE修改一个表。
    - **DROP** 删除数据库结构。例如，DROP TABLE删除一个表。
    - **RENAME** 更改表名。
    - **TRUNCATE** 删除表的全部内容。
* **事务控制（Transaction Control，TC）** 用于将对行所做的修改永久性地存储到表中，或者取消这些修改操作。TC语句有3种：
    - **COMMIT** 永久性地保存对行所做的修改。
    - **ROLLBACK** 取消对行所做的修改。
    - **SAVEPOINT** 设置一个“保存点”，可以将对行所做的修改回滚到此处。
* **数据控制语言（Data Control Language，DCL）** 用于修改数据库结构的操作权限。DCL语句有两种：
    - **GRANT** 授予其他用户对数据库结构的访问权限。
    - **REVOKE** 阻止其他用户访问数据库结构。

从命令行启动**SQL Plus**
```
语法：
sqlplus [user_name[/password][@ host_string]]
例子：
sqlplus scott/tiger
sqlplus scott/tiger@orcl
sqlplus scott@orcl
sqlplus hr/hr@10.17.1.98/orcl
```
**dual**表是只包含一行的一个表。当需要数据库来计算一个表达式（例如，2*15/5）时，或者想要得到当前的日期时，dual表是非常有用的。
通过输入**EDIT**命令，可以编辑**SQLPlus**中的最后一条SQL语句。在输错SQL语句或想修改SQL语句时，这种功能非常有用。在Windows系统中输入EDIT命令后，会启动记事本；然后就可以编辑SQL语句。在退出记事本并保存SQL语句时，SQL语句就会被传递到**SQLPlus**中，在这里通过输入一个斜杠（/）可以重新执行该SQL语句。

SQLPlus中运行sql脚本：
```sql
--语法：
directory \store_schema.sql
--例子：
@ E:\sql_book\SQL\store_schema.sql
@ "E:\Oracle SQL book\sql_book\SQL\store_schema.sql"
```
