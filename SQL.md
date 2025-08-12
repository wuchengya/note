# SQL

## 1创建表

```sql
create table sc(
	sno char(5),
	cno varchar(5),
	credit int
);
```

### 2主键

主键（Primary Key）是用于唯一标识表中每一行记录的字段或字段组合。

主键的定义方式主要有以下几种：

1. 在创建表时：
   - 单列主键：列后直接加 PRIMARY KEY。
   - 复合主键：表末尾使用 PRIMARY KEY (列1, 列2, ...)。
   - 使用 CONSTRAINT 命名主键。
2. 在已有表上：
   - 使用 ALTER TABLE ADD PRIMARY KEY (列)。
   - 使用 ALTER TABLE ADD CONSTRAINT 名称 PRIMARY KEY (列)。

```sql
//创建表时单列主键:
create table sc(
	sno char(5) primary key,
	cno varchar(5),
    credit int
);
```

```sql
//创建主键时复合主键
create table sc(
	sno char(5),
	cno varchar(5),
	credit int,
    primary key(sno,cno)
);
```

```sql
//使用constraint 关键字来添加主键
create table sc(
	sno char(5),
	cno varchar(5),
	credit int,
    constraint pk_sc primary key(sno,cno)
);
```

```sql
//在已有表上添加主键
create table sc(
	sno char(5),
	cno varchar(5),
	credit int
);
//不使用constraint
alter table sc add primary key(sno);
//使用constraint
alter table sc add constraint pk_sc primary key(sno);
```

constraint 可以理解为命名作用，和简单版本只有两字之差

### 3外键

外键（Foreign Key）是数据库中用来建立和强化两个表之间关联关系的字段，它指向另一个表的主键或唯一键，确保数据的参照完整性和一致性。

​	外键的定义方式可以归纳为以下几种：

1. 在创建表时：
   - **列级定义：** 在列定义后直接使用 REFERENCES 表(列)。
   - **表级定义：** 在表末尾使用 FOREIGN KEY (列) REFERENCES 表(列)。
   - **命名外键：** 使用 CONSTRAINT 名称 FOREIGN KEY (列) REFERENCES 表(列)。
2. 在已有表上：
   - **基本添加：** 使用 ALTER TABLE ADD FOREIGN KEY (列) REFERENCES 表(列)。
   - **命名外键：** 使用 ALTER TABLE ADD CONSTRAINT 名称 FOREIGN KEY (列) REFERENCES 表(列)。

```sql
create table course(
	sno char(5),
    cno varchar(5),
    infor varchar(10),
    primary key(sno,cno)
);
create table sc(
	sno char(5) references course(sno),
	cno varchar(5) references course(cno),
	credit int
);
//这种方式简单直观，但无法为外键约束指定自定义名称，系统会自动生成约束名。

create table sc(
	sno char(5),
	cno varchar(5),
	credit int,
    primary key (sno) references course(sno),
    primary key (cno) references course(cno),
);
//又是应该是比较容易看出外键

create table sc(
	sno char(5),
	cno varchar(5),
	credit int,
    constraint fk_sno primary key (sno) references course(sno),
    constraint fk_cno primary key (cno) references course(cno),
);
//在表级定义的基础上，可以通过 CONSTRAINT 关键字为外键约束指定一个自定义名称，便于后续管理。

//在已经建立的表上添加外键
alter table sc add foreign key(sno) references couse(sno);
//自定义约束名
alter table sc add constraint fk_sno foreign key(sno) references course(sno);
```

### 4unique约束

UNIQUE 约束是 SQL 中用于确保数据唯一性的重要工具。它的定义方式包括：

- 在创建表时

  ：

  - 列级定义：列后直接加 UNIQUE。
  - 表级定义：表末尾使用 UNIQUE (列)。
  - 复合 UNIQUE：UNIQUE (列1, 列2, ...)。
  - 使用 CONSTRAINT 命名约束。

- 在已有表上

  ：

  - 使用 ALTER TABLE ADD UNIQUE (列)。
  - 使用 ALTER TABLE ADD CONSTRAINT 名称 UNIQUE (列)。

1.unique约束的作用

- **确保数据唯一性**：unique 约束保证表中某列或某组列的值不会出现重复。例如，可以用来确保学生表中的学号或姓名不重复。
- **允许 NULL 值**：与主键（PRIMARY KEY）不同，unique 约束允许列中包含 NULL 值。在大多数数据库中，NULL 被视为不重复，因此可以有多个 NULL 值。

2.unique约束的的定义方式(和主键的定义方式一摸一样)

