# MySQL

## 执行顺序

![image-20220314104635347](Notes_MySQL.assets/image-20220314104635347.png)

## 增删改

### 插入

#### 插入记录的方式汇总

- 普通插入（全字段）：INSERT INTO table_name VALUES (value1, value2, ...)

- 普通插入（限定字段）：INSERT INTO table_name (column1, column2, ...) VALUES (value1, value2, ...)

- 多条一次性插入：INSERT INTO table_name (column1, column2, ...) VALUES (value1_1, value1_2, ...), (value2_1, value2_2, ...), ...

- 从另一个表导入：INSERT INTO table_name SELECT * FROM table_name2 [WHERE key=value]

  ```mysql
  INSERT INTO exam_record_before_2021(uid, exam_id, start_time, submit_time, score)
  SELECT uid, exam_id, start_time, submit_time, score
  FROM exam_record
  WHERE YEAR(submit_time) < '2021';
  ```

- ```mysql
  INSERT INTO actor(actor_id,
                    first_name,
                    last_name,
                    last_update)
  VALUES(1,'PENELOPE','GUINESS','2006-02-15 12:34:33'),
        (2,'NICK','WAHLBERG','2006-02-15 12:34:33');
  ```

#### 确保一定插入成功

##### replace

```mysql
REPLACE INTO examination_info
VALUES(NULL,9003,'SQL','hard',90,'2021-01-01 00:00:00');
```

1. 关键字NULL可以用DEFAULT替代。
2. 掌握replace into···values的用法

replace into 跟 insert into功能类似，不同点在于：replace into 首先尝试插入数据到表中，

1. 如果发现表中已经有此行数据（根据主键或者唯一索引判断）则先删除此行数据，然后插入新的数据；
2. 否则，直接插入新数据。

要注意的是：插入数据的表必须有主键或者是唯一索引！否则的话，replace into 会直接插入数据，这将导致表中出现重复的数据。

```mysql
# mysql中常用的三种插入数据的语句: 
# insert into表示插入数据，数据库会检查主键，如果出现重复会报错； 
# replace into表示插入替换数据，需求表中有PrimaryKey，
#             或者unique索引，如果数据库已经存在数据，则用新数据替换，如果没有数据效果则和insert into一样； 
# insert ignore表示，如果中已经存在相同的记录，则忽略当前新数据；
insert ignore into actor values("3","ED","CHASE","2006-02-15 12:34:33");
```

##### delete

```mysql
DELETE FROM examination_info
WHERE exam_id=9003;
INSERT INTO examination_info
VALUES(NULL,9003, 'SQL','hard', 90, '2021-01-01 00:00:00')
```



#### 不单独处理列的主键自增的处理方式

>  第一列在增加数据的时候，可以写为0或者null，这样添加数据可以自增， 从而可以添加全部数据，而不用特意规定那几列添加数据。

```mysql
insert into exam_record VALUES
(
    null,1001,9001, "2021-09-01 22:11:12","2021-09-01 23:01:12",90
),
(
    null,1002,9002, "2021-09-04 07:01:02",null,null
)
```

注意几点

1. insert into...values 如果要插入多行，指令是 values <row1>,<row2>...多行之间用逗号隔开；

2. AI(auto_increment)类型的列，在插入时用null/default关键字补位，在插入时sql会自动计算各行应有的值；

3. interval 时间间隔关键字，常和date_add() 或 date_sub()搭配使用。

   ```mysql
   insert into exam_record
   VALUES (null,1001,9001,'2021-09-01 22:11:12','2021-09-01 22:11:12' +INTERVAL 50 minute,90),
   (null,1002,9002,'2021-09-04 07:01:02',null,NULL);
   
   insert into exam_record(uid,exam_id,start_time,submit_time,score) 
   VALUES (1001,9001,'2021-09-01 22:11:12','2021-09-01 23:01:12' ,90),
   (1002,9002,'2021-09-04 07:01:02',null,NULL);
   ```

### 更新

修改记录的方式汇总：

- 设置为新值：UPDATE table_name SET column_name=new_value [, column_name2=new_value2] [WHERE column_name3=value3]

  - #注意set后面有多个对象用','而不是'and'连接

  - ```mysql
    UPDATE examination_info
    SET tag = "Python"
    WHERE tag = "PYTHON";
    
    update titles_test 
    set to_date = null,from_date = '2001-01-01'
    where to_date = '9999-01-01';
    ```

- 根据已有值替换：UPDATE table_name SET key1=replace(key1, '查找内容', '替换成内容') [WHERE column_name3=value3]

  - ```mysql
    UPDATE examination_info
    SET tag = REPLACE(tag, "PYTHON", "Python")
    WHERE tag = "PYTHON";
    ```

  - 思维扩展：第二种方式不仅可用于整体替换，还能做子串替换，例如要实现将tag中所有的PYTHON替换为Python（如CPYTHON=>CPython），可写作：

    - ```mysql
      UPDATE examination_info
      SET tag = REPLACE(tag, "PYTHON", "Python")
      WHERE tag LIKE "%PYTHON%";
      ```

#### Replace

REPLACE INTO当遇到primary 或者 unique key 的时候，会首先进行update

```mysql
REPLACE INTO titles_test 
    VALUES(5, 10005 ,'Senior Engineer', '1986-06-26', '9999-01-01') ;
```

将id=5以及emp_no=10001的行数据替换成id=5以及emp_no=10005

```mysql
UPDATE titles_test
SET emp_no = REPLACE(emp_no, 10001, 10005)
WHERE id = 5;
```

#### 将所有获取奖金的员工当前的薪水增加10%

方法1：连接查询（先join两张表）

```mysql
update salaries as s join emp_bonus as e on s.emp_no=e.emp_no
set salary=salary*1.1
where to_date='9999-01-01'
```

方法2：子查询

```mysql
update salaries
set salary=salary*1.1
where to_date='9999-01-01' 
    and salaries.emp_no in(select emp_no from emp_bonus)
```

讨论区:

```mysql
UPDATE salaries SET salary = salary * 1.1 WHERE emp_no IN
(SELECT s.emp_no FROM salaries AS s INNER JOIN emp_bonus AS eb 
ON s.emp_no = eb.emp_no AND s.to_date = '9999-01-01')
```



比较：
推荐使用连接查询（JOIN）
连接查询不需要创建+销毁临时表，因此速度比子查询快。

### 删除

删除记录的方式汇总：

- 根据条件删除：DELETE FROM tb_name [WHERE options] [ [ ORDER BY fields ] LIMIT n ]
- 全部删除（表清空，包含自增计数器重置）：TRUNCATE tb_name
  -  truncate 删除表中的所有行，但表的结构及其列，约束，索引等保持不变。新行标识所用的技术值重置为该列的种子。如果想保留标识计数值，轻盖拥delete 。如果要删除表定义及其数据，请使用drop table 语句。
- DROP TABLE [IF EXISTS] 表名1 [, 表名2]

时间差：

- TIMESTAMPDIFF(interval, start_time,end_time)可计算start_time-end_time的时间差，单位以指定的interval为准，常用可选：
  - SECOND 秒
  - MINUTE 分钟（返回秒数差除以60的整数部分）
  - HOUR 小时（返回秒数差除以3600的整数部分）
  - DAY 天数（返回秒数差除以3600*24的整数部分）
  - MONTH 月数
  - YEAR 年数
- end_time - start_time差过1分钟的话，它的进制是100，不是60。而timestampdiff(second,start_time,end_time)进制得到的是60。比如start_time是2021-09-01 10:00:00，end_time是2021-09-01 10:01:06，timestampdiff(second,start_time,end_time) 计算得到的是66，而end_time - start_time得到的是106

```mysql
DELETE FROM exam_record
WHERE TIMESTAMPDIFF(MINUTE, start_time, submit_time) < 5
    AND score < 60;
```

```mysql
请删除exam_record表中未完成作答或作答时间小于5分钟整的记录中，开始作答时间最早的3条记录。
delete from exam_record
where timestampdiff(minute,start_time, submit_time) < 5 or submit_time is null
order by start_time limit 3;
```

```mysql
删除exam_record表中所有记录；
并重置自增主键；
truncate exam_record;
或者
DELETE FROM exam_record;
ALTER TABLE exam_record auto_increment=1;
```

### 删除emp_no重复的记录

错误方法:**MySQL中不允许在子查询的同时删除表数据（不能一边查一边把查的表删了）**

```mysql
DELETE FROM titles_test
WHERE id NOT IN(
    SELECT MIN(id)
    FROM titles_test
    GROUP BY emp_no);
```

正确方法

```mysql
DELETE FROM titles_test
WHERE id NOT IN(
    SELECT * FROM(
    SELECT MIN(id)
    FROM titles_test
    GROUP BY emp_no)a);  -- 把得出的表重命名那就不是原表了（机智.jpg
```



### 创建表

创建数据表时，表名和字段名不需要用引号括起来。因此，下面的代码是错误的：

```mysql
CREATE TABLE 'actor'(
'actor_id' smallint(5) primary key,
'first_name' varchar(45) not null,
'last_name' varchar(45) not null,
'last_update' date not null);
```

正确示例：

```mysql
CREATE TABLE actor (
    actor_id SMALLINT ( 5 ) NOT NULL COMMENT '主键id',
    first_name VARCHAR ( 45 ) NOT NULL COMMENT '名字',
    last_name VARCHAR ( 45 ) NOT NULL COMMENT '姓氏',
    last_update date NOT NULL COMMENT '日期',
PRIMARY KEY ( actor_id ) 
)
```



- 1.1 直接创建表：

  ```mysql
  CREATE TABLE
  [IF NOT EXISTS] tb_name -- 不存在才创建，存在就跳过
  (column_name1 data_type1 -- 列名和类型必选
    [ PRIMARY KEY -- 可选的约束，主键
     | FOREIGN KEY -- 外键，引用其他表的键值
     | AUTO_INCREMENT -- 自增ID
     | COMMENT comment -- 列注释（评论）
     | DEFAULT default_value -- 默认值
     | UNIQUE -- 唯一性约束，不允许两条记录该列值相同
     | NOT NULL -- 该列非空
    ], ...
  ) [CHARACTER SET charset] -- 字符集编码
  [COLLATE collate_value] -- 列排序和比较时的规则（是否区分大小写等）
  ```

- 1.2 从另一张表复制表结构创建表

```mysql
CREATE TABLE tb_name LIKE tb_name_old
```

- 1.3 从另一张表的查询结果创建表：

```mysql
CREATE TABLE tb_name AS SELECT * FROM tb_name_old WHERE options
```

- 2.1 修改表：`ALTER TABLE 表名 修改选项` 。选项集合：

  ```mysql
      { ADD COLUMN <列名> <类型>  -- 增加列
       | CHANGE COLUMN <旧列名> <新列名> <新列类型> -- 修改列名或类型
       | ALTER COLUMN <列名> { SET DEFAULT <默认值> | DROP DEFAULT } -- 修改/删除 列的默认值
       | MODIFY COLUMN <列名> <类型> -- 修改列类型
       | DROP COLUMN <列名> -- 删除列
       | RENAME TO <新表名> -- 修改表名
       | CHARACTER SET <字符集名> -- 修改字符集
       | COLLATE <校对规则名> } -- 修改校对规则（比较和排序时用到）
  ```

  ### 细节剖析：

  - 自增ID：AUTO_INCREMENT；

  - 设置主键：PRIMARY KEY；

  - 唯一性约束：UNIQUE

  - 非空约束：NOT NULL

  - 设置默认值：DEFAULT 0

  - 当前时间戳：CURRENT_TIMESTAMP

  - 评论/注释：COMMENT

  - 如果该表已创建过，正常返回：IF NOT EXISTS

  - 列名是关键字的话用

    ```
    `level` 【用``】
    ```

### 修改表

```mysql

#alter table 增加的表格 add 增加列的名称 数据类型 位置(after level 在level 之后)
alter table user_info add school varchar(15) after level;
增加列在某列之后

#alter table user_info change 原列名 修改列名 修改数据类型
alter table user_info change job profession varchar(10);
更换列的名称及数据类型

#alter table 表名 modify 修改列名称 数据类型 默认值等
alter table user_info modify achievement int(11) default 0;
更改数据类型

alter table actor add create_date datetime not null default '2020-10-01 00:00:00' after last_update;
```

