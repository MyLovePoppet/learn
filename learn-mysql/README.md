# 安装MySQL
1. 编写my.ini文件如下：
```ini
[mysqld]
#换成自己的目录
basedir=D:\mysql-5.7.19-winx64\
datadir=D:\mysql-5.7.19-winx64\data\
port=3306
skip-grant-tables
```
2. 启动管理员模式下的CMD，并将路径切换至mysql下的bin目录，然后输入```mysqld –install``` (安装mysql)
3. 输入```mysqld --initialize-insecure --user=mysql```初始化数据文件
4. 再次启动mysql，然后用命令```mysql –u root –p```（注意-p后面不加空格）进入mysql管理界面（密码可为空）
5. 进入界面后更改root密码
```mySQL
update mysql.user set authentication_string=password('123456') where user='root' 
and Host = 'localhost';
```
6. 刷新权限```flush privileges;```
7. 修改 my.ini文件删除最后一句skip-grant-tables
8. 重启mysql即可正常使用
```cmd
net stop mysql
net start mysql
```
上述的执行过程如下
![img.png](https://i.niupic.com/images/2020/09/29/8Kty.png)
![img.png](https://i.niupic.com/images/2020/09/29/8KtB.png)
![img.png](https://i.niupic.com/images/2020/09/29/8KtD.png)
9. 连接上测试出现以下结果就安装好了
![img.png](https://i.niupic.com/images/2020/09/29/8KtE.png)

# 基本的SQL语句
```MySQL
update user set password=password('123456')where user='root';--修改密码
flush privileges;-- 刷新数据库
show databases;-- 显示所有数据库
use dbname;-- 打开某个数据库
show tables;-- 显示数据库mysql中所有的表
describe user;-- 显示表mysql数据库中user表的列信息
create database name;-- 创建数据库
use databasename;-- 选择数据库

exit; --退出Mysql
? 命令关键词 : 寻求帮助
-- 表示注释
```
# 数据库的操作
```SQL
create database [if not exists] 数据库名;--创建数据库

drop database [if exists] 数据库名;--删除数据库

show databases;--查看数据库

use `数据库名`;--使用数据库，如果数据库名是特殊字段需使用反引号
```
![img.png](https://i.niupic.com/images/2020/09/29/8KwX.png)

# 数据值和列的类型

1. 数值类型：
![img.png](https://i.niupic.com/images/2020/09/29/8Kx8.png)
（注意金融类数值需用decimal来进行存放数据）
2. 字符串类型
![img.png](https://i.niupic.com/images/2020/09/29/8Kxa.png)
3. 日期和时间型数值类型
![img.png](https://i.niupic.com/images/2020/09/29/8Kxc.png)
4. NULL值
>理解为 “没有值” 或 “未知值”。不要用NULL进行算术运算 , 结果仍为NULL

# 创建数据表
示例：
```MySQL
create table `student`
(
    `id`       int(4)      not null auto_increment comment '学号',
    `name`     varchar(30) not null default '匿名' comment '姓名',
    `pwd`      varchar(30) not null default '123456' comment '密码',
    `sex`      varchar(2)  not null default '女' comment '性别',
    `birthday` datetime             default null comment '出生日期',
    `address`  varchar(100)         default null comment '家庭住址',
    `email`    varchar(20)          default null comment '邮箱',
    primary key (`id`)
) engine = innodb
  default charset = utf8;
```
格式：
```sql
create table [if not exists] `表名`(
    `字段名1` 列类型 [属性] [索引] [注释],
    `字段名2` 列类型 [属性] [索引] [注释],
    `字段名3` 列类型 [属性] [索引] [注释],
    primary key (`字段名`)
);
```

可以使用命令```show create table student```来查看创建表的语句

![img.png](https://i.niupic.com/images/2020/10/02/8L5U.png)

# 数据库的存储引擎
INNODB（默认）和MYISAM（早些年使用的），其区别如下:

|              | MYISAM               | INNODB                  |
| :----------- | :------------------- | :---------------------- |
| 事务的支持   | 不支持               | 支持                    |
| 数据行锁定   | 不支持（只支持表锁） | 支持行锁（效率高）      |
| 外键约束     | 不支持               | 支持                    |
| 全文索引     | 不支持               | 支持                    |
| 表空间的大小 | 较小                 | 较大（约为MYISAM的2倍） |

常规使用操作：
- MYISAM 节约空间，速度较快
- INNODB 安全性较高，事务处理，多表多用户操作

**在物理空间存在的位置**

所有的数据库文件都存在mysql安装目录的data文件夹下，本质还是文件存储。

MySQL引擎在物理文件上的区别：
- INNODB在数据库表中只有一个*.frm文件，以及上级目录下的ibdata1文件。
- MYISAM对应文件
  - *.frm表结构定义文件
  - *.MYD 数据文件（data）
  - *.MYI 索引文件（index）

**设置数据库表的字符集编码**
```charset=utf8;```
如果不设置的话会使用MySQL默认的字符集编码（Latin1），不支持中文。

# 修改和删除数据表
## 修改表
关键字alert
```sql
alert table teacher rename as teacher1;--修改表名

alert table teacher1 add age int(10);--增加表的字段

alert table teacher1 modify age varchar(10);--修改约束，使用modify关键字

alert table teacher1 change age age1 int(10);--字段重命名

alert table teacher1 drop age1;--删除表的字段
```
modify与change关键字的区别总结：
- change用来字段重命名，不能修改字段类型和约束
- modify不能用来字段重命名，只能修改字段类型和约束

## 删除表
```sql
drop table [if exists] teahcer1;
```
- **注意点：**
  - **所有的创建和删除尽量加上判断，以免报错**
  - **字段名使用``符号进行包裹**
  - **sql关键字大小写不敏感，建议小写**
  - **所有的符号全部使用英文**

# MySQL数据管理
## 外键
![img.png](https://i.niupic.com/images/2020/10/02/8L68.png)
### 方式一，创建表的时候声明外键关系。
```mysql
create table `grade`
(
    `gradeid`   int(10)     not null auto_increment comment '年级id',
    `gradename` varchar(10) not null comment '年级名称',
    primary key (`gradeid`)
) engine = innodb
  default charset = utf8;

drop table if exists `student`;

create table if not exists `student`
(
    `id`       int(4)      not null auto_increment comment '学号',
    `name`     varchar(30) not null default '匿名' comment '姓名',
    `pwd`      varchar(30) not null default '123456' comment '密码',
    `sex`      varchar(2)  not null default '女' comment '性别',
    `birthday` datetime             default null comment '出生日期',
    `address`  varchar(100)         default null comment '家庭住址',
    `email`    varchar(20)          default null comment '邮箱',
    
    `gradeid`  int(10)     not null comment '年级id',
    primary key (`id`),
    -- 方式一
    key `FK_gradeid` (`gradeid`),
    constraint `FK_gradeid` foreign key (`gradeid`) references `grade` (`gradeid`)
) engine = innodb
  default charset = utf8;
-- 学生表的gradeid字段要去引用年级表的gradeid
-- 定义外键key[key `FK_gradeid` (`gradeid`)]
-- 给这个外键添加约束（执行reference引用）[constraint `FK_gradeid` foreign key (`gradeid`) references `grade` (`gradeid`)]
```

**注意：删除有外键约束的表的时候，必须先删除引用别人的表（从表），再删除被引用的表（主表）**

即student表引用了grade表，删除时要先删除student表，再删除grade表。

### 方式二，创建时不声明外键约束，后续增加外键约束
```mysql
drop table if exists `student`;

create table if not exists `student`
(
    `id`       int(4)      not null auto_increment comment '学号',
    `name`     varchar(30) not null default '匿名' comment '姓名',
    `pwd`      varchar(30) not null default '123456' comment '密码',
    `sex`      varchar(2)  not null default '女' comment '性别',
    `birthday` datetime             default null comment '出生日期',
    `address`  varchar(100)         default null comment '家庭住址',
    `email`    varchar(20)          default null comment '邮箱',
    `gradeid`  int(10)     not null comment '年级id',
    primary key (`id`)
) engine = innodb
  default charset = utf8;
-- 进行定义外键的语句
alter table `student`
    add constraint `FK_gradeid` foreign key (`gradeid`) references `grade` (`gradeid`);
```
以上的操作都是物理外键，数据库级别的外键，不建议使用（避免数据库过多造成困扰）。

最佳的实践方式：
- 数据库就是单纯的表，只用来存数据，只有行（数据）和列（字段）。
- 如果有多张表，想使用外键的话通过程序去实现。

## DML语言
DML语言：数据操作语言
- insert
- delete

### 插入insert 
```mysql
-- 插入语句
-- insert into `表名` ([字段1,字段2,字段3,...]) values(值1,值2,值3,...);
use shuqy_database;
-- 自增可以省略
insert into grade (`gradename`)
values ('大四');
-- 一次性插入多行数据
insert into grade (`gradename`)
values ('大三'),
       ('大二');

insert into student (`name`, `pwd`, `sex`, `birthday`, `address`, `email`, `gradeid`)
values ('shuqy', '123456', '男', '1999-12-10', 'SZU', '2393800169@qq.com', 1);
```
语法：**insert into `表名` ([字段1,字段2,字段3,...]) values(值1,值2,值3,...);**

注意事项：
1. 字段和字段之间使用英文逗号分开
2. 字段是可以省略的，但是后面的值必须要一一对应，不能少。
3. 可以同时插入多条数据，values后面的值使用英文逗号隔开即可。

### 修改update

>修改谁(查询条件) set原来的值=新值;
```mysql
update `student`
set `name` = '舒钦瑜'
where id = 1;-- 不指定条件会改动所有的表

-- 修改多个字段使用逗号隔开
update `student`
set `gradeid` = 2,
    `name`='shuqy'
where id = 1;
```
语法：```update 表名 set 字段名=值,[字段名=值,...] where [条件];```

where的操作符会返回bool值
| 操作符              | 含义                 | 范围           | 示例            | 结果  |
| :------------------ | :------------------- | :------------- | :-------------- | :---- |
| ==                  | 等于                 | 相等           | 5=6             | false |
| <>或!=              | 不等于               | 不相等         | 3<>4            | true  |
| >                   |
| <                   |
| >=                  |
| <=                  |
| between ... and ... | 区间                 | 在区间内       | between 3 and 5 |       |
| AND                 | 多个条件都符合       | 操作符内的&&   |
| OR                  | 多个条件符合一个即可 | 操作符内的\|\| |

注意：
- 字段名是数据库的列，尽量带上反引号
- 条件，筛选的条件，如果没有指定则会修稿所有的数据行
- value，是一个具体的值，也可以是一个变量（如current_time）
- 多个属性之间，使用英文逗号隔开。

### 删除delete
>delete命令

语法：delete from 表名 [where 条件];
```mysql
delete
from `grade`
where `gradeid` = 3;
```
>truncate命令

清除一个数据库表时可使用truncate命令，其作用为完全清空一个数据库表，保留数据库表的结构和索引约束。

命令：```truncate `student`;```

**delete和truncate的区别：**
- 相同点：都能删除数据，都不会删除表的结构
- 不同点：
  - truncate会重新设置自增列，使其计数器归零（delete则不会）。
  - truncate不会影响事务。

## DQL语言（数据库查询语言）
- 所有的查询操作都使用它，select
- 简单的查询与复杂的查询都能做
- 数据库中最核心的语言
- 使用频率最高的语句

### 查询
**查询全部的结果**
```mysql
-- 查询全部的结果，
select *
from `student`;

-- 查询指定的字段
select `studentno`, `studentname`
from `student`;

-- 指定别名，可以给字段起别名，也可以给表取别名
select `studentno` as `学号`, `studentname` as `学生姓名`
from `student`;

-- 函数concat(a,b)：字符串连接
select concat('姓名：', `studentname`) as `新名字`
from student;
```
语法：```select [字段名] from 数据表;```

可以使用 as 对字段名或者是表名取别名。


**去重**
```mysql
-- 使用distinct关键字来去重数据
select distinct `studentno`
from result;
```

**数据库的列（表达式）**
```mysql
select version();-- 查询函数

select 100 * 32 - 1 as `计算结果`;-- 用来计算（表达式）

select @@auto_increment_increment;
-- 查询自增的步长（变量）

-- 查询学员成绩+1分的结果
select `studentno`, `studentresult` + 1 as '提分'
from result
```
数据库中的表达式：文本，列，NULL，函数，计算表达式，系统变量...

**where条件查询**
```mysql
-- 查询考试成绩在95-100分之间的数据
select `studentno`, `studentresult`
from result
where `studentresult` between 95 and 100;

-- 查询学号不是1000的数据
select `studentno`, `studentresult`
from result
where `studentno` != 1000;

select `studentno`, `studentresult`
from result
where not `studentno` = 1000;
```
语法：-- select 表达式 from 表 where 条件;

逻辑运算符有and, or, not等。


**模糊查询：比较运算符**
| 运算符      | 语法              | 描述                                   |
| :---------- | :---------------- | :------------------------------------- |
| is null     | a is null         | 如果操作符结果为null，结果为真         |
| is not null | a is not null     | 如果操作符结果不是null，结果为真       |
| between     | a between b and c | 若结果在b和c之间，结果为真             |
| like        | a like b          | SQL匹配，如果a匹配b，则结果为真        |
| in          | a in(a1,a2,a3...) | 假设a在{a1,a2,a3...}集合中，则结果为真 |

例子如下：
```mysql
-- 模糊查询
-- like 结合%表示0或任意个字符 _则表示一个字符

-- 查询姓刘的同学
select `studentno`, `studentname`
from `student`
where `studentname` like '刘%';

-- 查询姓刘，而且刘后面只有一个字
select `studentno`, `studentname`
from `student`
where `studentname` like '刘_';

-- 查询名字中有嘉的姓名
select `studentno`, `studentname`
from `student`
where `studentname` like '%嘉%';

-- 查询在北京的学生
select `studentno`, `studentname`
from `student`
where `address` like '北京%';

-- 查询1001,1002,1003学号学员
select `studentno`, `studentname`
from `student`
where `studentno` in (1001, 1002, 1003);

-- null和not null
-- 查询地址为空的学生（null, ''）
select `studentno`, `studentname`
from `student`
where `address` in (null, '');

-- 查询有出生日期的学生
select `studentno`, `studentname`
from `student`
where `borndate` is not null;
```

**连表查询**
![img.png](https://i.niupic.com/images/2020/10/04/8Ltp.png)

```mysql
-- 连表查询join
/*
 1.分析需求，分析查询的字段来自于哪些表，（使用连接查询）
 2.确定使用哪种连接查询？
 3.确定交叉点，（两张表中哪个数据是相同的）
 */
-- 查询参加了考试的同学（学号，姓名，科目编号，分数）
select s.`studentno`, `studentname`, `subjectno`, `studentresult`
from student as s
         inner join result as r
where s.`studentno` = r.`studentno`;


select s.`studentno`, `studentname`, `subjectno`, `studentresult`
from student as s
         right join result r on s.studentno = r.studentno;

-- 使用left join则如果有学生没去考试也会被查询出来
```
| 操作       | 描述                                                             |
| :--------- | :--------------------------------------------------------------- |
| inner join | 如果表中至少有一个匹配则返回结果                                 |
| left join  | 会从左表中返回所有的值，即使右表中没有匹配（会使用null值来填充） |
| right join | 会从右表中返回所有的值，即使左表中没有匹配（会使用null值来填充） |

- 使用join on为连接查询
- 使用where为等值查询

两种方式的查询结果是相同的

三表连接查询示例：
```mysql
-- 查询参加了考试的同学信息：学号，学生姓名，科目名和分数
-- 三张表：student,result,subject
select s.`studentno`, `studentname`, `subjectname`, `studentresult`
from `student` s
         right join `result` r on s.`studentno` = r.`studentno`
         inner join subject s2 on r.`subjectno` = s2.`subjectno`;
```
![img.png](https://i.niupic.com/images/2020/10/05/8LEG.png)

**自连接查询**
核心：该表和自己相连接，即一张表拆为两张相同的表。
```mysql
CREATE TABLE `school`.`category`
(
    `categoryid`   INT(3)      NOT NULL COMMENT 'id',
    `pid`          INT(3)      NOT NULL COMMENT '父id 没有父则为1',
    `categoryname` VARCHAR(10) NOT NULL COMMENT '种类名字',
    PRIMARY KEY (`categoryid`)
) ENGINE = INNODB
  CHARSET = utf8
  COLLATE = utf8_general_ci;

INSERT INTO `school`.`category` (`categoryid`, `pid`, `categoryname`)
VALUES ('2', '1', '信息技术');
insert into `school`.`CATEGOrY` (`categoryid`, `pid`, `categoryname`)
values ('3', '1', '软件开发');
insert into `school`.`category` (`categoryid`, `PId`, `categoryname`)
values ('5', '1', '美术设计');
insert iNTO `school`.`category` (`categoryid`, `pid`, `categorynamE`)
VAlUES ('4', '3', '数据库');
insert into `school`.`category` (`CATEgoryid`, `pid`, `categoryname`)
values ('8', '2', '办公信息');
insert into `school`.`category` (`categoryid`, `pid`, `CAtegoryname`)
values ('6', '3', 'web开发');
insert into `school`.`category` (`categoryid`, `pid`, `categoryname`)
valueS ('7', '5', 'ps技术');
```
父类
| categoryid | categoryname |
| :--------- | :----------- |
| 2          | 信息技术     |
| 3          | 美术开发     |
| 5          | 美术设计     |
子类
| pid  | categoryid | categoryname |
| :--- | :--------- | :----------- |
| 3    | 4          | 数据库       |
| 2    | 8          | 办公信息     |
| 3    | 6          | web开发      |
| 5    | 7          | PS技术       |

```mysql
-- 查询父类子类关系
select a.`categoryname` as 父栏目, b.`categoryname` as 子栏目
from `category` as a,
     `category` as b
where a.categoryid = b.pid;
```
![img.png](https://i.niupic.com/images/2020/10/05/8LEP.png)

**排序和分页**
```mysql
-- 降序排列
select *
from `result`
order by `studentresult` desc;
```
排序：升序ASC，降序DESC

```limit a,b```表示从a开始查寻b条数据。
```mysql
-- 降序排列
select *
from `result`
order by `studentno` desc
limit 0,5;-- 从0开始，查5条数据
```
与网页中的分页查询的结果数据：```limit (n-1)*pageSize,pageSize```表示查询第n页，pageSize大小的数据。
```mysql
-- 查询Java第一学年，课程成绩排名前10的学生，而且分数要大于80的学生信息（学号，姓名，课程名，分数）
select stu.`studentno`, `studentname`, `subjectname`, `studentresult`
from `student` stu
         join `subject` sub on stu.`studentno` = sub.`subjectno`
         join `result` res on sub.`subjectno` = res.`subjectno`
where `subjectname` = 'Java第一学年'
  and `studentresult` > 80
order by `studentresult` desc
limit 0,10;
```


**子查询**
使用情况：where条件内的结果是计算出来的，不是固定的。其本质是在where里面嵌套一个select语句。
```mysql
-- 查询分数不小于80分的学生的学号和姓名
select distinct s.`studentno`, `studentname`
from `student` s
         join result r on s.`studentno` = r.`studentno`
where `studentresult` >= 80;

-- 增加一个条件，高等数学-2科目
select distinct s.`studentno`, `studentname`
from `student` s
         join `result` r on s.`studentno` = r.`studentno`
where `studentresult` >= 80
  and `subjectno` = (select `subjectno`
                     from `subject`
                     where `subjectname` = '高等数学-2');
-- 没有subjectname字段，故只能使用subjectno字段，涉及到子查询。
-- 双重子查询
select distinct `studentno`, `studentname`
from `student`
where `studentno` in (select `studentno`
                     from `result`
                     where `studentresult` >= 80
                       and `subjectno` = (select `subjectno`
                                          from `subject`
                                          where `subjectname` = '高等数学-2'));

-- 查询C语言-1的前五名的学生成绩的信息（学号姓名和分数）

select student.`studentno`, `studentname`, `studentresult`
from student
         join result r on student.studentno = r.studentno
where subjectno = (select `subjectno`
                   from subject
                   where subjectname = 'C语言-1')
order by `studentresult` desc
limit 0,5;
```



# MySQL的相关函数
## 常用函数
```mysql
select abs(-8); -- 绝对值
select ceiling(9.4);-- 向上取整
select floor(8.7); -- 向下取整
select rand(); -- 0~1之间的随机数
select sign(-5); -- sign 符号函数

select char_length('你好，世界');-- 字符串的长度
select concat('你好', '，', '世界');-- 拼接字符串
select insert('你好', 0, 2, '世界');
select lower('ShuQY');-- lower
select upper('shuqy');-- upper
select instr('abc', 'b');-- 查找
select replace('avcdsda', 'cd', 'ef');-- 替换
select substr('abcdefgb', 2);-- sub
select reverse('abcdef');


-- 时间日期函数
select current_date();
select current_time();
select current_timestamp();
select now();
select localtime();
select year(current_date());


select system_user();
select user();
```

## 聚合函数
```myql
| 函数名称 | 描述 |
| :------- | :--- |
| count()  | 计数 |
| sum()    | 求和 |
| avg()    | 平均 |

-- 聚合函数
select count(`studentname`) -- count()指定列，会忽略所有的null值
from student;

select count(*) -- count(*) 不会忽略null值
from student;

select count(1) -- count(1) 不会忽略null值，count(1)会比count(*)快
from student;


select sum(`studentresult`)
from result;

select avg(`studentresult`)
from result;

select max(`studentresult`)
from result;

select min(`studentresult`)
from result;



-- 查询不同课程的平均分最高分和最低分，平均分大于80分
-- 核心：根据不同的课程进行分组
select `subjectname`, avg(`studentresult`) as `平均分`, max(`studentresult`), min(`studentresult`)
from `result`
         join `subject` s on `result`.`subjectno` = s.`subjectno`
group by s.`subjectno` -- 通过什么字段来进行分组
having `平均分` >= 80;-- 这里不能使用where来进行判断条件，having为聚合之后的次要条件
```

# 数据库级别的md5加密
```mysql
create table if not exists `testmd5`
(
    `id`   int(4)      not null,
    `name` varchar(20) not null,
    `pwd`  varchar(50) not null,
    primary key (`id`)
) engine = innodb
  default charset = utf8;
-- 明文密码
insert into `testmd5`
values (1, 'zhangshan', '123456'),
       (2, 'lisi', '123456'),
       (3, 'wangwu', '123456');

-- 加密
update `testmd5`
set `pwd`=md5(`pwd`);

-- 插入的时候进行加密
insert into `testmd5`
values (4, 'xiaoming', md5('123456'));

-- 如何进行校验：将用户传入的密码进行md5加密后再进行对比
select *
from `testmd5`
where name = 'xiaoming'
  and `pwd` = md5('123456');
```


# 事务

**要么都成功，要么都失败**
------------
1. SQL执行   A给B转账     A 1000   --->200     B 200
2. SQL执行   B收到A的转账 A 800   --->200      B 400
------------
ACID:

**原子性（Atomicity）**

要么都成功，要么都失败

**一致性（Consistency）**

事务前后的数据完整性要保持一致

**隔离性（Isolation）**

事务的隔离性是多个用户并发访问数据库时，数据库为每一个用户开启的事务，不能被其他事务的操作数据所干扰，多个并发事务之间要相互隔离。

**持久性（Durability）**

事务一旦提交则不可逆，被持久化到数据库中

>隔离所导致的一些问题：

**脏读**

指一个事务读取了另外一个事务未提交的数据。

**不可重复读**

在一个事务内读取表中的某一行数据，多次读取结果不同。（这个不一定是错误，只是某些场合不对）

**幻读**

是指在一个事务内读取到了别的事务插入的数据，导致前后读取不一致。（一般是行影响，多了一行）

原理：
```mysql
set autocommit = 0;-- 数据库默认支持事务，需要通过该行来进行关闭事务

start transaction; -- 开始事务，从这之后的都表示一个事务内

commit; -- 提交：持久化成功

rollback; -- 回滚，回到原来的样子（失败）

set autocommit = 1;-- 表示开启

savepoint 保存点名; -- 设置一个保存点
rollback to savepoint 保存点名; -- 回滚到保存点
release savepoint 保存点名; -- 删除保存点
```
```mysql
-- 测试数据库的事务
create database `shop` character set utf8 collate utf8_general_ci;
use `shop`;
create table if not exists `account`
(
    `id`    int(3)        not null auto_increment,
    `name`  varchar(30)   not null,
    `money` decimal(9, 2) not null,
    primary key (`id`)
) engine = innodb
  default charset = utf8;

insert into `account` (`name`, `money`)
values ('A', 2000.00),
       ('B', 10000.00);

set autocommit = 0;
start transaction;

update account
set `money`= `money` - 500
where name = 'A';

update account
set `money`= `money` + 500
where name = 'B';

commit;-- 成功提交
rollback;-- 失败回滚

set autocommit = 1;-- 原先的自动提交
```
# 索引
>索引是帮助mysql高效获取数据的数据结构
索引的分类
- 主键索引（primary key）
  - 唯一的标识，主键不可重复，只能有一个列作为主键
- 唯一索引（unique key）
  - 避免重复的列出现，唯一索引可以重复，多个列都可以标识为唯一索引
- 常规索引（key/index）
  - 默认的，使用index或者key关键字来进行设置
- 全文索引（fulltext）
  - 在特定的数据库引擎下才有
  - 快速定位数据
```mysql
-- 增加一个索引
alter table `student`
    add fulltext index `studentName` (`studentname`);

-- explain 分析sql执行的状况

explain
select *
from student;
```
测试索引：
```mysql
-- 测试索引

CREATE TABLE `app_user`
(
    `id`          bigint(20) unsigned NOT NULL AUTO_INCREMENT,
    `name`        varchar(50)                  DEFAULT '',
    `eamil`       varchar(50)         NOT NULL,
    `phone`       varchar(20)                  DEFAULT '',
    `gender`      tinyint(4) unsigned          DEFAULT '0',
    `password`    varchar(100)        NOT NULL DEFAULT '',
    `age`         tinyint(4)                   DEFAULT NULL,
    `create_time` datetime                     DEFAULT CURRENT_TIMESTAMP,
    `update_time` timestamp           NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (`id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8;

-- 插入100万条数据
delimiter $$ -- 写函数之前标志

create function mock_data()
    returns int
begin
    declare num int default 1000000;
    declare i int default 1;
    while i < num
        do
            set i = i + 1;
            insert into `app_user`(`name`, `eamil`, `phone`, `gender`, `password`, `age`)
            values (concat('用户', i),
                    '2393800169@qq.com',
                    concat('18', floor(rand() * (999999999 - 100000000) + 100000000)),
                    floor(rand() * 2), UUID(), floor(rand() * 100));
        end while;
    return i;
end;

select mock_data();

explain
select *
from `app_user`
where `name` = '用户9999';-- 查询了992269行

alter table `app_user`
    add index `id_app_user_name` (`name`);-- 增加一个索引

explain
select *
from `app_user`
where `name` = '用户9999';-- 只查询了1行
```

索引在小数据时，区别不大，但是在大数据时区别十分明显。

**索引原则**
- 索引不是越多越好
- 不要对经常变动的数据加索引
- 小数据量的表不需要加索引
- 索引一般加在常用查询的字段。
- 