```sql
create table sc(
	sno char(5) unique,//之间在后面写
	cno varchar(5),
	credit int,
    unique(sno,cno)//列级定义，同时限定多个，清晰明了
    constraint unique_1 unique(sno,cno)//可以自己起约束名
);
alter table sc add unique(sno);
alter table sc add constraint unique_1 unique(sno,cno);
```

3.primary key 和unique的区别

+ primary key和unique一样保证了数据不重复

+ primary key只能出现一次，unique可以出现很多次

+ primary key不允许出现NULL，unique允许多次出现



### 5check,null,not null,default约束



```sql
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,  -- 主键约束
    FirstName VARCHAR(50) NOT NULL,  -- NOT NULL 约束，确保 FirstName 不能为空
    LastName VARCHAR(50) NOT NULL,  -- NOT NULL 约束，确保 LastName 不能为空
    Age INT CHECK (Age >= 18),  -- CHECK 约束，确保年龄大于或等于 18
    HireDate DATE DEFAULT GETDATE(),  -- DEFAULT 约束，如果没有提供 HireDate，则使用当前日期
    Salary DECIMAL(10, 2) NULL,  -- NULL 约束，允许 Salary 为空
    DepartmentID INT,
);
```

```sql
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName VARCHAR(50) NOT NULL,
    LastName VARCHAR(50) NOT NULL,
    Age INT,
    HireDate DATE DEFAULT GETDATE(),
    Salary DECIMAL(10, 2) NULL,
    DepartmentID INT,
    CONSTRAINT CHK_Age CHECK (Age >= 18)//check()约束的第二种用法
);
```

## 2修改表

添加新列:

```sql
alter table 表名 add 列名 数据类型 [约束]
//不带约束
alter table stu add email varchar(50);
//带约束
alter table stu add phone varchar(15) unique not null;
```

删除列：

```sql
alter table 表名 drop column 列名;
alter table str drop column email;//如果列被外键或其他约束引用，可能需要先删除相关约束。
```

添加主键，外键，unique约束在创建表中有

添加check约束

```sql
alter table 表名 add [constraint 约束名] check(条件)
alter table stu add constraint check_age check(age>=18);
```

添加default约束

```sql
alter table 表名 modify column 列名 数据类型 default 默认值;
alter table str modify column registrationdata data default current_data;	
```

添加not null约束（默认都是null）

```sql
alter table 表名 modify column 列名 数据类型 not null;
alter table str modify column sname varchar(20) not null;
```

添加操作讲完，下面是MYSQL的删除操作

删除主键:

```sql
alter table 表名 drop primary key;
//MySQL 不支持直接通过 DROP CONSTRAINT 删除主键，需使用 DROP PRIMARY KEY。
```

删除外键约束:

```sql
alter table 表名 drop foreign key 约束名;
alter table stu drop foreign key fk_sno;
//删除名为fk_sno的外键约束
//需要知道外键的名称，可通过 SHOW CREATE TABLE SC 查看。
```

删除unique约束:

```sql
alter table 表名 drop index 约束名;
//删除唯一约束只能通过删除唯一索引的方式删除。
//如果创建唯一约束时未指定名称，如果是单列，就默认和列名相同
//如果是有多个约束，那还是老实的取名字，然后删除名字
```

删除check()约束:

```sql
alter table 表名 drop check 约束名
alter table stu drop check check_age;
```

在MYSQL中，删除以上四个都是同样的逻辑，都要找到约束名。主键特殊一点

删除default 约束

```sql
alter table 表名 modify column 列名 数据类型
alter table str modify column registrationdata data;
//删除 RegistrationDate 的默认值（只需省略 DEFAULT 关键字,相比于添加这个约束）。
```

删除not null 约束,其实就是修改它为null

```sql
alter table 表名 modify 列名 数据类型 NULL;
//这个和前面的改为not null 一样的
```

## 3查询内容

```sql
//select
select 列1，列2... from 表;//所有信息可以使用*

//distinct 消除取值重复的行
select distinct 列1，列2... from 表;
//比如一个人选了很多课，要统计选课人数的时候，这个人的学号就会多次出现，使用distinct可以去重,作用范围是所有目标列。
```

**WHERE 子句常用查询条件表格**