- alter table 表名 change 原列名 新列名 类型； --修改表的列属性名

  alter table 表名 modify 列名 类型 ； --修改表的类类型
  alter table 表名 drop 列名； --删除表的某一列
  alter table 表名 add 列名 类型；--添加某一列
  alter table 表名 rename 新表名； --修改表名

### 创建视图

1.直接在视图名的后面用小括号创建视图中的字段名

```mysql
# CREATE VIEW <视图名> AS <SELECT 语句>
create view actor_name_view (first_name_v,last_name_v) as
select first_name ,last_name from actor
```

2.在select后面对列重命名为视图的字段名

```mysql
create view actor_name_view as
select first_name as first_name_v
       ,last_name as last_name_v
from actor; # 注意一定要有;
```

### 创建外键

创建外键语句结构：

ALTER TABLE <表名>

ADD CONSTRAINT FOREIGN KEY (<列名>)

REFERENCES <关联表>（关联列）

```mysql
ALTER TABLE audit
ADD CONSTRAINT FOREIGN KEY (emp_no)
REFERENCES employees_test(id);
```



### 创建索引

索引创建、删除与使用：

- 1.1 create方式创建索引：

```mysql
CREATE 
  [UNIQUE -- 唯一索引
  | FULLTEXT -- 全文索引
  ] INDEX index_name ON table_name -- 不指定唯一或全文时默认普通索引
  (column1[(length) [DESC|ASC]] [,column2,...]) -- 可以对多列建立组合索引  
```

- 1.2 alter方式创建索引：`ALTER TABLE tb_name ADD [UNIQUE | FULLTEXT] [INDEX] index_content(content)`
- 2.1 drop方式删除索引：`DROP INDEX <索引名> ON <表名>`
- 2.2 alter方式删除索引：`ALTER TABLE <表名> DROP INDEX <索引名>`
- 3.1 索引的使用：
  - 索引使用时满足最左前缀匹配原则，即对于组合索引(col1, col2)，在不考虑引擎优化时，条件必须是col1在前col2在后，或者只使用col1，索引才会生效；
  - 索引不包含有NULL值的列
  - 一个查询只使用一次索引，where中如果使用了索引，order by就不会使用
  - like做字段比较时只有前缀确定时才会使用索引
  - 在列上进行运算后不会使用索引，如year(start_time)<2020不会使用start_time上的索引

```mysql
-- 唯一索引
ALTER TABLE examination_info
ADD UNIQUE INDEX uniq_idx_exam_id(exam_id);

-- 全文索引
ALTER TABLE examination_info
ADD FULLTEXT INDEX full_idx_tag(tag);

-- 普通索引
ALTER TABLE examination_info
ADD INDEX idx_duration(duration);

# alter 可以一起写
alter table actor  
add unique index uniq_idx_firstname(first_name), add index idx_lastname(last_name);
```

或者

```mysql
CREATE INDEX idx_duration ON examination_info(duration);
CREATE UNIQUE INDEX uniq_idx_exam_id ON examination_info(exam_id);
CREATE FULLTEXT INDEX full_idx_tag ON examination_info(tag);

# 用create的话要分开写
create unique index uniq_idx_firstname on actor(first_name);
create  index idx_lastname on actor(last_name);
```

### 使用强索引进行查询

解题思路：先创建索引，再创建强制索引查询，（题目这里默认已经创建索引）。索引名一定要加括号，否则错误。

```mysql
create index idx_emp_no on salaries(emp_no);
select * from salaries FORCE INDEX (idx_emp_no) where emp_no = 10005;
```



### 删除索引

```mysql
方法一：
alter table examination_info drop index uniq_idx_exam_id;
alter table examination_info drop index full_idx_tag;
方法二：
drop index   uniq_idx_exam_id on examination_info;
drop index  full_idx_tag  on examination_info;
```

### 触发器

在MySQL中，创建触发器语法如下：

```mysql
create trigger triggerName       
    after/before/        // 触发时间 trigger_time
    insert/update/delete // 监视事件 trigger_event
    on table_name        // 监视地点 table
    for each row         // 这句话在mysql中是固定的
    begin
    sql语句(insert/update/delete);             // 触发事件  trigger_stmt 注意这里要有分号 
    end;
```

其中：

- trigger_name：标识触发器名称，用户自行指定；
- trigger_time：标识触发时机，取值为 BEFORE 或 AFTER；
- trigger_event：标识触发事件，取值为 INSERT、UPDATE 或 DELETE；
- tbl_name：标识建立触发器的表名，即在哪张表上建立触发器；
- trigger_stmt：触发器程序体，可以是一句SQL语句，或者用 BEGIN 和 END 包含的多条语句，每条语句结束要分号结尾。

【NEW 与 OLD 详解】
MySQL 中定义了 NEW 和 OLD，用来表示
触发器的所在表中，触发了触发器的那一行数据。
具体地：

1. 在 INSERT 型触发器中，NEW 用来表示将要（BEFORE）或已经（AFTER）插入的新数据；
2. 在 UPDATE 型触发器中，OLD 用来表示将要或已经被修改的原数据，NEW 用来表示将要或已经修改为的新数据；
3. 在 DELETE 型触发器中，OLD 用来表示将要或已经被删除的原数据；
   使用方法： NEW.columnName （columnName 为相应数据表某一列名）

```mysql
create trigger audit_log 
after insert on employees_test
for each row
begin 
    insert into audit values(new.id,new.name);
end
```

## 简单处理查询结果

### 结果去重

```mysql
1. 使用DISTINCT：SELECT DISTINCT university from user_profile;
2. 使用group by：SELECT university from user_profile GROUP BY university;
```

### 限制返回行数

```mysql
--LIMIT n：从第0+1(m=0)条开始，取n条数据，是LIMIT 0,n的缩写
SELECT device_id
FROM user_profile
LIMIT 2 

--LIMIT m,n:从第m+1条开始，取n条数据
SELECT device_id
FROM user_profile
LIMIT 0,2 #下标从0开始计算

--LIMIT n OFFSET m：从第m+1条开始，取n条数据
SELECT device_id
FROM user_profile
LIMIT 2 OFFSET 0
```

### 列重命名

```mysql
SELECT device_id [as] user_infors_example
FROM user_profile
LIMIT 2
```



## 条件查询

### 正则表达式

RLIKE后面可以跟正则表达式。

正则表达式``"^[0-9]+$"``的意思：

1、字符^

意义：表示匹配的字符必须在最前边。

例如：^A不匹配“an A”中的‘A’，但匹配“An A”中最前面的‘A’。

2、字符$

意义：与^类似，匹配最末的字符。

例如：t$不匹配“eater”中的‘t’，但匹配“eat”中的‘t’。

3、字符[0-9]

意义：字符列表，匹配列出中的任一个字符。你可以通过连字符-指出字符范围。

例如：[abc]跟[a-c]一样。它们匹配“brisket”中的‘b’和“ache”中的‘c’。

4、字符+

意义：匹配+号前面的字符1次及以上。等价于{1,}。

例如：a+匹配“candy”中的‘a’和“caaaaaaandy”中的所有‘a’。

### 不等于某字符

```mysql
#where university != '复旦大学'
#where university not like '复旦大学'
# where university not in ('复旦大学')
```

### 过滤空值

```mysql
(1) Where 列名 is not null
(2) Where 列名 != 'null'
(3) Where 列名 <> 'null'
```

### 多列排序

```mysql
select device_id, gpa, age from user_profile
order by gpa asc, age asc;

select device_id, gpa, age from user_profile
order by gpa asc, age desc;
```

### 某个范围的数据

```mysql
SELECT device_id,gender,age
FROM user_profile
where age>=20 and age<=23;

SELECT device_id,gender,age from user_profile where age between 20 and 23;

SELECT device_id,gender,age FROM user_profile WHERE age IN (20,21,22,23);

```

### 查找所有数据

```mysql
# 一共四个学校
SELECT device_id,gender,age, university, gpa
from user_profile
where university in ('北京大学','复旦大学','山东大学');
# where university="北京大学" or university="复旦大学" or university="山东大学"
# where university NOT IN ("浙江大学");

 #上面直接把列名写出来比下面速度快
SELECT *
from user_profile
where university in ('北京大学','复旦大学','山东大学');
```

### 多个条件-操作符混合运用

```mysql
法1 union
SELECT device_id, gender, age, university,gpa from user_profile where gpa > 3.8 and university = '复旦大学' 
UNION
SELECT device_id, gender, age, university,gpa from user_profile where gpa > 3.5 and university = '山东大学'
order by device_id asc;

法2 or
SELECT 
    device_id, gender, age, university,gpa 
from 
    user_profile 
where 
    (gpa > 3.8 and university = '复旦大学') or (gpa > 3.5 and university = '山东大学')
```

### 字符串匹配

```mysql
SELECT device_id, age, university
FROM user_profile
WHERE university REGEXP "北京"

SELECT device_id,age,university
FROM user_profile
where university like "%北京%"
```

## 高级查询

### 分组函数

功能：用作统计、又叫做聚合函数或统计函数或组函数

分类：sum  avg max min count

特点：

1. sum avg 一般处理数值型

   min max count 可以处理任何类型

2. 以上分组函数都直接忽略null

3. 可以和distinct使用实现去重 count(distinct salary)

4. count的单独介绍

   1. count(A) 忽略null

   2. count(*) 一行里面只要有一个不是Null就算1

   3. count(1) 相当于多了一列，一列都是1，统计1的个数，统计行数。改成(2)也可以，就相当于列都是2，也是统计函数。

   4. 效率：

      MYISAM存储引擎下，count(*)效率高

      INNODB存储引擎下，count(*)和count(1)效率差不多，比count(字段)要高

   5. 一般使用count(*)统计行数

5. 和分组函数一同查询的字段有限制，要求是group by 后的字段。

   错误实例：

   > select avg(day), id from employee.

### 分组查询

语法：

> select 分组函数, 列（要求出现在group by后面) 【查询列表必须是分组函数和group by后出现的字段】
>
> from 表 where 分组前的筛选条件【对原始表的筛选，不能使用分组函数】
>
> group by 分组的列表 having 分组后的筛选条件【分组后的结果集】
>
> [order by]

特点：

1. 能用分组前筛选的，就优先考虑使用分组前筛选
2. group by字距支持单个字段分组，多个字段分组（多个字段之间用逗号隔开没有顺序要求）
3. 表达式或函数分组用的较少
4. 也可以添加排序，排序放在整个分组最后。
4. where后面不能直接跟组函数，要用的话要放在select子句里面

#### 按表达式或函数分组

按员工姓名的长度分组，查询每一组的员工个数，筛选员工个数>5的

```mysql
select count(*), length(last_name) len_name
from employee
group by length(last_name)
having count(*)>5

#换成别名也ok
select count(*) c, length(last_name) len_name
from employee
group by len_name
having c>5
```

#### 按多个字段分组

每个部门每个工种的员工的平均工资

```mysql
select avg(salary) a, depart_id, job_id
from employee
group by job_id, depart_id
order by a desc; #换成别名也ok
```



### Exists相关子查询

查有员工的部门名

```
select dep_name from depa d
where exists (select * from employee e
where d.id = e.d_id)
```

```
select dep_name from depa d
where d.id in (
select d.id from employee);
```

没女朋友的人

```
select bo.*
from boys bo
where not exists(
select boyid from beauty b
where bo.id = b.bf_id
)
```

### 外连接

