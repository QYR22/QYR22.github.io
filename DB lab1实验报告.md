```toc
```
覃翊茹 10215101524

# 1. Ubuntu20.04下MySQL的安装
电脑已有Ubuntu20.04环境，在[PPT给出的安装教程](https://www.yuque.com/lidx/databaseexp/exp1) 指导下完成安装。同时安装Navicat，与虚拟机中的MySQL连接。安装过程比较顺利。
![[Pasted image 20230314091818.png#center|安装结果]]

# 2. 创建University数据库

>[!NOTE]
>DDL: Data Definition Language，表结构层的对表操作  create、drop、alter等等
> DML: Data Manipulation Language，表中数据层的，对表中数据增删改   insert、update、delete等等
> DQL: Data Query Language，表中数据查询 select等等

## 2.1 DDL--操作数据库
1. **创建**数据库db1 
    1. 直接创建 `create databse db1;`
    2. 创建并检查是否存在 `create databse if not exists db1;`
```bash
# 2. 修改字符集，默认是utf8 改成utf8mb4
mysql> alter database db1 default character set utf8mb4;
Query OK, 1 row affected (0.01 sec)
# 3. 查看xxx数据库的定义信息
mysql> show create database db1;
+----------+---------------------------------------------------------------+
| Database | create Database                                                                                                               
+----------+---------------------------------------------------------------+
| db1      | CREATE DATABASE `db1` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci */ /*!80016 DEFAULT ENCRYPTION='N' */ |
+----------+---------------------------------------------------------------+
1 row in set (0.00 sec)

# 4. 查看数据库-->显示了自己创建的db1数据库
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| db1                |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

# 5. 使用数据库db1
mysql> use db1
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
# 6. 查看正在使用数据库-->正在使用db1数据库
mysql> select database();
+------------+
| database() |
+------------+
| db1        |
+------------+
1 row in set (0.00 sec)

# 7. 删除数据库
mysql> drop database db1;
Query OK, 1 row affected (0.06 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```

## 2.2 DDL--操作表
首先创建好University数据库，`create database University;`
使用这个数据库 `use University;` ，再进行下面的操作。
### 数据类型
本次实验直接/间接接触到的数据类型(常用的)。

|整数类型|detail|字节数|
|---|---|---|
|TINYINT|-127到127 2<sup>7</sup>-1|1|
|INT(num)|整数|4|
|BIGINT|大整数|8|

|浮点数类型|detail|字节数|
|---|---|---|
|FLOAT|单精度浮点数|4|
|DOUBLE|双精度浮点数|8|
|DECIMAL(M,D),DEC|M指总位数(小数点前+小数点后)，D指小数位数。不指定M与D时默认为(10,0)|M+2|

|字符串类型|detail|字节数|
|---|---|---|
|CHAR(M)|固定长度字符串，会删除尾部空格|M|
|CARCHAR(M)|变长度字符串，不删除尾部空格|L+1字节 L<=M and 1<=M<=255|
|TEXT|变长小字符串，不删除尾部空格|L+2字节 L<2 and <sup>16</sup>|

### 具体操作指令
1. 创建表
    1. 普通创建  `create table 表名 (字段名1 字段类型1, 字段名2 字段类型2…);`
    2. 创建 新表newtable与 oldtable表 结构相同 `create table newtable like oldtable;`
2. 查看表：
    1. 查看正在使用数据库中所有的表 `show tables;`
    2. 查看表结构 `desc 表名;`
3. 删除表： `drop table 表名;`
4. 修改表名： `rename table 原表名 to 新表名;`
5. 修改表内的列
    1. 增加列 `alter table 表名 add 列名 类型;`
    2. 修改某一列的类型 `alter table 表名 modify 列名 新类型;`
    3. 修改某一列列名 `alter table 表名 change 原列名 新列名 类型;`
    4. 删除一列 `alter table 表名 drop 列名;`
```shell
# 1. 创建表，同时这里用primary key进行限定主键
mysql> create table department
    -> (
    -> dept_name VARCHAR(20),
    -> building VARCHAR(15),
    -> budget numeric(12, 2),
    -> primary key(dep_name)
    -> );
Query OK, 0 rows affected (0.06 sec)

mysql> desc department;
+----------+---------------+------+-----+---------+-------+
| Field    | Type          | Null | Key | Default | Extra |
+----------+---------------+------+-----+---------+-------+
| dep_name | varchar(20)   | NO   | PRI | NULL    |       |
| building | varchar(15)   | YES  |     | NULL    |       |
| budget   | decimal(12,2) | YES  |     | NULL    |       |
+----------+---------------+------+-----+---------+-------+
3 rows in set (0.00 sec)
```

## 2.3 DML、DQL--操作表内数据，增删改+查
1. DQL--查看数据
    1. 查看表中所有数据 `selct * from 表名;`
    2. 查看表中部分字段数据 `selct 字段1,字段2... from 表名;`
    3. 清除重复值 `select distinct 字段1,字段2... from 表名;`
2. 插入数据
    1. 指定字段添加一条数据，没指定的部分默认为Null。这里字段与值的顺序是一一对应的。
       `insert into 表名 (字段1，字段2...) values (值1，值2...);`
    2. 直接对应全部字段添加一条数据，要完整且一对一对应表中定义的字段。
       `insert into 表名 values (值1，值2...);`
    3. 批量添加 在1的基础上，多条数据之间使用逗号。**批量插入数据的最佳方式，避免与数据库建立多次连接，增加服务器负荷**。
```shell
mysql> insert into department values ('Biology','Watson', 90000);
mysql> insert into
    -> department (dep_name, building, budget)
    -> values
    -> ('Elec.Eng','Taylor',85000),
    -> ('Finance','Painter',120000),
    -> ('History','Painter',50000),
    -> ('Music','Packard',80000),
    -> ('Physics','Watson',70000);
Query OK, 5 rows affected (0.01 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> select * from department;
+----------+----------+-----------+
| dep_name | building | budget    |
+----------+----------+-----------+
| Biology  | Watson   |  90000.00 |
| Comp.Sci | Taylor   | 100000.00 |
| Elec.Eng | Taylor   |  85000.00 |
| Finance  | Painter  | 120000.00 |
| History  | Painter  |  50000.00 |
| Music    | Packard  |  80000.00 |
| Physics  | Watson   |  70000.00 |
+----------+----------+-----------+
7 rows in set (0.00 sec)
```
3. 删除数据 `delete from 表名 [where 字段名=值]`
### 限定
1. 设置primary key 主键
    1. 可以 `字段名 类型 primary key,`这样设置
    2. 也可以在最后 `primary key(字段名),`
2. 删除primary key主键 `alter table 表名 drop primary key;`
3. 设置foreign key外键，**这个被关联的必须是对应table的PK主键(非空且唯一)**。
   这个table的主键被关联之后，不能被修改or删除。
   这样两个table-department+course之间的dept_name就是相互关联的。 `foreign key(在建表的字段名) references 已有表名(要关联的已有表的字段名)`
![[Pasted image 20230314182235.png#center|500]]
![[Pasted image 20230314195538.png#center|Navicat中的操作，比较简单|600]]
## 完善University数据库时遇到的问题
>[!question] **MySQL中的大小写问题**
> 类似create、alter、insert这一类的命令是大小写不敏感的。
> 但是在不同OS系统上，表名、列名有不同的规则，需要用统一的标准进行约束。例如在虚拟机上create database University; 与 create database university; 就是创建了两个不同的数据库。

>[!question] Cannot add or update a child row: a foreign key constraint fails 
> **原因分析**： 1. 数据类型不匹配； 2. 存储引擎不同； 3. 插入数据不匹配；
> **解决方法** ： 先临时关闭外键约束，增添完数据之后再打开
> set foreign_key_checks=0; # 关闭外键约束
> set foreign_key_checks=1; # 打开外键约束

# optional部分作答
1. 向department中添加一条数据；
![[Pasted image 20230314160357.png#center|1. 自行向数据库中添加数据|600]]
2. 删除刚刚添加的这条数据；
![[Pasted image 20230314161129.png#center|2. 删除一条数据库原有数据|500]]
3. 自行设计问题，查询相关数据；同时注意到查询结果的表格栏索引就是自己命令行写的顺序；
   1. 使用单关系查询
   2. 使用多关系查询
![[Pasted image 20230314195151.png#center|单关系查询--Comp.Sci计算机学院学分为3的课|600]]

![[Pasted image 20230314194937.png#center|多关系--查询春季开课老师名字|500]]
4. 自行限定条件对原数据库信息进行更新；
![[Pasted image 20230314174523.png#center|更改department中某一列的列名|500]]

![[Pasted image 20230314200053.png#center|把音乐学院3学分的课改成8学分|600]]

参考：
[MySQL的数据类型](https://mp.weixin.qq.com/s?__biz=MzA5NzQxOTE4NA==&mid=2247484005&idx=1&sn=9cfc06b6605cfc10056971a745e400e2&chksm=90a068baa7d7e1ac5f14f0b42d54ebb8effc1b1e9bb6035f22ea1eb6d2ad2dcfd35fc6323e0b&scene=27)
[使用MySQL](https://blog.csdn.net/E699A8/article/details/103625085)
[DQL、DML指令学习](https://blog.csdn.net/l546492845/article/details/127208488)