| **条件类型**          | **说明**                                               | **语法/示例**                                                |
| --------------------- | ------------------------------------------------------ | :----------------------------------------------------------- |
| **等于 (=)**          | 检查值是否相等                                         | WHERE Age = 18                                               |
| **不等于 (!= 或 <>)** | 检查值是否不相等                                       | WHERE Age != 18 或 WHERE Age <> 18                           |
| **大于 (>)**          | 检查值是否大于指定值                                   | WHERE Age > 18                                               |
| **小于 (<)**          | 检查值是否小于指定值                                   | WHERE Age < 18                                               |
| **大于等于 (>=)**     | 检查值是否大于等于指定值                               | WHERE Age >= 18                                              |
| **小于等于 (<=)**     | 检查值是否小于等于指定值                               | WHERE Age <= 18                                              |
| **BETWEEN**           | 检查值是否在指定范围内（包含边界）                     | WHERE Age BETWEEN 18 AND 25                                  |
| **IN**                | 检查值是否在指定列表中                                 | WHERE City IN ('New York', 'London', 'Tokyo')                |
| **NOT IN**            | 检查值是否不在指定列表中                               | WHERE City NOT IN ('New York', 'London')                     |
| **LIKE**              | 模式匹配，使用通配符（% 表示任意字符，_ 表示单个字符） | WHERE Name LIKE 'A%' （以 A 开头的名字）                     |
| **NOT LIKE**          | 否定模式匹配                                           | WHERE Name NOT LIKE '%son' （不以 son 结尾）                 |
| **IS NULL**           | 检查值是否为 NULL                                      | WHERE Email IS NULL                                          |
| **IS NOT NULL**       | 检查值是否不为 NULL                                    | WHERE Email IS NOT NULL                                      |
| **AND**               | 逻辑与，要求所有条件都为真                             | WHERE Age > 18 AND City = 'London'                           |
| **OR**                | 逻辑或，要求至少一个条件为真                           | WHERE Age > 18 OR City = 'London'                            |
| **NOT**               | 逻辑非，否定条件                                       | WHERE NOT Age > 18 （等价于 Age <= 18）                      |
| **EXISTS**            | 检查子查询是否返回结果                                 | WHERE EXISTS (SELECT * FROM Orders WHERE Orders.StudentID = Student.ID) |
| **NOT EXISTS**        | 检查子查询是否不返回结果                               | WHERE NOT EXISTS (SELECT * FROM Orders WHERE Orders.StudentID = Student.ID) |

is null 不能使用=null代替

![image-20250613194144295](E:\Typora picture\image-20250613194144295.png)

数据的简单查询看似很多实则一个图概括。

这里讲一下分页查询，第一个参数是跳过多少条，第二个参数是每页有几条数据。



## 4函数

![image-20250610154425522](E:\Typora picture\image-20250610154425522.png)

![image-20250610154441562](E:\Typora picture\image-20250610154441562.png)

![image-20250610160027615](E:\Typora picture\image-20250610160027615.png)

![image-20250613194812431](E:\Typora picture\image-20250613194812431.png)

## 5多表查询

### 内连接

![image-20250613195339304](E:\Typora picture\image-20250613195339304.png)

### 外连接

![image-20250613195557498](E:\Typora picture\image-20250613195557498.png)

### 自连接

![image-20250613195708685](E:\Typora picture\image-20250613195708685.png)

## 6嵌套查询（子查询）

### 1标量子查询

第二个查询结果是单个值

```mysql
根据销售部门id,查询员工信息
A先查询出销售部门id
select id from dept where name = '销售部'
B根据销售部门id,求出员工信息
select * from emp where id = (select id from dept where name = '销售部');
```



### 2列子查询

第二个查询结果是一列

```mysql
查询销售部和市场部的所有员工信息
A查询销售部和市场部id
select id from dept where name = '销售部' or name = '市场部';
B根据id查询员工
select * from emp where id in (select id from dept where name = '销售部' or name = '市场部');
```



### 3行子查询

第二个查询结果是一行

```mysql
查询与张无忌薪资和直属领导相同的员工信息
A查询张无忌的薪资和领导
select salary,leader from emp where name = '张无忌';
B查询与张无忌薪资和直属领导相同的员工信息
select * from emp where (salary,leader) = (select salary,leader from emp where name = '张无忌');
```



### 4表子查询

第二个查询结果是一个表

```mysql
查询与郭嘉庆和赵文涛职位和薪资相同的员工信息
A查询郭嘉庆和赵文涛职位和薪资
select job,salary from emp where name = '郭嘉庆' or name = '赵文涛';
B查询与郭嘉庆和赵文涛职位和薪资相同的员工信息
select * from emp where (job,salary) in (select job,salary from emp where name = '郭嘉庆' or name = '赵文涛')
```

## 事务

### 事务四大特性

![image-20250617164754228](E:\Typora picture\image-20250617164754228.png)

### 并发事务问题

![image-20250617164643748](E:\Typora picture\image-20250617164643748.png)

### 事务隔离级别

![image-20250617165247557](E:\Typora picture\image-20250617165247557.png)

### 总结

![image-20250617171054426](E:\Typora picture\image-20250617171054426.png)

## 存储引擎