外连接 (OUTER JOIN)分为三种
**1. 左外连接 (LEFT OUTER JOIN 或 LEFT JOIN）：**左表的记录将会全部表示出来，而右表只会显示符合搜索条件的记录，右表记录不足的地方均为NULL
**2. 右外连接 (RIGHT OUTER JOIN 或 RIGHT JOIN)：**与左(外)连接相反，右(外)连接，右表的记录将会全部表示出来，而左表只会显示符合搜索条件的记录，左表记录不足的地方均为NULL
**3. 全外连接 (FULL OUTER JOIN 或 FULL JOIN)：**左表和右表都不做限制，所有的记录都显示，两表不足的地方用null 填充
**MySQL中不支持全外连接，可以使用 UNION 来合并两个或多个 SELECT 语句的结果集**

### 最高值

```mysql
# 复旦大学学生gpa最高值是多少
(1) select max(gpa) from user_profile
where university = '复旦大学';

(2)select gpa
from
(select gpa,
row_number()over(partition by university order by gpa desc) as ranking
from user_profile
where university = '复旦大学') as t
where t.ranking = 1;

这个方法实际上会在写法上麻烦很多 但是毕竟我们是学习 一个题可以尽量多学几种做法 增加自己的基础
这里我使用的是窗口排序函数加子查询解题
用row_number()over() 在学校组里对GPA进行倒序排序 再增加条件rank=1 此时为GPA最大值
我也不记得这是第几次刷题了 每一次都能想到不同的方法 希望自学的朋友可以多思考 提升自己的能力 一起加油
带row_number()over函数的语句，一定要在此嵌入的sql 语句后面加上别名（比如此例c），否则会报错\

(3)SELECT gpa
FROM user_profile
WHERE university ="复旦大学"
ORDER BY gpa DESC
LIMIT 1;
```

### 统计数量以及平均值

```mysql
SELECT COUNT(gender) AS male_num,
ROUND(AVG(gpa),1) AS avg_gpa
FROM user_profile
WHERE gender = 'male';

select count(gender),round(avg(gpa),1) from user_profile
where gender='male'
group by gender
```

### 对多个变量分组

```mysql
#对性别和学校分组
SELECT gender,university,count(gender) as user_num,
avg(active_days_within_30) as avg_active_days,avg(question_cnt) as avg_question_cnt
from user_profile
GROUP by university,gender;
```

### 分组过滤

```mysql
select university, avg(question_cnt) avg_question_cnt, 
avg(answer_cnt) avg_answer_cnt
from user_profile
group by university
having avg(question_cnt) < 5 or avg(answer_cnt) < 20;
```

### 分组排序

```mysql
方法一：需要单独命名avg_question
select university,avg(question_cnt) AS avg_question_cnt from user_profile
group by university 
order by avg_question_cnt
方法二：不需要单独命名
select university,avg(question_cnt) from user_profile
group by university 
order by avg(question_cnt)
这样操作能运行成功的原因是 order by后面可以加聚合函数。但要注意，group by后面不可
有三类后面可以加聚合函数
1、select
2、order by
3、having
```

### 多表查询

```mysql
#内连接
select q.device_id, q.question_id, result 
from question_practice_detail q join user_profile u 
on q.device_id = u.device_id
where u.university = '浙江大学'
order by q.question_id asc;

# 连接查询
select question_practice_detail.device_id,question_id,result
from question_practice_detail,user_profile
where question_practice_detail.device_id=user_profile.device_id and user_profile.university='浙江大学';

# 子查询
select device_id,question_id,result 
from question_practice_detail 
where device_id=(
     select device_id 
     from user_profile
     where university="浙江大学"
); # 这里=改成in也行
```

```mysql
/*
分析思路
select 查询结果 [university,'每个人平均答题数量':问题数/设备数(一个设备对应多个题，要去重)]
from 从哪张表中查找数据[两张表联结]
where 查询条件 [无]
group by 分组条件 [university]
*/

-- avg_answer_cnt = count(question_id)/count(device_id)  
select university,(count(question_id)/count(distinct(q.device_id))) 
as avg_answer_cnt 
from user_profile u 
join question_practice_detail q on u.device_id = q.device_id
group by university;

select university, count(question_id)/count( distinct u.device_id)
from user_profile u, question_practice_detail q
where u.device_id = q.device_id
group by university
order by university;
```

### 三表链接

```mysql
select university, difficult_level, count(q.question_id)/count(distinct q.device_id)
from question_practice_detail q join question_detail d
on q.question_id = d.question_id join user_profile u on q.device_id = u.device_id
group by university, difficult_level;

select university , difficult_level , count(qpd.question_id)/count(distinct qpd.device_id) avg_answer_cnt
from user_profile up , question_practice_detail qpd , question_detail qd
where up.device_id = qpd.device_id and qpd.question_id = qd.question_id
group by university , difficult_level;
```

### 组合查询

```mysql
#注意输出的顺序，先输出学校为山东大学再输出性别为男生的信息, 要求不去重

其实这里的不去重表示：只要满足一个条件就被筛选出来，但总会存在一个人满足了两个条件只筛选一次。这里的坑时使用or，因为or自带去重，而union等价于or，UNION默认去重, 但union all 可以不去重，所以本体考察or与union的细节使用。

select device_id , gender , age , gpa from user_profile where university = '山东大学' 
UNION ALL
select device_id , gender , age , gpa from user_profile where gender = 'male'
```

## 常用函数

概念：类似java的方法，对外暴露方法名

好处：

1. 隐藏了实现细节
2. 提高代码重用性

调用：

> select 函数名(实参列表) [from 表]; 【[]表示可以省略】

特点：

1. 函数名
2. 函数功能

分类：

1. 单行函数：concat length ifnull
   1. 字符函数 length concat substr instr trim upper lower lpad rpad replace
   2. 数学函数 rounc ceil floor mod truncate
   3. 日期函数 now curdate curtime year month monthname day hour minute second str_to_date, data_format
   4. 其他函数 version database user
   5. 控制函数 if case
2. 分组函数：做统计使用的，又称为统计函数、聚合函数、组函数

### 类型转换

#### cast

Cast(字段名 as 转换的类型 )，其中类型可以为：

CHAR[(N)] 字符型 
DATE  日期型
DATETIME  日期和时间型
DECIMAL  float型
SIGNED  int
TIME  时间型

unsigned 是mysql自定义的类型，表示**无符号数值即非负数**。signed为整型默认属性。

### 字符函数

#### length

```
select length('aaa'); 
> 3
select length('张三') 
> 9 [utf-8 一个中文3个字节]
```

#### CHAR_LENGTH

```mysql
筛选昵称字符数大于10的用户：
WHERE CHAR_LENGTH(nick_name) > 10
```



#### concat

拼接字符串

```
select concat(last_name,'_',first_name)
from employee
```

#### concat.ws

```mysql
->对于concat函数而言，其中可以有一个写法是用于连接两个字段（列）的：
concat.ws ----(ws: with separator)
语法为：
concat.ws('分隔符'， str1, str2,...)

所以该题在sql(mysql8.0)版本中的答案是：
select concat_ws(' ', last_name, first_name) as Name
from employees
```



#### upper lower

```
select upper('john')
-> 'JOHN'
select lower('JOHN')
-> 'john'
select concat(upper(last_name),'_',lower(first_name)) from employee;
```

#### substr = substring

```
select substr('abcdefg',2) 
> bcdefg [索引从1开始]，截取从2开始的所有字符
select substr('陈乐华最聪明',4,3)
> 最聪明[截取从1开始，截取3个字符长度]
```

#### left

`LEFT()`函数是一个字符串函数，它返回具有指定长度的字符串的左边部分。

下面是`LEFT()`函数的语法 - 

```sql
LEFT(str,length);
SELECT LEFT('MySQL LEFT', 5);
> MySQL
```

`LEFT()`函数接受两个参数：

- `str`是要提取子字符串的字符串。
- `length`是一个正整数，指定将从左边返回的字符数。

`LEFT()`函数返回`str`字符串中最左边的长度字符。如果`str`或`length`参数为`NULL`，则返回`NULL`值。

如果`length`为`0`或为负，则`LEFT`函数返回一个空字符串。如果`length`大于`str`字符串的长度，则`LEFT`函数返回整个`str`字符串。

#### instr

```
select instr('杨不悔爱上了殷六侠','殷六侠')
> 7 [返回第一次出现的起始索引]
select instr('殷六侠爱上了殷六侠','殷六侠')
> 1 [返回第一次出现的起始索引]
select instr('殷六侠爱上了殷六侠','殷8侠')
> 0 [找不到返回0]
```

#### trim

```
select trim('   aaa   ')
> 'aaa' [删除前后空格]
select trim('a' from 'aaaabbabbaaaa')
> 'bbabb' [删除前后的a]
select trim('aa' from 'aaaaabbabbaaaa')
> 'abbabb' [删除前后的aa]
```

#### lpad

```
select lpad('啦啦啦',10,'*')
> *******啦啦啦 [用指定的字符左填充到指定长度]
select lpad('啦啦啦',2,'*')
>'啦啦' [超出了就会截断]
```

#### rpad

```
select rpad('啦啦啦',6,'ab')
> 啦啦啦aba
```

#### replace

```
select replace('你们好，你吗','你','我')
> 我们好，我吗 [全部都会替换]
```

### 数学函数

#### round

```
select round(1.65)
>2 [四舍五入]
select round(-1.65)
>-2 [四舍五入]
select round(1.567,2)
> 1.57 [小数点保留几位]
```

#### ceil

返回>=该参数的最小整数

```
select ceil(1.52)
> 2[向上取整]
select ceil(-1.02)
> -1
```

#### floor

向下取整，返回<=该参数的最大整数

```
select floor(9.99)
> 9
select floor(-9.99)
> -10
```

#### truncate

截断

```
select truncate(1.69,1)
> 1.6
```

#### mod

取余

```
select mod(10,3)
> 1
select mod(-10,3)
> -1
select mod(10,-3)
> 1 [符合和被除数一致，自己想想]
```

### 其他函数

select version(); 查看版本号

select database(); 查看当前数据库

select user(); 查看当前用户

### 流程函数

#### case..when IF

```mysql
CASE <单值表达式>
    WHEN <表达式值> THEN <SQL语句或返回值>
    WHEN <表达式值> THEN <SQL语句或返回值>
   ...
    WHEN <表达式值> THEN <SQL语句或返回值>
    ELSE <SQL语句或返回值>
END

SELECT *,
       (CASE sex  # 注意此处 sex
            WHEN '1' THEN '男'
            WHEN '0' THEN '女'
            ELSE '保密'
        END) AS sex_text
FROM USER

SELECT *,
       (CASE
            WHEN sex='1' THEN '男'
            WHEN sex='0' THEN '女'
            ELSE '保密'
        END) AS sex_text
FROM USER
```

```mysql
# 找25岁以上以下用户数量
select age_cut,count(device_id)as number 
from(Select if(age>=25,'25岁及以上','25岁以下' )as age_cut,device_id 
From user_profile)u2
Group by age_cut

# 注意没有标点
# case 字段名 when 值1 then 值2 else 值3 end
# case when 字段名 运算符 值1 then 值2 else 值3 end
select (case when age>=25 then '25岁及以上' else '25岁以下' end) as age_cut, 
count(device_id) as number
from user_profile
group by age_cut

SELECT 
    case 
        when age >= 25 then '25岁及以上'
        else '25岁以下'
    end as age_cut,
    count(device_id) as number
FROM user_profile
GROUP by age_cut;

# if(字段名 运算符 值1,值2,值3)
SELECT 
IF(age>=25,'25岁及以上','25岁以下') AS age_cut, 
COUNT(device_id) AS number 
FROM user_profile 
GROUP BY age_cut 

select '25岁以下' as age_cut,count(device_id) as number
from user_profile
where age<25 or age is null
union all
select '25岁及以上' as age_cut,count(device_id) as number
from user_profile
where age>=25;

select device_id, gender,(case when age < 20 then '20岁以下'
       when age <25 then '20-24岁'
       when age >= 25 then '25岁及以上'
       else '其他' end) as age_cut
from user_profile;
```

### 日期函数

#### now

```
select now();
> 返回当前系统日期+时间
```

#### curdate

```
select curdate();
> 返回当前系统日期，不包含时间
```

#### curtime

```
select curtime();
> 返回当前时间，不包含日期
```

#### 获取指定部分

```
select year(date)/month(date)/monthname(date)
```

#### str_to_date

将日期格式的字符转换成指定格式的日期

```
select str_to_date('1999-3-2','%Y-%c-%d')
select * from employ where hiredate = str_to_date('4-3 1992','%c-%d %Y')
```

#### date_format

将日期转换成字符

```
select date_format(now(),'%y年%m月%d日')
```



#### DAYOFWEEK(date)

返回日期date的星期索引(1=星期天，2=星期一, ……7=星期六)。这些索引值对应于ODBC标准。

```
select DAYOFWEEK('1998-02-03')
```

-> 3

#### WEEKDAY(date)

返回date的星期索引(0=星期一，1=星期二, ……6= 星期天)。

```
mysql> select WEEKDAY('1997-10-04 22:23:00');
```

-> 5

#### DAYOFMONTH(date)

返回date的月份中日期，在1到31范围内。

```
mysql> select DAYOFMONTH('1998-02-03');
```

-> 3

#### DAYOFYEAR(date)

返回date在一年中的日数, 在1到366范围内。

```
mysql> select DAYOFYEAR('1998-02-03');
```

-> 34

#### MONTH(date)

返回date的月份，范围1到12。

```
mysql> select MONTH('1998-02-03');
```

-> 2

#### DAYNAME(date)

返回date的星期名字。

```
mysql> select DAYNAME("1998-02-05");
```

-> 'Thursday'

#### MONTHNAME(date)

返回date的月份名字。

```
mysql> select MONTHNAME("1998-02-05");
```

-> 'February'

#### QUARTER(date)

返回date一年中的季度，范围1到4。

```
mysql> select QUARTER('98-04-01');
```

-> 2

#### WEEK(date)

对于星期天是一周的第一天的地方，有一个单个参数，返回date的周数，范围在0到52。

```
mysql> select WEEK('1998-02-20');
```

-> 7

#### WEEK(date,first)

2个参数形式WEEK()允许你指定星期是否开始于星期天或星期一。
如果第二个参数是0，星期从星期天开始，
如果第二个参数是1，从星期一开始。

```
mysql> select WEEK('1998-02-20',0);
```

-> 7

```
mysql> select WEEK('1998-02-20',1);
```

-> 8

#### YEAR(date)

返回date的年份，范围在1000到9999。

```
mysql> select YEAR('98-02-03');
```

-> 1998

#### HOUR(time)

返回time的小时，范围是0到23。

```
mysql> select HOUR('10:05:03');
```

-> 10

#### MINUTE(time)

返回time的分钟，范围是0到59。

```
mysql> select MINUTE('98-02-03 10:05:03');
```

-> 5

#### SECOND(time)

回来time的秒数，范围是0到59。

```
mysql> select SECOND('10:05:03');
```

-> 3

#### PERIOD_ADD(P,N)

增加N个月到阶段P(以格式YYMM或YYYYMM)。以格式YYYYMM返回值。注意阶段参数P不是日期值。

```
mysql> select PERIOD_ADD(9801,2);
```

-> 199803

#### PERIOD_DIFF(P1,P2)

返回在时期P1和P2之间月数，P1和P2应该以格式YYMM或YYYYMM。注意，时期参数P1和P2不是日期值。

```
mysql> select PERIOD_DIFF(9802,199703);
```

-> 11

### DAY

```mysql
#计算出2021年8月每天用户练习题目的数量
select
DAY(date) as day,
count(question_id) as question_cnt
from question_practice_detail
where 
SUBSTR(date,1,7) = '2021-08'
group by day

select
day(date)as day,
count(question_id)
from question_practice_detail
where date like '2021-08-%'
group by day

Select day(date) as day, count(question_id) as question_cnt
From question_practice_detail
Where year(date)=2021 and month(date)=08
Group by day
# date_format函数的date_format(date, "%Y-%m")="202108"

```

### 平均次日留存率[hard]

#### lead

#### date_add

date_add(date,interval 值 type)

```
向日期添加指定的时间间隔
例如：date_add('2021-06-01',interval 2 day)，向日期函数2021-06-01加了2天，返回值2021-06-03
```

#### data_sub

date_sub(date,interval 值 type)

- 从日期减去指定的时间间隔
- type参数与date_add相同，使用方法也相同

#### datediff

- datediff(date1,date2)

- 【解释】

  - 返回两个日期之间的天数，date1-date2

  - date1 和 date2 代表是日期或日期加时间的表达式

  - 注意只有值的日期部分参与计算

  - 例如：datediff('2021-06-08','2021-06-01')返回7，datediff('2021-06-01','2021-06-08')返回-7

```mysql
#在某天刷题后第二天还会再来刷题的平均概率
SELECT COUNT(b.device_id)/ COUNT(a.device_id)
FROM(
    SELECT DISTINCT device_id,date
    FROM question_practice_detail ) as a
LEFT JOIN (
    SELECT DISTINCT device_id,DATE_SUB(date,interval 1 day) as date
    FROM question_practice_detail ) as b
ON a.date=b.date AND a.device_id=b.device_id

先用左连接，可以得到第一天回答但是第二天没有回答的记录，然后通过count，并且通过和device_id和日期进行去重
select count(distinct b.device_id,b.date)/count(distinct a.device_id,a.date) as avg_ret
from question_practice_detail a left JOIN question_practice_detail b
on a.date = b.date-1 and a.device_id = b.device_id


```

#### 前后函数

用途：返回位于当前行的前n行（LAG(expr,n)）或后n行（LEAD(expr,n)）的expr的值

```mysql
#查询前1名同学的成绩和当前同学成绩的差值
mysql> SELECT stu_id, lesson_id, score, pre_score,
    -> score-pre_score AS diff
    -> FROM(
    ->     SELECT stu_id, lesson_id, score,
    ->     LAG(score,1) OVER w AS pre_score
    ->     FROM t_score
    ->     WHERE lesson_id IN ('L001','L002')
    ->     WINDOW w AS (PARTITION BY lesson_id ORDER BY score)) t
    -> ;
```



### 文本函数

#### concat

- concat(字段名1,字段名2,'字符串')

- 【解释】

  - concat是字符串拼接函数，用于将多个字符串连接在一起

  - 例如：concat('123',' ','ABC')返回数据123 ABC

#### left right

- left(字符串,n)

- right(字符串,n)

- 【解释】

  - left函数返回字符串从左到右n个字符

  - right函数返回字符串从右到左n个字符

  - 例如：left('123456',2)返回12，right('123456',3)返回456

#### substring

- substring(字符串,x,y)

- 【解释】

  - substring函数用来截取字符串从x位开始取y位，如果没有y则取从x位开始到最后一位

  - 例如：substring('New York',3,4)返回w Yo，substring('New York',3)返回w York

#### 根据文本统计性别人数

```mysql
select if(substring(profile,15)='male','male','female') as gender,
count(device_id)
from user_submit
group by gender

select substring_index(profile,',',-1) as gender,count(device_id)
from user_submit
group by gender

SELECT IF(u.profile LIKE '%female','female','male'),COUNT(1)
FROM user_submit u
GROUP BY IF(u.profile LIKE '%female','female','male');

SELECT 
CASE 
WHEN profile like '%,male' THEN 'male'
WHEN profile like '%,female' THEN 'female'
END
AS gender
, COUNT(*) as number
FROM user_submit
GROUP BY gender;
```

count

> count(*)包括了所有的列，相当于行数，在统计结果的时候，不会忽略列值为NULL
>
> count(1)包括了忽略所有列，用1代表代码行，在统计结果的时候，不会忽略列值为NULL
>
> count(列名)只包括列名那一列，在统计结果的时候，会忽略列值为空（这里的空不是只空字符串或者0，而是表示null）的计数，即某个字段值为NULL时，不统计

LOCATE(substr , str )

>  返回子串 substr 在字符串 str 中第一次出现的位置，如果字符substr在字符串str中不存在，则返回0；

POSITION(substr IN str )

> 返回子串 substr 在字符串 str 中第一次出现的位置，如果字符substr在字符串str中不存在，与LOCATE函数作用相同；

SUBSTRING_INDEX(str ,substr ,n)

> 返回字符substr在str中第n次出现位置之前的字符串;

REPLACE(str, n, m)

> 将字符串str中的n字符替换成m字符；

LENGTH(str) 

> 计算字符串str的长度。

#### 截取出年龄

```mysql
#profile: 180cm,75kg,27,male
#易错点：SUBSTRING_INDEX(profile,',',-2)只是27，male
select (case substring(profile,12,2)
       when '27' then 27
       when '26' then 26
       when '25' then 25
       when '23' then 23
       else 22 end) as age, count(1)
       from user_submit
       group by age;
       
select
SUBSTRING_INDEX(SUBSTRING_INDEX(profile,',',-2),',',1) as age,
count(device_id) as number
From user_submit
Group by age

```

## 窗口函数

- 【标准语法】

  - 窗口函数 over (partition by 用于分组的字段名 order by 用于排序的字段名 asc|desc)
    - over()中两个子句为可选项
  - 常用窗口函数
    - 排序窗口函数
      - rank()over()
      - dense_rank()over()
      - row_number()over()
    - 偏移分析函数
      - lag(字段名,偏移量|, 默认值) over()
      - lead(字段名,偏移量|, 默认值) over()
    - ![img](https://api2.mubu.com/v3/document_image/38650b76-a784-46fc-a545-be5cc2483302-9404487.jpg)

- 知识点讲解
  1. 窗口函数只能写在select字句中
  2. 窗口函数中的partition by子句可以指定数据的分区，和group by要去重分组不同的是，partition by 只分区不去重
  3. 窗口函数中没有partition by子句时，即不对数据分区，直接整个表示一个区
  4. 排序窗口函数中order by子句是必选项，窗口函数中order by子句在分区内，依据指定字段和排序方法对数据行排序
  5. rank(), dense_rank(), row_number()指定排序赋值方法，对比三个排序窗口函数的异同，对【99,99,90,89】进行排序
     1. rank(): 1,1,3,4
     2. dense_rank(): 1,1, 2,3
     3. row_number():1,2,3,4
     4. 根据对排序值的需求选择相应的排序窗口函数，由于值的不同特性，比如数值不重复，这三个函数可以通用
  6. 偏移分析窗口函数中order by 子句是必选项
  7. lag()和lead()指定偏移方向，lag向上偏移，行向上取数据，lead是向下偏移，行向下取数据
  8. lag(字段名，偏移量，（默认值）)over()还有一个参数为默认值，是可选项，在分区中没有前一行的情况下填充默认值，不填的情况下默认是null

### 开窗函数

开窗函数的窗口选项（frame_clause）

```mysql
{ ROWS | RANGE } frame_start
{ ROWS | RANGE } BETWEEN frame_start AND frame_end
```

有以下几种写法

```mysql
-- dt升序排序，向前6行
partition by tag order by dt rows 6 preceding 
partition by tag order by dt rows between 6 preceding and CURRENT row
-- dt降序排列，当前行往后6行
partition by tag order by dt desc rows between  CURRENT row and 6 following
```

### 每个学校GPA最低的同学

#### row_number

```mysql
Select device_id,
university,
gpa
From (
Select device_id,
university,
gpa,
row_number() over(partition by university order by gpa) as rk
From user_profile
)a
where rk = 1
```

#### min()

```mysql
select distinct device_id,university,min(gpa)over(partition by university ) as gpa from user_profile order by university
```



#### 相关子查询

不用group by 也能做！
利用相关子查询，把每个学校的最低gpa当作查询条件，去找出每个学校的gpa最低的同学。因为每个学校只有一个gpa最低的同学，所以最后出来的结果不需要再用group by，用order by排序就好。看代码！！
在子查询中，我们利用到了主查询的表，WHERE university = u.university 这个条件使得mysql去主表得每一行进行查询，比如第一行是2138的北京大学的同学，那么子查询会找出所有北京大学的同学，并且找出其中最低得gpa，如果他是最低的那个就留下，不是就下一个。以此类推找出所有大学的最低gpa的同学，最后排序得最终结果。[【中字】SQL进阶教程 | 史上最易懂SQL教程！](https://www.bilibili.com/video/BV1UE41147KC?p=51)

【一定要看视频，看完就懂了！】

```mysql
SELECT
    device_id,
    university,
    gpa
FROM user_profile u
WHERE gpa = 
    (SELECT MIN(gpa)
     FROM user_profile
     WHERE university = u.university)
ORDER BY university
```



1.每个学校，所以需要按学校分组 group by university

2.最低使用min函数，直接用min查出来的device_id和gpa对应不上。想到用子查询，先分组学校查出每个学校的最低gpa，再自联结根据条件查出对应数据行，注意on的条件里device_id没什么用，因为子程序里的device_id和min（gpa）不对应

3.按要求排序学校返回

```mysql
Select a.device_id,a.university,a.gpa
from user_profile a join 
(select university,min(gpa) as gpa 
from user_profile group by university) b
on a.university=b.university and a.gpa=b.gpa 
order by a.university

```

```mysql
#这逻辑值得多探讨
SELECT device_id, university, gpa
FROM user_profile
WHERE gpa IN( # IN 可替换为 = ANY
    SELECT min(gpa)
    FROM user_profile
    GROUP BY university
)
GROUP BY university # 保证学校名不重复
ORDER BY university; # 保证与题目要求输出顺序一致
```

## 一些常见条件

1. where 后面可以跟 (select xxx) 但是不能对 子查询命名
2. from 后面跟着子查询表一定要命名

#### 常用窗口函数介绍

一定要看：https://blog.csdn.net/weixin_39010770/article/details/87862407

近30天，date_sub(current_date,interval 29 day)

第二天，t1.dt=date_sub(t2.dt,INTERVAL ``1` `day)

join：完之后列数是不会变化的，即=表1的列 | 表2的列，不会让相同的列合并！

凡是统计第n天内的问题，要求包含当天，利用DATE_SUB前推日期时要用【T- (n-1)】

> 例子：
>
> 要求统计近7天内，区间为【当天前推6天，当天】
>
> 要求统计近14天内，区间为【当天前推13天，当天】
>
> 要求统计近30天内，区间为【当天前推29天，当天】

join 表2 on 1就是这个条件总是成立，这里需要全连接，所以可以这么用，当然也可以写on 1=1

#### 播放率

```mysql
round(avg(if(timestampdiff(second,start_time,end_time)>=duration,duration,timestampdiff(second,start_time,end_time))/duration*100),2) avg_play_progress
```

#### 有交互视频的最近一个月

```mysql
WHERE DATEDIFF(DATE((select max(start_time) FROM tb_user_video_log)), DATE(a.start_time)) <= 29
```

#### 每个月截止当前的粉丝量

不用在partition by里面加month 是因为不用分组了 这里就已经按照month累加 然后第二个sum其实就是每个月的粉丝数量

```mysql
sum(SUM(case when if_follow=1 then 1
    when if_follow=2 then -1
   else 0
   end)) over(partition by author order by DATE_FORMAT(DATE(end_time),'%Y-%m'))
total_fans
FROM tb_user_video_log JOIN tb_video_info USING(video_id)
WHERE YEAR(end_time)=2021
GROUP BY author,month
```

#### 每天的近一周总点赞量和一周内最大单天转发量

```mysql
select 
tag,
dt, 
sum(like_cnt) over w sum_like_cnt_7d,
max(retweet_cnt) over w sum_retweet_cnt_7d
from t
window w as (partition by tag order by dt desc ROWS BETWEEN CURRENT ROW AND 6 FOLLOWING)
```

#### 近一个月发布的视频

```mysql
SELECT MAX(DATE(end_time)) as cur_date FROM tb_user_video_log -- 当天
GROUP BY video_id
HAVING TIMESTAMPDIFF(DAY, release_date, cur_date) < 30
```

#### 最近无播放天数

```mysql
MAX(DATE(end_time)) as recently_end_date,  -- 最近被播放日期
MAX(cur_date) as cur_date  -- 非分组列，加MAX避免语法错误
TIMESTAMPDIFF(DAY, recently_end_date, cur_date)
```

#### 计算瞬时的最大计数（在看人数）

首先，我们自然会想到常见的编码+联立。在此对原表in_time和out_time进行编码，in为观看人数+1， out为观看人数-1，进行两次SELECT联立，并按artical_id升序，时间戳升序：

```mysql
代码：

    SELECT 
      artical_id, in_time dt, 1 diff
    FROM tb_user_log
    WHERE artical_id != 0
    UNION ALL
    SELECT 
      artical_id, out_time dt, -1 diff
    FROM tb_user_log
    WHERE artical_id != 0
    ORDER BY 1,2
    
结果：

9001|2021-11-01 10:00:00|1
9001|2021-11-01 10:00:01|1
9001|2021-11-01 10:00:09|1
9001|2021-11-01 10:00:11|-1
9001|2021-11-01 10:00:28|1
9001|2021-11-01 10:00:38|-1
9001|2021-11-01 10:00:51|1
9001|2021-11-01 10:00:58|-1
9001|2021-11-01 10:00:59|-1
9001|2021-11-01 10:01:50|-1
9002|2021-11-01 11:00:45|1
9002|2021-11-01 11:00:55|1
9002|2021-11-01 11:01:11|-1
9002|2021-11-01 11:01:24|-1

意义：

某篇文章artical_id，在给定的时间戳dt的，瞬时观看人数变化diff
```

获取瞬时计数的关键：` SUM(diff) OVER(PARTITION BY artical_id ORDER BY dt) instant_viewer_cnt`

```mysql
代码：

  SELECT
    artical_id,
    dt,
    SUM(diff) OVER(PARTITION BY artical_id ORDER BY dt) instant_viewer_cnt
  FROM (
    SELECT 
      artical_id, in_time dt, 1 diff
    FROM tb_user_log
    WHERE artical_id != 0
    UNION ALL
    SELECT 
      artical_id, out_time dt, -1 diff
    FROM tb_user_log
    WHERE artical_id != 0) t1 
 
结果：
 
9001|2021-11-01 10:00:00|1
9001|2021-11-01 10:00:01|2
9001|2021-11-01 10:00:09|3
9001|2021-11-01 10:00:11|2
9001|2021-11-01 10:00:28|3
9001|2021-11-01 10:00:38|2
9001|2021-11-01 10:00:51|3
9001|2021-11-01 10:00:58|2
9001|2021-11-01 10:00:59|1
9001|2021-11-01 10:01:50|0
9002|2021-11-01 11:00:45|1
9002|2021-11-01 11:00:55|2
9002|2021-11-01 11:01:11|1
9002|2021-11-01 11:01:24|0

意义：

某篇文章artical_id，在给定的时间戳dt的，瞬时累计观看人数instant_viewer_cnt
```

#### 次日留存率

```mysql
select 
    a.dt,
    ifnull(round(count(distinct b.uid)/count(a.uid),2),0)uv_left_rate
from
    # 每个用户最初注册的日期
    (SELECT uid,date(min(in_time)) dt
    FROM tb_user_log
    GROUP BY uid) a 
# 把日期拆分成登陆和退出日期，再通过uid联立表
# 查找每个用户注册日期增加1天后的日期是否在登陆和退出日期当中
left join
    (
    SELECT uid,DATE(in_time) dt
    FROM tb_user_log
    union
    SELECT uid,DATE(out_time) dt
    FROM tb_user_log
    ) b 
ON a.uid = b.uid
# 每个uid在初次登陆日期的第二天
and b.dt = DATE_ADD(a.dt,INTERVAL 1 DAY)
# WHERE的时机，在全部表联合完之后进行筛选
WHERE a.dt like "2021-11%"
GROUP BY a.dt
ORDER BY a.dt;
```

```mysql
-- solution 1
SELECT ROUND(COUNT(DISTINCT user_id) * 1.0 / (SELECT COUNT(DISTINCT user_id) FROM login), 3) as p
FROM login
WHERE (user_id, date) in (
  select user_id, DATE_ADD(min(date), INTERVAL 1 DAY)
  from login
  GROUP by user_id
)

-- solution 2
with t1 as (select 
user_id,
date(min(date)) as dt
from login
group by user_id)

select 
round(count(distinct b.user_id)/count(t1.user_id),3) as p
from t1 left join login b on t1.user_id = b.user_id and b.date = date_add(t1.dt,interval 1 day)
```



#### 日活数及日新用户

```mysql
select a.dt,count(distinct a.uid) dau,round(count(distinct b.uid)/count(distinct a.uid),2)
from 
        (select uid,date(in_time) dt
        from tb_user_log
        union all
        select uid,date(out_time) dt
        from tb_user_log
        group by uid,dt
         ) a
left join
         (select uid,min(date(in_time)) dt
          from tb_user_log
          group by uid
         ) b
on a.uid=b.uid and a.dt=b.dt
group by a.dt
order by a.dt;
```

```mysql
select date,count(uid) as dau,
    round( sum(if(times=1,1,0))/count(uid) ,2)
from
(select *,count(*) over (partition by uid order by date ) as times
from
(select uid,left(in_time,10) as date
from tb_user_log
union
SELECT uid,left(out_time,10) as date
from tb_user_log) tmp) base
GROUP by date
order by date
```

#### 连续天数

1. 推到这里那其实思路已经清晰了，求**连续签到的天数**，那无非就是**连续问题**了
   1. 连续问题核心就是利用**排序编号与签到日期的差值是相等的**。因为如果是连续的话，编号也是自增1，日期也是自增1。
   2. 如图，***dt***是签到日期，***dt_tmp***是编号和签到日期的差值。可以发现 编号 8 是断了连续签到的，所以***dt_tmp***与前面的不相同
   3. 那么再以dt_tmp和 uid 来分组，再***dense_rank*** 一次，就可以获得连续签到的天数了。那么问题就解决了。

```mysql
# 连续签到的日期序列 与  1,2,3,4,...  这样的等差数列作差值。  
#  每一组连续序列得到的差值相同。 
# 签到日期  序列    差值    连续签到次数  分数
# 20210902   1    20210901      1      1
# 20210903   2    20210901      2      1
# 20210905   3    20210902      1      1
# 20210906   4    20210902      2      1
# 20210907   5    20210902      3      3
# 20210908   6    20210902      4      1
# 20210909   7    20210902      5      1
# 20210910   8    20210902      6      1
# 20210911   9    20210902      7      7
#.....      
#再按照差值分组, 那么每一组都是一个连续签到的组。
#然后跟根据连续签到次数给分，最后按月求和

WITH t1 AS( -- t1表筛选出活动期间内的数据，并且为了防止一天有多次签到活动，distinct 去重
	SELECT
		DISTINCT uid,
		DATE(in_time) dt,
   		# 注意使用dense_rank
		DENSE_RANK() over(PARTITION BY uid ORDER BY DATE(in_time)) rn -- 编号
	FROM
		tb_user_log
	WHERE
		DATE(in_time) BETWEEN '2021-07-07' AND '2021-10-31' AND artical_id = 0 AND sign_in = 1
),
t2 AS (
	SELECT
	*,
	DATE_SUB(dt,INTERVAL rn day) dt_tmp, 
    # 这里用第二个日期和uid分区 用第一个日期排序
    # 注意使用dense_rank
	case DENSE_RANK() over(PARTITION BY DATE_SUB(dt,INTERVAL rn day),uid ORDER BY dt )%7 -- 再次编号
		WHEN 3 THEN 3
		WHEN 0 THEN 7
		ELSE 1
	END as day_coin -- 用户当天签到时应该获得的金币数
	FROM
	t1
)
	SELECT
		uid,DATE_FORMAT(dt,'%Y%m') `month`, sum(day_coin) coin  -- 总金币数
	FROM
		t2
	GROUP BY
		uid,DATE_FORMAT(dt,'%Y%m')
	ORDER BY
		DATE_FORMAT(dt,'%Y%m'),uid;
```

```mysql
select uid,DATE_FORMAT(dt,'%Y%m') month ,sum(grade) from
     (select uid ,dt,  
     case 
     when   mod( rank() over  (partition by uid,rank_day order by dt),7) =3 then 3     
     when   mod( rank() over (partition by uid,rank_day order by dt) ,7)=0 then 7 
     else 1   end   grade    
     from 
        (
        select
        uid,
        dt,
        date_sub(dt,INTERVAL RANK() over(PARTITION by uid order by dt) day )  rank_day  from
            (
            select DISTINCT uid, date(in_time) dt  #查出所有签到记录 过滤掉可能重复的记录
            from tb_user_log 
            where artical_id=0 and sign_in=1 and date(in_time) BETWEEN '2021-07-07' and '2021-10-31' 
            ) t1
        ) t2
     )t3
group by uid,month;
```

#### GMV

GMV为已付款订单和未付款订单两者之和

#### 每个日期下 901商店的 在售商品总数

```mysql
with t as # 取日期 10-01 到 10-03
(select date(event_time) dt
 from tb_order_overall 
 where date(event_time) between '2021-10-01' and '2021-10-03'),
 
 t1 AS # 901商店已成交订单中，每个下单日期里的 product_id
 (select date(event_time) dt,tod.product_id
 FROM tb_order_detail tod JOIN tb_order_overall too ON tod.order_id = too.order_id and status = 1
                          JOIN tb_product_info tpi ON tod.product_id = tpi.product_id and shop_id = '901'),
                          
t2 as # 计算 每个日期下 901商店的 在售商品总数
(select date(event_time) dt,count(distinct case when datediff(date(event_time),date(release_time)) >= 0  then product_id end ) sum_product
 FROM tb_product_info,tb_order_overall
 where shop_id = '901'
 group by dt)
 
select t.dt, round(count(distinct t1.product_id) / sum_product ,3) as sale_rate, 
             ROUND(1- (count(distinct t1.product_id) / sum_product ),3) as unsale_rate
from t left join t1 ON datediff(t.dt,t1.dt) between 0 and 6 
            JOIN t2 on t.dt = t2.dt
group by t.dt
ORDER by t.dt 
```

#### 追加汇总信息

- 找出2021年10月有取消订单的司机：`WHERE DATE_FORMAT(order_time, "%Y-%m")='2021-10' AND ISNULL(fare)`
- 筛选他们的已完成订单的评分：WHERE driver_id in (...) AND NOT ISNULL(grade)
- 按司机分组：GROUP BY driver_id
- 追加汇总信息：WITH ROLLUP
- 输出每个司机的平均评分：
  - 司机ID或总体：IFNULL(driver_id, "总体") as driver_id
  - 平均评分：AVG(grade) as avg_grade
  - 保留1位小数：ROUND(x, 1)

Mysql中有一个with rollup是用来在分组统计数据的基础上再进行统计汇总，即用来得到group by的汇总信息；

IFNULL() 函数用于判断第一个表达式是否为 NULL，如果为 NULL 则返回第二个参数的值，如果不为 NULL 则返回第一个参数的值。

IFNULL() 函数语法格式为：

```
IFNULL(expression, alt_value)
```

```mysql
SELECT IFNULL(driver_id, "总体") as driver_id,
    ROUND(AVG(grade), 1) as avg_grade
FROM tb_get_car_order
WHERE driver_id in (
    SELECT driver_id
    FROM tb_get_car_order
    WHERE DATE_FORMAT(order_time, "%Y-%m")='2021-10' AND ISNULL(fare)
) AND NOT ISNULL(grade)
GROUP BY driver_id
WITH ROLLUP;

```

#### 工作日

1. 筛选条件：周一到周五 -- WHERE DATE_FORMAT(event_time,'%W') NOT IN ( 'Saturday' ,'Sunday')

#### 时间段

DATE_FORMAT(event_time,'%H-%i-%s') >= '07-00-00'

#### 时间需要精确到小时分钟秒

```mysql
date_format(a.in_datetime,'%H:%i:%s') <= '19:00:00'
date_format(a.out_datetime,'%H:%i:%s') >= '19:00:00'

right(a.in_datetime,8) <= '19:00:00'
right(a.out_datetime,8) >= '19:00:00'

time(a.in_datetime) <= '19:00:00'
time(a.out_datetime) >= '19:00:00'
```



#### 特殊字符

```
select concat(round(sa_p/tag_p*100,2),'%') as "discount_rate(%)"
```

#### 电商

##### SKU和SPU

【SPU】

Standard Product Unit （标准产品单位），是商品信息聚合的最小单位。通俗点来讲，就是产品的款式/型号。例子：手机 -> 苹果手机 -> iPhone 13 Pro Max。

iPhone 13 Pro Max就是一个SPU。

【SKU】

Stock Keeping Unit (库存量单位)。在很多业务场景下，是计算库存进出计量，物理上不可分割的最小存货单元，表示某款商品的具体配置（规格、颜色等）。例子：iPhone 13 Pro Max 远峰蓝 128G。

##### 动销率

【动销率，pin_rate】

题目：有销售的SKU数量 / 在售SKU数量

解释：

有销售的SKU数量 --- 售出的SKU数量总和（商品期间销售数量）

在售的SKU数量 --- 剩余（总库存 - 售出）库存SKU的数量总和（商品期末库存数量）

例如：某店iPhone 13 Pro Max商品库存为1000台。一个月内卖出400台（有销售的SKU数量），剩余600台（在售的SKU数量）。动销率为 400/600 * 100 ≈ 66.7%

注意：

- 上面是动销率的一种计算方式，关注数量，即：售出 / 库存 ；
- 还有一种计算关注种类， 即：有销售记录的产品种类 / 所有库存产品种类；
- 很多情况下 （1- 动销率）就是滞销率。

##### 售罄率

题目：GMV / 备货值（备货值 = 吊牌价 * 库存数）

解释：

GMV --- 所有订单的金额总和（很多场景下，下单未付款的订单金额也计算在内）

吊牌价 --- 商品详情页显示的价格

> 例子：【iPhone 13 Pro Max 远峰蓝 128G】在拼夕夕，标价 8999，这个8999就是吊牌价（又称韭菜价格/忽悠价格/期望值管理价格）。如果我最后下单价格 6999，那么下单价 / 吊牌价，就是折扣率。在本例中，在拼夕夕下单一台丐版13香的折扣率为 6999 / 8999 ≈ 77.8%（做梦）

#### 倒数第三

```mysql
SELECT * 
FROM employees
WHERE hire_date = (
    SELECT DISTINCT hire_date     -- DISTINCT 很重要！防止出现  排名相同 然后拿到的不是倒数第三
    FROM employees
    ORDER BY hire_date DESC       -- 倒序
    LIMIT 1 OFFSET 2              -- 去掉排名倒数第一第二的时间，取倒数第三
);    
```

```mysql
# 这个答案只适合查日期不相同的数据。如果入职日期相同，查询到的结果是经过系统默认排序后的倒数第三项，结果只显示这一条，会有误差。
以下的两种方式均表示取2,3,4三条条数据。
1.select* from test LIMIT 1,3;
当limit后面跟两个参数的时候，第一个数表示要跳过的数量，后一位表示要取的数量。

2.select * from test LIMIT 3 OFFSET 1;(在mysql 5以后支持这种写法)
当 limit和offset组合使用的时候，limit后面只能有一个参数，表示要取的的数量,offset表示要跳过的数量 。
```

考虑到工资第一多第二多的员工都有可能有多个，所以需要将其按照工资分组再排序

```mysql
SELECT emp_no, salary
FROM salaries
WHERE salary = (SELECT salary
                FROM salaries
                WHERE to_date = '9999-01-01'
                GROUP BY salary
                ORDER BY salary DESC LIMIT 1,1 )  
```

#### 不能用order获取排名第二

```mysql
-- 方法一
select s.emp_no, s.salary, e.last_name, e.first_name
from salaries s join employees e
on s.emp_no = e.emp_no
where s.salary =              -- 第三步: 将第二高工资作为查询条件
    (
    select max(salary)        -- 第二步: 查出除了原表最高工资以外的最高工资(第二高工资)
    from salaries
    where salary <    
        (
        select max(salary)    -- 第一步: 查出原表最高工资
        from salaries
        where to_date = '9999-01-01'   
        )
    and to_date = '9999-01-01'
    )
and s.to_date = '9999-01-01'



-- 方法二
select s.emp_no, s.salary, e.last_name, e.first_name
from salaries s join employees e
on s.emp_no = e.emp_no
where s.salary =
    (
    select s1.salary
    from salaries s1 join salaries s2      -- 自连接查询
    on s1.salary <= s2.salary
    group by s1.salary                     -- 当s1<=s2链接并以s1.salary分组时一个s1会对应多个s2
    having count(distinct s2.salary) = 2   -- (去重之后的数量就是对应的名次)
    and s1.to_date = '9999-01-01'
    and s2.to_date = '9999-01-01'
    )
and s.to_date = '9999-01-01'
```

#### 查找字符串中逗号出现的次数

mysql中查找字符串中某个符号或字符出现的次数
例：查找字符串’10,A,B’ 中逗号’,'出现的次数？
用到的函数：
length(s)函数： s是字符串， 返回的是所求的字符串s的长度。

replace(a,b,c)： 在字符串a中，将a中出现的b，替换成c。再把这个替换之后的串的结果返回。

```mysql
select id,length(string)-length(replace(string,",","")) from strings;
```

#### 字符串最后两个字符

```mysql
SELECT first_name FROM employees ORDER BY substr(first_name,length(first_name)-1) 
SELECT first_name FROM employees ORDER BY substr(first_name,-2) 
right(first_name,2)
```

#### 同一个部门的emp_no按照逗号进行连接

```mysql
group_concat( [distinct] 要连接的字段 [order by 排序字段 asc/desc ] [separator '分隔符'] )
当数据太大，group_concat超出了默认值1024，超过就会截断，group_concat查询出来的数据就会不全。

select dept_no, group_concat(emp_no) as employees
from dept_emp
group by dept_no
```

#### 分页查询 跟倒数第三一个套路

**LIMIT 语句结构：** LIMIT X,Y 

- Y ：返回几条记录
- X：从第几条记录开始返回（第一条记录序号为0，默认为0）

```mysql
SELECT *
FROM employees
LIMIT 5,5

SELECT * FROM employees LIMIT 5 OFFSET 5
```

#### Exist使用

exists 与 not exists 的用法：

1、该子查询如果“有数据结果”(至少返回一行数据)， 则该EXISTS() 的结果为“true”，外层查询执行

2、该子查询如果“没有数据结果”（没有任何数据返回），则该EXISTS()的结果为“false”，外层查询不执行

3、EXISTS后面的子查询不返回任何实际数据，只返回真或假，当返回真时 where条件成立

注意：EXISTS关键字，比IN关键字的运算效率高，因此，在实际开发中，特别是大数据量时，推荐使用EXISTS关键字

1.什么时候用EXISTS，什么时候用IN？
    主表为employees，从表为dept_emp，在主表和从表都对关联的列emp_no建立索引的前提下：
      当主表比从表大时，IN查询的效率较高；
      当从表比主表大时，EXISTS查询的效率较高；
    原因如下：
      in是先执行子查询，得到一个结果集，将结果集代入外层谓词条件执行主查询，子查询只需要执行一次
      exists是先从主查询中取得一条数据，再代入到子查询中，执行一次子查询，判断子查询是否能返回结果，主查询有多少条数据，子查询就要执行多少次

#### 前N个累积和

两个内连接

```mysql
把所有小于等于当前编号的表s1和当前编号表s2联立起来，然后按照当前编号分组，计算出所有小于等于
当前标号的工资总数
SELECT s2.emp_no,s2.salary,SUM(s1.salary) AS running_total
FROM salaries AS s1 INNER JOIN salaries AS s2 
ON s1.emp_no <= s2.emp_no
WHERE 
s1.to_date = "9999-01-01"
AND s2.to_date = "9999-01-01"
GROUP BY s2.emp_no
```

窗口函数

```mysql
select emp_no,
salary,
sum(salary)over(order by emp_no) as running_total
from salaries
where to_date = '9999-01-01'
```

#### 中位数

```mysql
SELECT job, 
    floor(( count(*) + 1 )/ 2 ) AS "start", 
    floor(( count(*) + 2 )/ 2 ) AS 'end' 
FROM grade 
GROUP BY job 
ORDER BY job
```

法1：当某一数的正序和逆序累计均大于整个序列的数字个数的一半即为中位数

```
比如:
A A B B C C D D 
1 2 3  4  5 6  7 8
8 7 6  5  4  3 2 1
那么上面的4，5以及5，4就是中位数，如果是奇数的话，就只有1个
再比如
A2个，B3个，C5个，D2个，
正序2，5，10，12
倒序12，10，7，2
正序和12，大于等于6的，为C,D，
逆序和为12，大于等于6的为ABC，所以最后中位数为C
```

```mysql
select grade from (select grade,(select sum(number) from class_grade) as total,
        sum(number) over(order by grade) a,
        sum(number) over(order by grade desc) b
        from class_grade) t1
where a >= total/2 and b >=total/2
order by grade;
```

法2：1. 求出每个等级的开始名次和结束名次

```mysql
select grade
      ,number
      ,sum(number) over(order by grade)-number+1 as left_order
      ,sum(number) over(order by grade)          as right_order
  from class_grade
 order by grade
```

2. 求出中位数所在的名次

   ```mysql
   select floor((sum(number)+1)/2) as mid
     from class_grade
   union 
   select ceil((sum(number)+1)/2) as mid
     from class_grade
   ```

3. 找到中位数对应的等级

   直接用`mid between`介于开始名次`left_order`和结束名词`right_order`之间就能找到对应的等级。
   注意中位数可能有两个，恰好在同一个等级里面，所以这里要加上`distinct`关键词去重。

   ```mysql
   -- 第1张表
   with t1 as 
   (
   select grade
         ,number
         ,sum(number) over(order by grade)-number+1 as left_order
         ,sum(number) over(order by grade)          as right_order
     from class_grade
    order by grade
   )
   -- 第2张表
   , t2 as 
   (
   select floor((sum(number)+1)/2) as mid
     from class_grade
   union 
   select ceil((sum(number)+1)/2) as mid
     from class_grade
   )
   -- 3. 找到中位数对应的等级
   select distinct grade
     from t1,t2
    where t2.mid between left_order and right_order
   ```

   我自己改的

   ```mysql
   with t1 as (select grade
         ,number
         ,sum(number) over(order by grade)-number+1 as left_order
         ,sum(number) over(order by grade)          as right_order
     from class_grade
    order by grade),
   
   t2 as (select 
    floor(( sum(number) + 1 )/ 2 ) AS "n1", 
   floor(( sum(number) + 2 )/ 2 ) AS 'n2' 
   from class_grade)
   
   select distinct grade
   from t1, t2
   where n1 between left_order and right_order or n2 between left_order and right_order
   ```

   

#### 同月不同年

```mysql
-- 因date日期类型经过 DATE_FORMAT()后变成 字符串，所以使用right()函数取后两位即为月数
ON h1.job=h2.job AND  right(first_year_mon,2)=right(second_year_mon,2)
```

#### 写代码当天

```mysql
where create_time = current_date
```

## 综合处理

### SQL34 统计复旦用户8月练题情况

- 我自己写的，不知道哪里错了
  - 1 注意要用Left join
  - 2 date is null 表示引入日期为空
  - 3 group by 
  - 4 university 在group by里面没出现 在where里面没出现 能不能放在select 里面
  - 5 我count输出的结果会有是null

```mysql
select t1.device_id, university, question_cnt,right_question_cnt
from
(select u.device_id, university, count(*) question_cnt
from user_profile u left join question_practice_detail q on u.device_id = q.device_id
where university = '复旦大学' and (date like '2021-08-%' or date is null)
group by u.device_id) as t1 left join
(select u2.device_id, count(result) as right_question_cnt
from user_profile u2 left join question_practice_detail q2
on u2.device_id = q2.device_id
where (date like '2021-08-%' or date is null) and result = 'right'
group by device_id) as t2 on t1.device_id = t2.device_id;
```

#### Solution 1

**问题分解：**

- 限定条件：需要是复旦大学的（来自表user_profile.university），8月份练习情况（来自表question_practice_detail.date）
- 从date中取month：用month函数即可；
- 总题目：count(question_id)
- 正确的题目数：`sum(if(qpd.result='right', 1, 0))`
- 按列聚合：需要输出每个用户的统计结果，因此加上`group by up.device_id`

------

**细节问题：**

- 8月份没有答题的用户输出形式：题目要求『对于在8月份没有练习过的用户，答题数结果返回0』因此明确使用left join即可，即输出up表中复旦大学的所有用户，如果8月没有练习记录，输出0就好了
- 老样子-表头：as语法重命名后两列就好

**完整代码：**

```mysql
select up.device_id, '复旦大学',
    count(question_id) as question_cnt,
    sum(if(qpd.result='right', 1, 0)) as right_question_cnt
from user_profile as up

left join question_practice_detail as qpd
  on qpd.device_id = up.device_id and month(qpd.date) = 8

where up.university = '复旦大学'
group by up.device_id

```

#### Solution 2

需要判断月份
通过IF语句可以设置值
用SUM统计所有值

```mysql
SELECT up.device_id,up.university,SUM(IF(MONTH(date)=8,1,0)) AS question_cnt,SUM(IF(MONTH(date)=8 AND result='right',1,0)) AS right_question_cnt
FROM user_profile up
LEFT JOIN question_practice_detail qpd
ON up.device_id = qpd.device_id
WHERE university = '复旦大学'
GROUP BY up.device_id
```

#### Solution 3

解题思路

- 一看到这种题还是比较懵的，又是要分组，又是要判断，又要连接，还要进行日期判断
- 这时候第一步是要仔细读题，先分析可能出现的子句，和要用到的语句。
- 题目有要求统计没有答过题的用户，所有我们要以user_profile为主表来操作
- 对于本题来说，实现两表的连接是比较容易实现的，
- 然后是选择要使用的字段，与本题有关的字段包括，user_profile 的device_id,university ，和 question_practice_detail 的 question_id、result 和 date
- 先选好五个字段，然后对五个字段进行操作

**分析过程**

```
1、用where字段 操作 WHERE t1.university = '复旦大学'
2、时间方面要注意 从题意“8月没有练习过的用户”可知没有答过题的也要统计，
3、但是没有答过题的在左连接下question_id为null 所以要加上 
	OR t2.date IS NULL
4、result字段是字符型的，题目要求统计回答正确的题数，直接计数肯定不行
	所以用case 或者 if 函数转换一下，然后用求和函数统计，可以一并把null和
    wrong值转换成0值
5、接下来就是统计题目数，和对答题结果求和
6、最后根据用户分一下组
```

**代码**

```mysql
SELECT t1.device_id, t1.university, 
count(t2.question_id) AS question_cnt, 
sum(
case
when t2.result = 'right' then 1
ELSE 0
end
) AS right_question_cnt

FROM user_profile AS t1
LEFT JOIN question_practice_detail AS t2
ON t1.device_id = t2.device_id
WHERE t1.university = '复旦大学' AND ((MONTH(t2.date)=8 OR t2.date IS NULL ))
GROUP BY device_id
;
```

#### Solution 4

根据前面的进行修改

group by是有去重和剔除null的作用，所有这里的分组依据一定要用第一张表，第二张表有null，如果用第二张表就会出现数据缺失的。

```mysql
select u.device_id, university, 
count(question_id) as question_cnt,
sum(if(result='right',1,0)) as right_question_cnt
from user_profile u left JOIN question_practice_detail q
on u.device_id = q.device_id
where u.university = '复旦大学' and (date like '2021-08-%' or date is null)
group by u.device_id;
```

### SQL35 浙大不同难度题目的正确率

#### Solution 1 笛卡尔积

我自己写的

```mysql
select difficult_level, sum(if(result='right',1,0))/count(1) as correct_rate
from user_profile u, question_practice_detail p, question_detail q
where u.device_id = p.device_id and p.question_id = q.question_id and university='浙江大学'
group by difficult_level
order by correct_rate;
```

### SQL35 返回每个顾客不同订单的总金额

根据实际情况，可以知道顾客和订单号是一对多的关系，那么，同一订单号应当是对应一位顾客ID，故可以先在OrderItems表中按订单号分组用SUM()函数计算出同一订单的总金额:

```mysql
SELECT order_num, SUM(item_price*quantity) AS total_ordered
FROM OrderItems
GROUP BY order_num
```

可以根据以上用WITH...AS创建新表total_order_price，然后通过键order_num将其与Orders表相连，完整代码如下：

```mysql
WITH total_order_price AS (
    SELECT order_num, SUM(item_price*quantity) AS total_ordered
    FROM OrderItems
    GROUP BY order_num
)
SELECT cust_id, total_ordered
FROM Orders 
    LEFT JOIN 
    total_order_price
    USING(order_num)
ORDER BY total_ordered DESC
```

### SQL38 返回顾客名称和相关订单号以及每个订单的总价

#### Solution 1

我的

忽略了group by 可以对多列分组！

```mysql
select cust_name, o2.order_num, OrderTotal from 
(select cust_id, sum(quantity*item_price) OrderTotal
from Orders o join OrderItems t 
on o.order_num = t.order_num
group by cust_id) t1, Customers c, Orders o2
where t1.cust_id = c.cust_id and c.cust_id = o2.cust_id
order by cust_name, o2.order_num;
```

#### Solution 2

order by 1 2 说明按照第 1 2列排序

```mysql
select
    cs.cust_name,
    os.order_num,
    sum( oi.quantity * oi.item_price ) as OrderTotal
from Customers cs,
     Orders os,
     OrderItems oi
where cs.cust_id = os.cust_id
and os.order_num = oi.order_num

group by cs.cust_name, os.order_num

order by 1, 2
```

### SQL49 组合 Products 表中的产品名称和 Customers 表中的顾客名称

#### Solution 1

我的

```mysql
select t.*  【prod_name这里不能重命名否则出错】from (select prod_name
from Products
union all
select cust_name
from Customers) as t
order by prod_name
```

#### Solution 2

- union--连接表，对行操作。
- union--将两个表做行拼接，同时自动删除重复的行。
- union all---将两个表做行拼接，保留重复的行

```mysql
select prod_name from Products
union 
select cust_name as prod_name from Customers
order by prod_name
```

### SQL18 月总刷题数和日均刷题数

- 看了半天才明白,大概是因为day(last_day(submit_time运算结果还是跟submit_time同样的一串数列，只有加上avg(),min()或max()运算才变成了一个数值

```mysql
select DATE_FORMAT(submit_time,'%Y%m') submit_month,
count(question_id) month_q_cnt,
ROUND(count(question_id)/avg(day(LAST_DAY(submit_time))),3) avg_day_q_cnt 
from practice_record
where DATE_FORMAT(submit_time,'%Y')='2021'
group by submit_month
union ALL
SELECT '2021汇总' as submit_month,
count(question_id) month_q_cnt,
round(count(id)/31,3) avg_day_q_cnt
from practice_record
where DATE_FORMAT(submit_time,'%Y')='2021'
order by submit_month
```

- 我修改后的

  ```mysql
  select date_format(submit_time, '%Y%m') submit_month, 
          count(submit_time) month_q_cnt, 
          round(count(submit_time)/avg(day(last_day(submit_time))),3) avg_day_q_cnt
  from practice_record
  where year(submit_time)=2021 and submit_time is not NULL
  group by date_format(submit_time, '%Y%m')
  union all
  select '2021汇总' submit_month,
  count(submit_time) month_q_cnt,
  round(count(submit_time)/31,3) avg_day_q_cnt
  from practice_record
  where year(submit_time)=2021 and submit_time is not null
  order by submit_month
  ```

### SQL19 未完成试卷数大于1的有效用户[hard]

- 对于一个人（组内）的多条作答，用;连接去重后的作答记录：`group_concat(distinct concat_ws(':', date(start_time), tag) SEPARATOR ';')`

#### concat_ws

CONCAT_WS(separator,str1,str2,…)

CONCAT_WS() 代表 CONCAT With Separator ，是CONCAT()的特殊形式。 第一个参数是其它参数的分隔符。分隔符的位置放在要连接的两个字符串之间。分隔符可以是一个字符串，也可以是其它参数。如果分隔符为 NULL，则结果为 NULL。函数会忽略任何分隔符参数后的 NULL 值。

#### group_concat

GROUP_CONCAT函数返回一个字符串结果，该结果由分组中的值连接组合而成。

GROUP_CONCAT([DISTINCT] expr [,expr ...]
[ORDER BY {unsigned_integer | col_name | formula} [ASC | DESC] [,col ...]]
[SEPARATOR str_val])

```mysql
SELECT uid, count(incomplete) as incomplete_cnt,
    count(complete) as complete_cnt,
    group_concat(distinct concat_ws(':', date(start_time), tag) SEPARATOR ';') as detail
from (
    SELECT uid, tag, start_time,
        if(submit_time is null, 1, null) as incomplete,
        if(submit_time is null, null, 1) as complete
    from exam_record 
    left join examination_info using(exam_id)
    where year(start_time)=2021
) as exam_complete_rec
group by uid
having complete_cnt >= 1 and incomplete_cnt BETWEEN 2 and 4
order by incomplete_cnt DESC

```

下面是我自己写的

- 在sum(if())里面不能用score，因为有些没有subtime却有score……

```mysql
select uid, sum(if(submit_time is null, 1,0)) incomplete_cnt, 
sum(if(submit_time is not null, 1,0)) complete_cnt,
group_concat(distinct concat_ws(':',date(start_time),tag) separator ';') as detail
from  exam_record e left join examination_info i 
on e.exam_id = i.exam_id
where year(start_time)=2021
group by uid
having incomplete_cnt < 5 and complete_cnt >= 1 and incomplete_cnt > 1
order by incomplete_cnt DESC
```

### SQL20 月均完成试卷数不小于3的用户爱作答的类别

```mysql
# 对月均完成试卷理解不清楚 以为是每个月完成试卷>=3 实际上只要求平均数>=3即可
select tag, count(tag) tag_cnt
from exam_record r left join examination_info i
on r.exam_id = i.exam_id
where uid in (
select uid  from exam_record
 where submit_time is not null 
 group by uid
 having count(exam_id)/count(distinct date_format(submit_time,'%Y%m')) >= 3
)
group by tag
order by tag_cnt desc
```

### SQL23 每个题目和每份试卷被作答的人数和次数

#### union排序问题

```mysql
SELECT * FROM
(SELECT exam_id AS tid, COUNT(DISTINCT exam_record.uid) uv,
COUNT(*) pv FROM exam_record 
GROUP BY exam_id
ORDER BY uv DESC, pv DESC) t1

UNION 

SELECT * FROM
(SELECT question_id AS tid, COUNT(DISTINCT practice_record.uid) uv,
COUNT(*) pv FROM practice_record 
GROUP BY question_id
ORDER BY uv DESC, pv DESC) t2;
```

知识点是UNION后的排序问题，ORDER BY子句只能在最后一次使用。 如果想要在UNION之前分别单独排序，那么需要这样：

```mysql
SELECT * FROM
( SELECT * FROM t1  ORDER BY 字段 ) newt1 ## 一定要对表重新命名，否则报错 
UNION
SELECT * FROM
( SELECT * FROM t2  ORDER BY 字段 ) newt2

```

法二：用tid字段的左边第一个数来排序。

```mysql
SELECT exam_id AS tid, COUNT(DISTINCT exam_record.uid) uv,
COUNT(*) pv FROM exam_record 
GROUP BY exam_id

UNION 

SELECT question_id AS tid, COUNT(DISTINCT practice_record.uid) uv,
COUNT(*) pv FROM practice_record 
GROUP BY question_id
 
ORDER BY LEFT(tid,1) DESC, uv DESC, pv DESC;
```

### SQL25 满足条件的用户的试卷完成数和题目练习数

- 记得使用left join
- with 新建表名 as (查询语句)

```mysql
with users as (
select uid from user_info u
left join exam_record er using(uid)
left join examination_info ei using(exam_id)
where tag='SQL' and difficulty = 'hard' and `level`=7
group by u.uid
having avg(er.score)>80    
)

select u.uid, count(distinct er.id) exam_cnt, count(distinct pr.id) question_cnt
from users u
left join exam_record er on u.uid = er.uid and year(submit_time)=2021
left join practice_record pr on u.uid = pr.uid and year(pr.submit_time)=2021
group by u.uid
order by exam_cnt, question_cnt desc
```

### SQL26 每个6/7级用户活跃情况

- 涉及新建表，引入标签
- union all
- 用户活跃的定义
  - 试卷以开始答题时间作为活跃时间
  - 题目以提交时间作为活跃时间
- 直接看题解吧

#### 找不出bug

下面是我自己写的，实在不知道错哪里了

```mysql
# 试卷活跃时间
# select uid, 
# date(start_time) as ac_day,
# month(start_time) as ac_mon,
# 'exam' as tag
# from exam_record er
# where year(start_time)=2021;

# # 题目活跃时间
# select uid, 
# date(submit_time) as ac_day,
# month(submit_time) as ac_mon
# 'ques' as tag
# from practice_record pr
# where year(start_time)=2021;

# 将上面两个连接一起
select 
ui.uid,
count(distinct(ac_mon)) act_month_total,
count(distinct(ac_day)) act_days_2021,
count(distinct(if(tag='exam',ac_day,null))) act_days_2021_exam,
count(distinct(if(tag='ques',ac_day,null))) act_days_2021_question
from user_info ui
left join 
(
select uid, 
date(start_time) as ac_day,
month(start_time) as ac_mon,
'exam' as tag
from exam_record er
where year(start_time)=2021
union all
select uid, 
date(submit_time) as ac_day,
month(submit_time) as ac_mon,
'ques' as tag
from practice_record pr
where year(submit_time)=2021
) as exam_ques

on ui.uid = exam_ques.uid
where `level`>=6
group by ui.uid
order by act_month_total desc, act_days_2021 desc;
```

就算把上面的month(start_time)

改成date_format(start_time,'%Y%m') 也没用

### SQL28 第二快/慢用时之差大于试卷时长一半的试卷

我写的，又不知道错在哪……

```mysql
# 先找出第二快
with fa as (select exam_id, 
row_number()over(partition by exam_id order by timestampdiff(minute, start_time, submit_time))
as ranking,
start_time as st,
submit_time as sm,
timestampdiff(minute, submit_time, start_time) as dur
from exam_record
where ranking = 2)

# 找出第二慢
with sl as (select exam_id, 
row_number()over(partition by exam_id order by timestampdiff(minute, start_time, submit_time) desc)
as ranking,
start_time as st,
submit_time as sm,
timestampdiff(minute, submit_time, start_time) as dur
from exam_record
where ranking = 2)

# 第二快和第二慢时长差
select ei.exam_id, duration, release_time
from examination_info ei 
join(
select exam_id, sl.dur-fa.dur as durdiff
from fa join sl on fa.exam_id = sl.exam_id
) as t2
on ei.exam_id = t2.exam_id
where durdiff>duration/2
order by exam_id desc;
```

搞了半天终于成功

```mysql
# 找出第二慢 快
with sl as (select exam_id, 
row_number()over(partition by exam_id order by timestampdiff(second, start_time, submit_time))
as ranking1,
row_number()over(partition by exam_id order by timestampdiff(second, start_time, submit_time) desc)
as ranking2,
timestampdiff(second, start_time, submit_time) as dur
from exam_record)

# 第二快和第二慢时长差
select ei.exam_id, duration, release_time
from examination_info ei 
join (select a1.exam_id,  (a2.dur-a1.dur)/60 as dursub
 from sl a1  join sl a2 on a1.exam_id = a2.exam_id and a2.ranking2=2
 where a1.ranking1=2 ) as t2
on ei.exam_id = t2.exam_id
where dursub >= duration/2
order by exam_id desc;
```

### SQL31 未完成率较高的50%用户近三个月答卷情况

我的，不知道错哪了

```mysql
#  exam_record er left join user_info ui on er.uid = ui.uid and `level`>=6
select uid, 
date_format(start_time, '%Y%m') as start_month,
       count(start_time) as tatol_cnt,
       count(score) as complete_cnt
       
from(
select uid, start_time, score,
    dense_rank()over(partition by uid order by date_format(start_time, '%Y%m') desc) as recent_months
from exam_record
) recent_table
where recent_months <=3 and uid in
(select t11.uid from
    (select uid, 
    incom_rate,
    row_number()over(partition by uid order by incom_rate desc) as ranking
    from 
        (
        select er.uid,
        -- count(if(submit_time is null,1,null)) incomplte,
        -- count(start_time) total_,
        sum(if(submit_time is null,1,0))/count(start_time) incom_rate
        from exam_record er  left join examination_info ei 
        on er.exam_id = ei.exam_id and tag='SQL'
        group by uid
        ) as t1
     ) as t11
    join (
        select count(distinct uid) as total_user -- 不加on相当于直接拼在后面，加了on需要选择相同的连接项连接
                 from exam_record
        ) as t3 
    left join user_info ui 
    on t11.uid = ui.uid
  where `level`>=6 and ranking <= ceiling(total_user/2)
) 
group by uid, start_month
order by uid
```

### SQL33 对试卷得分做min-max归一化

我的，有误，没找出来错哪里，但是有几个知道要注意的点：

1. 判断 max = min?
2. 标准化的时候*100

```mysql
#1 先获取高难度试卷的Max min
with exam_score as (select er.exam_id,
max(score) as top,
min(score) as bot
from exam_record er  join examination_info ei on er.exam_id = ei.exam_id
where difficulty='hard' and score is not null
group by exam_id
)

#2 将分数转换为标准分
# select uid, 
# exam_record.exam_id,
# if(bot=top, score,round((score-bot)/(top-bot)*100)) as new_score
# from exam_record join exam_score es on exam_record.exam_id = es.exam_id

#3 计算平均分
select uid, 
t.exam_id,
round(avg(new_score)) as avg_new_score
from 
(
select uid, 
exam_record.exam_id,
if(bot=top, score,(score-bot)/(top-bot)*100) as new_score
from exam_record join exam_score es on exam_record.exam_id = es.exam_id
) as t
group by uid, exam_id
order by exam_id, avg_new_score desc
```

### SQL34 每份试卷每月作答数和截止当月的作答总数。

聚合窗口函数中，over()的括号中有order by 时，即为计算到当前时间为止的累计数量

```mysql
select distinct exam_id
,date_format(start_time,'%Y%m') as start_month
,count(start_time)over(partition by exam_id,date_format(start_time,'%Y%m')) as month_cnt
,count(start_time)over(partition by exam_id order by date_format(start_time,'%Y%m')) as cum_exam_cnt
from exam_record
order by exam_id,start_month

```

### SQL40 根据指定记录是否存在输出不同情况

- 未完成数要用：COUNT(start_time) - COUNT(submit_time)。要不然有些用户根本没做题，不能用if 没做题也判断成1。

### SQL6 近一个月发布的视频中热度最高的top3视频

```mysql
SELECT video_id,
    ROUND((100 * comp_play_rate + 5 * like_cnt + 3 * comment_cnt + 2 * retweet_cnt)
        / (TIMESTAMPDIFF(DAY, recently_end_date, cur_date) + 1), 0) as hot_index
FROM (
    SELECT video_id,
        AVG(IF(
            TIMESTAMPDIFF(SECOND, start_time, end_time)>=duration, 1, 0
        )) as comp_play_rate,
        SUM(if_like) as like_cnt,
        COUNT(comment_id) as comment_cnt,
        SUM(if_retweet) as retweet_cnt,
        MAX(DATE(end_time)) as recently_end_date,  -- 最近被播放日期
        MAX(DATE(release_time)) as release_date,  -- 发布日期
        MAX(cur_date) as cur_date  -- 非分组列，加MAX避免语法错误{这里好绝！！}
    FROM tb_user_video_log
    JOIN tb_video_info USING(video_id)
    LEFT JOIN (
        SELECT MAX(DATE(end_time)) as cur_date FROM tb_user_video_log
    ) as t_max_date ON 1
    GROUP BY video_id
    HAVING TIMESTAMPDIFF(DAY, release_date, cur_date) < 30
) as t_video_info
ORDER BY hot_index DESC
LIMIT 3;

```

