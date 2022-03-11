# MySQL

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

### 字符函数

#### length

```
select length('aaa'); 
> 3
select length('张三') 
> 9 [utf-8 一个中文3个字节]
```

#### concat

拼接字符串

```
select concat(last_name,'_',first_name)
from employee
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

  - 窗口函数 over (partition by 用于分组的字段名 order by 用于排序的字段名)

  - 常用窗口函数
    - ![img](https://api2.mubu.com/v3/document_image/38650b76-a784-46fc-a545-be5cc2483302-9404487.jpg)

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

