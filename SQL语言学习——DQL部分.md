SQL语言学习——DQL部分
------

——作者：Su

- MySQL服务的启动与停止
  - 在【计算机管理】中可以查看【服务】，右键启动或停止
  - 管理员身份打开CMD
    - 停止，如：`net stop mysql80`
    - 启动，如：`net start mysql80`
    - `mysql80`为服务名，在安装MySQL时会设置此名称，Service Name
- 在路径：C:\ProgramData\MySQL\MySQL Server 8.0下可以看到【my.ini】文件，打开后可以在【datadir】项（约96行）设置数据的存储位置，以及更改其他设置。**更改完成后需要重启服务**。
- 环境变量：C:\Program Files\MySQL\MySQL Server 8.0\bin;
- CMD登录MySQL命令语句
  - `mysql -h -P -u -p`
  - 如：`mysql -h localhost -P 3306 -u root -proot`
- SQL语言包括DQL, DML, DDL, DCL, TCL
  - DQL：Data Query Language，数据库查询语言，SELECT等
  - DDL：Data  Definition Language ，数据库定义语言，CREATE、DROP等
  - DML：Data  Manipulation Language，数据操纵语言，INSERT、UPDATE、DELETE等
  - DCL：Data  Control Language，数据库控制语言，授权、角色控制等，GRANT、REVOKE等
  - TCL：Transaction Control Language，事务控制语言，设置保存点，回滚等，SAVEPOINT、ROLLBACK等

1. 查看所有数据库

	- show datebases;

2. 打开指定的数据库

	- use 库名;

3. 查看当前库中的所有表

	- show tables;

4. 查看其它数据库中的表

	- show tables from 库名；

5. 创建表

	- create table 表名(
列名 列类型，
列名 列类型，
。。。
);
6. 查看表结构

   - desc 表名;

7. 查看服务器的版本

   方式一：登录到MySQL

   - select version();

   方式二：没有登陆到MySQL

   - mysql --version
   - mysql --V
   

## SQL的句法规范

### 1. 不区分大小写，但建议关键字大写，表名、列名小写

- SELECT 

- FROM

- .....

- 执行顺序

  ```mysql
  from
  where
  group by
  having
  select
  order by
  ```

### 2. 每条命令最好用分号结尾

### 3. 每条命令根据需要，可以进行缩进或换行

### 4. 注释

   1. 单行注释：#注释文字
   2. 单行注释：-- 注释文字（注意空格）
   3. 多行注释： /* 注释文字 */

##  进阶1：基础查询

```mysql
SELECT
	【查询列表】
FROM
	【表名】;
```
### 1. 查询列表可以是：表中字段、常量值、表达式、函数

### 2. 查询结果是一个虚拟的表格

```mysql
SELECT
	last_name,
	salary,
	email
FROM
	emlpoyee;
```
或

`SELECT * FROM employee;`

### 3. 查询常量值

`SELECT 100;`

`SELECT 'Jhon';`

### 4. 查询表达式

`SELECT 100*98;`

### 5. 查询版本函数
`SELECT VERSION();`
### 6. 起别名
- 便于理解
- 如果要查询的字段有重名的情况，使用别名可以区分开来

1. 方式一：使用AS

     `SELECT 100*98 AS 结果;`

     `SELECT last_name AS 姓,first_name AS 名 FROM employee;`

2. 方式二：使用空格

     `SELECT last_name 姓,first_name 名 FROM employee;`	

- **实例**：

  查询salary，显示结果为 out put

  `SELECT salary AS "out put" FROM employee;`
  
  **双引号将别名括起来，否则可能会报错，因为out为关键字**

### 7. 去重

- 使用关键字 **DISTINCT**

  `SELECT DISTINCT department_id FROM employees;`

### 8. +号的作用

- 运算符
1. 两个操作数都为数值型，则作加法运算

    `SELECT 100+90;`

2. 只要其中一个操作数为字符型，试图将字符型数值转换成数值型

  - 如果转换成功，则继续加法运算，如： 

    `SELECT '123'+90;`

  - 如果转换失败，则将字符型数值转换为0，如：

    `SELECT 'john'+90;`

  - 只要其中一方为null，则结果肯定为null，如：

    `SELECT null+90;`

  - **CONCAT()函数**

    将几个字段合成为一个字段显示在查询结果

    **`SELECT CONCAT('a','b','c') AS 结果`**

- **实例**：

  查询员工【姓】和【名】，连接成一个字段，并显示为【姓名】
  ```mysql
  SELECT
  	CONCAT(last_name,first_name) AS 姓名
  FROM
  	employees;
  ```
  显示出来`employees`的全部列，各个列之间用**逗号**连接，列头显示成OUTPUT

  ```mysql
  SELECT 
  	CONCAT( first_name,',',last_name,',',job_id) AS OUTPUT
  FROM
  	employees;
  ```

## 进阶2：条件查询

```mysql
SELECT
    查询列表
FROM
    表名
WHERE
    【筛选条件】；
```

###  1. 按照条件表达式筛选

- 简单条件运算符
  - \> 大于

  - < 小于

  - = 等于

  - <> 不等于

  - \>= 大于等于

  - <= 小于等于

    

- 实例
  查询部门编号不等于90的员工名和部门编号

  ```mysql
  SELECT
    last_name,
    department_id
  FROM
    employees
  WHERE
    department_id<>90;
  ```

### 2. 按照逻辑表达式筛选

- 逻辑运算符
  - &&， AND， 与

  - ||， OR， 或

  - !， NOT， 非

- 案例

  查询部门编号不是在90到110之间，或者工资高于15000的员工信息

  ```mysql
  SELECT
  	*
  FROM
  	employees
  WHERE
  	NOT(deparment_id>=90 AND deparment_id<=110) OR salary>15000;
  ```

### 3. 模糊查询

- 如：
  - like
  - between and
  - in
  - is null

#### 1）LIKE

- `LIKE`语句前面没有`IS`

- 可以使用`NOT LIKE`语句

- 实例1：LIKE

  查询员工名中包含字符a的员工信息

  ```mysql
  SELECT
  	*
  FROM
  	employees
  WHERE
  	last_name LIKE '%a%';
  ```

  **`'%a%'`中的`%`是通配符，类似于word中的【*】**

  LIKE '%% '可以查询该列中**<u>不为</u>**NULL的全部值

  

- 实例2：LIKE

  查询员工名中第三个字符为e，第五个字符为a的员工名和工资

  ```mysql
  SELECT
  	last_name,
  	salary
  FROM
  	employees
  WHERE
  	last_name LIKE '__e_a%';
  ```
  **`'__e_a%'`中的`_`是通配符，类似于word中的【?】**

  可以判断数值型或字符型，如`LIKE '1__'`即查询百位数字为1的三位数

  

- 实例3：LIKE

  查询员工名中第二个字符为_的员工名

  ```mysql
  SELECT
  	last_name
  FROM
  	employees
  WHERE
  	last_name LIKE '_$_%' ESCAPE '$';
  ```

  `ESCAPE`子句使得`$`为转义字符，`'_$_%'`中的第二个`_`不表示通配符

#### 2）BETWEEN AND

- 实例4：BETWEEN AND

  查询部门编号在90到120之间的员工信息

  ```mysql
  SELECT
  	*
  FROM
  	employees
  WHERE
  	employee_id BETWEEN 90 AND 120;
  ```

  **BETWEEN 【A】 AND 【B】，即大于<u>等于</u>A小于<u>等于</u>B**

- 查询部门编号**不**在90到120之间的员工信息

  ```mysql
  SELECT
  	*
  FROM
  	employees
  WHERE
  	employee_id NOT BETWEEN 90 AND 120;
  ```

#### 3）IN

- 实例5：IN

  查询员工的工种编号是IT_PROG、AD_VP、AD_PRES中的一种的员工名和工种编号

  ```mysql
  SELECT
  	last_name,
  	job_id
  FROM
  	employees
  WHERE
  	job_id='IT_PROG' OR job_id='AD_VP' OR job_id='AD_PRES';
  ```

  等价于

  ```mysql
  SELECT
  	last_name,
  	job_id
  FROM
  	employees
  WHERE
  	job_id IN ('IT_PROG','AD_VP','AD_PRES');
  ```

#### 4）IS NULL

- 实例6：IS NULL

  ```mysql
  SELECT
  	last_name,
  	job_id
  FROM
  	employees
  WHERE
  	job_id IS NULL;
  ```

  ```mysql
  SELECT
  	last_name,
  	job_id
  FROM
  	employees
  WHERE
  	job_id IS NOT NULL;
  ```

  **=或<>不能用于判断NULL值**

#### 5）安全等于<=>

- 实例7：安全等于<=>

  既可以判断NULL值，也可以判断普通数值

#### 6）IFNULL

- 实例8：IFNULL

  查询员工号为176的员工的姓名和部门号的年薪

  ```mysql
  SELECT
  	last_name,
  	department_id,
  	salary*12*(1+IFNULL(commission_pct,0)) AS 年薪
  FROM
  	employees
  WHERE
  	employee_id=176;
  ```

## 进阶3：排序查询

```mysql
SELECT
    查询列表
FROM
    表名
WHERE
    【筛选条件】
ORDER BY 排序列表 【ASC | DESC】
```

asc为升序（可省略），desc为降序

- 实例1

  查询员工信息，要求工资从高到低排序

  ```mysql
  SELECT
  	*
  FROM
  	employees
  ORDER BY
  	salary
  DESC;
  ```

- 实例2

  查询部门编号大于90的员工信息，按入职时间的先后进行排序

  ```mysql
  SELECT
  	*
  FROM
  	employees
  WHERE
  	department_id>=90
  ORDER BY
  	hiredate ASC;
  ```

- 实例3：按表达式排序

  按年薪的高低显示员工的信息和年薪

  ```mysql
  SELECT
  	*,
  	salary*12*(1+IFNULL(commission_pct,0)) AS 年薪
  FROM
  	employees
  ORDER BY
  	salary*12*(1+IFNULL(commission_pct,0)) DESC;
  ```

- 实例4：按别名排序

  ```mysql
  SELECT
  	*,
  	salary*12*(1+IFNULL(commission_pct,0)) AS 年薪
  FROM
  	employees
  ORDER BY
  	年薪 DESC;
  ```

- 实例5：按函数排序

  ```mysql
  SELECT
  	LENGTH(last_name) 字节长度,
  	last_name,
  	salary
  FROM
  	employees
  ORDER BY
  	LENGTH(last_name);
  ```

- 实例6：按多个字段排序

  ```mysql
  SELECT
  	*
  FROM
  	employees
  ORDER BY
  	salary ASC,
  	employee_id DESC;
  ```

## 进阶4：常见函数

### 1. 字符函数

#### 1）LENGTH

获取参数值的字节个数

#### 2）CONCAT

拼接字符串

`SELECT CONCAT(last_name,'_',first_name) 姓名 FROM employees;`

#### 3）UPPER、LOWER

转换为大小写

`SELECT CONCAT(UPPER(last_name),LOWER(first_name)) 姓名 FROM employees;`

#### 4）SUBSTR、SUBSTRING

截取字符串

**注意：索引从1开始，不是0开始**

`SELECT SUBSTR('李莫愁爱上了陆展元',7) OUT_PUT；`

OUT_PUT为【陆展元】

`SELECT SUBSTR('李莫愁爱上了陆展元',1,3) OUT_PUT；`

OUT_PUT为【李莫愁】

- 实例：姓名中首字母大写，其他字母小写然后用_拼接，显示出来

  ```mysql
  SELECT 
  	LOWER(SUBSTR(first_name,2)),
  	'_',
  	UPPER(SUBSTR(last_name,1,1)),
  	LOWER(SUBSTR(last_name,2)) AS OUT_PUT
  FROM
  	employees;
  ```

#### 5）INSTR

返回字符串第一次出现的索引，如果找不到返回0

`SELECT INSTR('李莫愁爱上了陆展元','陆展元') OUT_PUT；`

OUT_PUT为7

`SELECT INSTR('李莫愁爱上了陆展元','陆一元') OUT_PUT；`

OUT_PUT为0

#### 6）TRIM

去掉字符串开头结尾的多余字符

`SELECT TRIM('          张翠山         ') AS out_put;`

out_put为【张翠山】

`SELECT TRIM('a' FROM 'aaaaaaaa张aaaaaaaaaaa翠山aaaaaaaaaaaaaa') AS out_put;`

out_put为【张aaaaaaaaaaa翠山】，trim函数只能去掉字符串**开头结尾**的多余字符

#### 7）LPAD

用指定的字符实现左填充至指定长度

`SELECT LPAD('张无忌',7,'a') AS out_put;`

out_put为【aaaa张无忌】

`SELECT LPAD('张无忌',2,'a') AS out_put;`

out_put为【张无】，指定长度小于初始字符串长度，从右面删除原字符串

#### 8）RPAD

用指定的字符实现右填充至指定长度

#### 9）REPLACE

替换

`SELECT REPLACE('周芷若周芷若张无忌爱上了周芷若','周芷若','赵敏') AS out_put;`

out_put为【赵敏赵敏张无忌爱上了赵敏】



### 2. 数学函数

#### 1）round

四舍五入

`select round(-1.55);`

结果：-2

`select round(1.567,2);`

结果：1.57，保留2位小数

#### 2）ceil

向上取整，返回>=参数的最小整数

`select ceil(-1.02);`

结果：-1

#### 3）floor

向下取整，返回<=参数的最大整数

`select floor(-9.99);`

结果：-10

#### 4）truncate

截断

`select truncate(1.6999999,1);`

结果：1.6，小数点后保留1位

#### 5）mod

取余

`select mod(10,3);`

结果：1

`select mod(10,-3);`

结果：1

**简记：mod(a,b)，a为正则结果位正，a为负则结果为负**
$$
mod(a,b) = a-\frac{a}{b}*b
$$



### 3. 日期函数

#### 1）now

返回当前系统日期+时间

`select now();`

#### 2）curdate

返回当前系统日期，不包括时间

`select curdate();`

####  3）curtime

返回当前系统时间，不包括日期

`select curtime();`

#### 4）指定部分，年、月、日、小时、分钟、秒

`select year(now()) 年;`

`select year('1998-1-1');`

`select year(hiredate) from employees;`

`select month(now()) 月;`

`select monthname(now()) 月;` 结果为英文月份名称

#### 5）str_to_date

将日期格式的字符转换成指定格式的日期

`str_to_date('9-3-2019','%m-%d-%Y')`

结果：2019-09-03

`select * from employees where hiredate=str_to_date('4-3 2019','%m-%d %Y');`

#### 6）date_format

将日期转化成字符

`select date_format(now(),'%y年%m月%d日');`

![1555903113074](md image/SQL语言学习——DQL部分/1555903113074-1557281306014.png)

### 4. 其他函数

`select version();`

`select database();`

`select user();`

`select password('字符')` 返回该字符的密码形式，*应该也是一种哈希运算*

`select md5('字符')` 返回该字符的md5加密形式

### 5. 流程控制函数

#### 1）if

`select if(10>5,'真'，'假')；`

结果：真

#### 2）case

- 实例1

  查询员工的工资，要求：

  部门号=30，显示的工资为1.1倍

  部门号=40，显示的工资为1.2倍

  部门号=50，显示的工资为1.3倍

  其他部门，显示的工资为原工资

  ```mysql
  select
  	salary 原始工资，
  	department_id,
  case department_id
  when 30 then salary*1.1
  when 40 then salary*1.2
  when 50 then salary*1.3
  else salary
  end as 新工资
  from employees；
  ```

​	结果为三列，分别为【原始工资】、【department】、【新工资】

- 实例2

  查询员工的工资的情况

  如果工资>20000，显示A

  如果工资>15000，显示B

  如果工资>10000，显示C

  否则，显示D

  ```mysql
  select salary,
  case
  when salary>20000,then 'A'
  when salary>15000,then 'B'
  when salary>10000,then 'C'
  else 'D'
  end as 工资级别
  from employees;
  ```

### 6. 分组函数

sum求和、avg平均值、max最大值、min最小值、count计算个数

  ```mysql
select sum(salary) from employees;
select max(salary) from employees;
select min(salary) from employees;
select avg(salary) from employees;
select count(salary) from employees;
  ```

1. **sum、avg一般用于处理数值型**

2. **max、min、count可以用于处理任何类型**

3. **以上分组函数都忽略null值**

4. **可以和distinct搭配实现去重**

   `select sum(distinct salary),sum(salary) from employees`

5. **count函数的相似介绍**

   查询总行数，一般使用`count(*)`；

   `select count(*) from employees;`

## 进阶5：分组查询

```mysql
select 分组函数，列 -- （要求出现在group by的后边）
from 表
where 【筛选条件】
group by 分组的列
order by
```

查询列表比较特殊，要求是【分组函数】和【`group by`后面出现的字段】

### 1.添加分组前的筛选条件

- 使用where

- 实例1

  查询每个工种的最高工资

  ```mysql
  select max(salary), job_id
  from employees
  group by job_id;
  ```

- 实例2

  查询每个位置上的部门个数

  ```mysql
  select count(*),location_id
  from deparments
  group by location_id;
  ```

- 实例3

  查询邮箱中包含字符a的，每个部门的平均工资

  ```mysql
  select avg(salary),department_id
  from employees
  where email like '%a%'
  group by department_id;
  ```

- 实例4

  查询有奖金的每个领导手下的员工的最高工资

  ```mysql
  select max(salary),manager_id
  from employees
  where commission_pct is not null
  group by manager_id;
  ```

### 2.添加分组后的筛选条件

- 使用`having`连接条件，不可用`where`，因为`where`后面只可接表中字段

- `having`后的筛选条件并不一定要出现在`select`后面

- 实例1

  查询哪个部门的员工个数>2

  ```mysql
  select count(*),department_id
  from employees
  group by department_id
  having count(*)>2;
  ```

- 实例2

  查询每个工种有奖金的员工的最高工资>12000的工种编号和最高工资

  ```mysql
  select max(salary),job_id
  from employees
  group by job_id
  having max(salary)>12000;
  ```

- 实例3

  查询领导编号>102的每个领导手下的最低工资>5000的领导编号是哪个，以及其最低工资

  ```mysql
  select min(salary),manager_id
  from employees
  where manager_id>102
  group by manager_id
  having min(salary)>5000;
  ```

### 3.按表达式或函数分组

- 实例1

  按员工姓名的长度分组，查询每一组的员工个数，筛选员工个数>5的有哪些

  ```mysql
  select count(*) c,length(last_name) len_name
  from employees
  group by len_name
  having c>5;
  ```

  *支持别名*

### 4.按多个字段分组

多个字段之间用逗号隔开，没有顺序要求 

- 实例1

  查询每个部门每个工种的员工的平均工资

  ```mysql
  select avg(salary),department_id,job_id
  from employees
  group by department_id,job_id;
  ```


- 实例2：添加排序

  查询每个部门每个工种的员工的平均工资，并且按平均工资的高低显示

  ```mysql
  select avg(salary),department_id,job_id
  from employees
  where department_id is not null
  group by department_id,job_id
  having avg(salary)>10000
  order by avg(salary) desc;
  ```

## 进阶6：连接查询

用于有外键或多对多的形式

### 1. SQL92语法

**内连接：等值连接、非等值连接、自连接**

支持一部分外连接，用于Oracle、sqlserver

```mysql
select name, boyname from boys, beauty
where beauty.boyfriend_id=boys.id;
```

#### 1）等值连接

1. 多表等值连接的结果为多表的交集部分
2. n表连接，至少需要n-1个连接条件
3. 多表的顺序没有要求（`from`后的表的顺序）
4. 可以为表起别名
5. 可以搭配前面介绍的所有子句，如排序、分组、筛选

- 案例1

  查询员工名和对应的部门名
  ```mysql
  select last_name,department_name
  from employee,departments
  where employee.'department_id' = department.'department.id';
  ```

- 案例2：为表起别名

  查询员工名、工种号、工种名

  ```mysql
  select last_name,e.job_id,job_title  
  -- job_id字段在表【employees】以及【job】中都存在，因此需要限定表名为【e】
  from employees e,job j  -- 表【employees】的别名为e，表【job】的别名为j
  where e.'job_id'=j.'job_id';
  ```

  **注意**：在`from`后为表设置了别名后，在`select`后必须使用表的**别名**

- 案例3：添加筛选

  查询有奖金的员工名、部门名

  ```mysql
  select last_name,department_name
  from employees e,departments d
  where e.department_id=d.department_id
  	and e.commission_pct is not null;
  ```

- 案例4

  查询城市名中第二个字符为o的部门名和城市名

  ```mysql
  select departmemt_name,city
  from departments d,locations l
  where d.location_id=l.location_id
  	and city like '_o%';
  ```

- 案例5：添加分组

  查询每个城市的部门个数

  ```mysql
  select count(*) 个数，city
  from departments d,locations l
  where d.location_id=l.location_id
  group by city;
  ```

- 案例6

  查询有奖金的每个部门的部门名、部门的领导编号和该部门的最低工资

  ```mysql
  select department_name,d.manager_id,min(salary)
  from departments d,employees e
  where e.department_id=d.department_id
  	and commission_pct is not null
  group by department_name,manager_id;
  ```

  *答案如上，但个人感觉最后应为`group by department_name;`,因为题目中为【**部门**的领导编号和**部门**的最低工资】，最低工资是部门的而不是每个领导的*

- 案例7：添加排序

  查询每个工种的工种名和员工的个数，并且按员工个数降序

  ```mysql
  select job_title,count(*)
  from jobs j,employees e
  where e.job_id=j.job_id
  group by job_title
  order by count(*) desc;
  ```

- 案例8：实现三表连接

  查询员工名、部门名和所在的城市

  ```mysql
  select last_name,department_name,city
  from employees e,departments d,locations l
  where e.department_id=d.department.id
  	and d.location_id=l.location_id
  	and city like 's%';
  ```

#### 2）非等值连接

- 案例1

  查询员工的工资和工资级别

  ```mysql
  select salary,grade_level
  from employees e,job_grades g
  where salary between g.lowest_sal and g.highest_sal
  	and g.grade_level='A';
  ```

#### 3）自连接

- 案例1

  查询员工名和上级的名称

  ```mysql
  select e.employee_id,e.last_name,m.employee_id,m.last_name
  from employees e,employees m  -- 同一个表起两个别名
  where e.manager_id=m.employee_id;
  ```

### 2. SQL99语法

内连接：【连接类型】`inner`

- 等值
- 非等值
- 自连接

外连接：

- 左外【连接类型】`left outer join`
- 右外【连接类型】`right outer join`
- 全外 【连接类型】`full outer join`，**MySQL不支持**

​	*以上`outer`均可省略*

交叉连接：【连接类型】`cross`

```mysql
select 查询列表
from 表1 别名1 【连接类型】
join 表2 别名2 on 连接条件
join 表3 别名3 on 连接条件
。。。
where 筛选条件
group by 分组
having 筛选条件
order by 排序列表
```

#### 1）内连接

1. 可以添加排序、分组、筛选
2. `inner`可以省略
3. 筛选条件放在`where`后面，连接条件放在`on`后面，提高分离性，便于阅读

- 等值连接
  - 案例1

    查询员工名、部门名

    ```mysql
    select last_name,department_name
    from employees e
    inner join departments d
    on e.department_id = d.department_id;
    ```

  - 案例2

    查询名字中包含e的员工名和工种名（添加筛选）

    ```mysql
    select last_name,job_title
    from employees e
    inner join jobs j
    on e.job_id = j.job_id
    where e.last_name like '%e%';
    ```

  - 案例3

    查询部门个数>3的城市名和部门个数（添加分组和筛选）

    ```mysql
    select city,count(*) 部门个数
    from locations l
    inner join departments d
    on l.location_id=d.location_id
    group by city
    having count(*)>3;
    ```

  - 案例4

    查询哪个部门的员工个数>3的部门名和员工个数，并按个数降序（添加排序）

    ```mysql
    select department_name,count(*)
    from departments d
    inner join employees e
    on e.department_id=d.department_id
    group by department_name
    having count(*)>3
    order by count(*) desc;
    ```

  - 案例5

    查询员工名、部门名、工种名，并按部门名降序（三表连接）

    ```mysql
    select last_name,department_name,job_title
    from employees e
    inner join departments d on e.department_id=d.department_id
    inner join jobs j on e.job_id=j.job_id
    order by department_name desc;
    ```

- 非等值连接

  - 案例1

  	查询员工工资级别
    
    ```mysql
    select salary,grade_level
    from employees e
    join job_grade g on e.salary between g.lowest_sal and g.highest_sal;
    ```
    
  - 案例2

    查询工资级别的个数>20的工资级别，并且按工资级别降序

    ```mysql
    select count(*),grade_level
    from employees e
    join job_grade g on e.salary between g.lowest_sal and g.highest_sal
    group by grade_level
    having count(*)>20
    order by grade_level desc;
    ```

- 自连接

  - 案例1

    查询员工的名字、上级的名字

    ```mysql
    select e.last_name,m.last_name
    from employees e
    join employees m on e.manager_id=m.employee_id;
    ```

  - 案例2

    查询姓名中包含字符k的员工的名字、上级的名字

    ```mysql
    select e.last_name,m.last_name
    from employees e
    join employees m on e.manager_id=m.employee_id
    where e.last_name like '%k%';
    ```

#### 2）外连接

用于查询一个表中有，另一个表中没有的记录

1. 外连接的查询结果为主表的所有记录 

   - 如果从表中有与其匹配的，则显示匹配的值
   - 如果从表中没有和它匹配的，则显示null
   - 外连接查询结果=内连接结果+主表中有而从表中没有的记录

2. 左外连接：`left join`左边的是主表

   右外链接：`right join`右边的是主表

3. 左外和右外交换两个表的顺序，可以实现同样的效果

4. 全外连接=内连接结构+表1中有表2中没有的+表1中没有表2中有的，**即两表的全部条目都会列出来**

- 案例1：左外连接

  查询男朋友不在男神表中的女神名

  ```mysql
  select b.name,bo.*
  from beauty b
  left outer join boys bo
  on b.boyfriend_id = bo.id
  where bo.id is null;
  ```

  【beauty】是主表，`bo.id`是主键，因为题目是男朋友不在男生表中，所以一定是主键为null

- 案例2

  查询哪个部门没有员工

  ```mysql
  select d.department_name
  from departments d
  left join employees e -- departments是主表
  on e.department_id=d.department_id
  where e.employee_id is null; -- 主键为null
  ```

  `employee_id`是主键，题目中是没有员工，所以一定是主键为null

- 案例3：全外连接，MySQL不支持

  ```mysql
  select b.*,bo.*
  from beauty b
  full join boys bo
  on b.boyfriend_id=bo.id;
  ```

  查询结果会列出两表中所有条目，没有对应项时，使用（Null）填充

#### 3）交叉连接

```mysql
select b.*,bo.*
from beauty b
cross join boys bo;
```

表1有m行，表2有n行，查询结果有m x n行，表1中的每个条目均和表2中的每个条目都有对应

### 3. SQL92 vs SQL99

- 功能：SQL99 支持的多
- 可读性：SQL99实现连接条件和筛选条件的分离，可读性较高

### 4. 各类查询常用代码：

- 内连接

![1558409421957](md image/SQL语言学习——DQL部分/1558409421957.png)

```mysql
select <select_list>
from A
inner join B
on A.key=B.key;
```

- 左外连接

![1558409708263](md image/SQL语言学习——DQL部分/1558409708263.png)

```mysql
select <select_list>
from A
left join B
on A.key=B.key;
```

- 右外连接

![1558409799165](md image/SQL语言学习——DQL部分/1558409799165.png)

```mysql
select <select_list>
from B
right join A
on A.key=B.key;
```

- 左外连接，去掉交集

![1558409892207](md image/SQL语言学习——DQL部分/1558409892207.png)

```mysql
select <select_list>
from A
left join B
on A.key=B.key
where B.key is null;
```

- 右外连接，去掉交集

![1558410104231](md image/SQL语言学习——DQL部分/1558410104231.png)

```mysql
select <select_list>
from A
right join B
on A.key=B.key
where A.key is null;
```

- 全外连接

![1558410219974](md image/SQL语言学习——DQL部分/1558410219974.png)

```mysql
select <select_list>
from A
full join B
on A.key=B.key;
```

- 全外连接，去掉交集

![1558410509721](md image/SQL语言学习——DQL部分/1558410509721.png)

```mysql
select <select_list>
from A
full join B
on A.key=B.key
where A.key is null or B.key is null;
```

### 5. 练习题

- 案例1

  查询编号>3的女神的那朋友信息，如果有则列出详细，如果没有，用null填充

  ```mysql
  select b.last_name, bo.*
  from beauty b
  left join boys bo
  on b.boyfriend_id = bo.id
  where b.id>3;
  ```

- 案例2

  查询哪个城市没有部门

  ```mysql
  select city, d.*
  from locations l
  left join departments d
  on d.location_id = l.location_id
  where d.department_id is null;
  ```

  d.department_id是主键

- 案例3

  查询部门名为SAL或IT的员工信息

  ```mysql
  select e.*,d.department_name
  from employees e
  right join departments d
  on e.department_id=d.department_id
  where d.department_name in ("SAL","IT");
  ```

  可能部门名为SAL或者IT，但是没有员工，因此需要`departments`为主表

  若`employees`为主表，则会出现

## 进阶7：子查询

- 含义：出现在其他语句中`select`语句，称为子查询或内查询

- 分类：
  - 按子查询出现的位置：
    - select后面：仅支持标量子查询
    - from后面：支持表子查询
    - where或having后面：标量子查询、列子查询、行子查询
    - exists后面（相关子查询）：表子查询
  - 按结果集行列数不同：
    - 标量子查询（结果集只有一行一列）
    - 列子查询（结果集一列多行）
    - 行子查询（结果集一行多列）
    - 表子查询（结果集一般为多行多列）

### 1. where或having后面

1. 子查询放在小括号内
2. 子查询一般放在条件右侧
3. 标量子查询，一般搭配着单行操作符使用：> < >= <= = <>
4. 列子查询，一般搭配着多行操作符使用：in、any/some、all

#### 1）标量子查询

子查询结果为标量（一行一列，确切的一个值）

- 案例1

  谁的工资比Abel高

  1. 查询Abel的工资

     ```mysql
     select salary
     from employees
     where last_name = 'Abel';
     ```

  2. 查询员工的信息，满足条件salary>上述结果

     ```mysql
     select *
     from employees
     where salary>(
         select salary
         from employees
         where last_name = 'Abel'
     );
     ```

- 案例2

  返回`job_id`与141号员工相同，salary比143号员工多的员工姓名，job_id，工资

  ```mysql
  select last_name, job_id, salary
  from employees
  where job_id=(
      select job_id
      from employees
      where employee_id = 141
  ) and salary>(
      select salary
      from employees
      where employee_id = 143
  );
  ```

- 案例3

  返回公司工资最少的员工的last_name，job_id，salary

  ```mysql
  select last_name, job_id, salary
  from employees
  where salary=(
      select min(salary)
      from employees
  );
  ```

- 案例4

  查询最低工资大于50号部门最低工资的部门id和其最低工资

  ```mysql
  select department_id, min(salary)
  from employees
  group by department_id
  having min(salary)>(
      select min(salary)
      from employees
      where department_id=50
  )；
  ```

#### 2）列子查询

子查询结果为1列

- 案例1

  查询`location_id`是1400或1700的部门中的所有员工姓名

  ```mysql
  select last_name
  from employees
  where departments_id in (
      select department_id
      from departments
      where location_id in (1400,1700)
  );
  ```

- 案例2

  返回其他工种中比`job_id`为IT_PROG的工种任一工资低的员工的 工号、姓名、job_id以及salary

  ```mysql
  select employee_id, last_name, job_id, salary
  from employees
  where salary<(
      select max(salary)
      from employees
      where job_id="IT_PROG"
  ) and job_id <> "IT_PROG";
  ```

#### 3）行子查询

- 案例1

  查询员工编号最小并且工资最高的员工信息

  ```mysql
  select *
  from employees
  where (employee_id, salary)=(
      select min(employee_id),max(salary)
      from employees
  );
  ```

### 2. select后面

**仅支持标量子查询**

- 案例1

  查询每个部门的员工个数

  ```mysql
  select d.*,(
      select count(*)
      from employees e
      where e.department_id=d.department_id
  )
  from departments d;
  ```

- 案例2

  查询员工号=102的部门名

  ```mysql
  select (
  	select department_name
      from departments d
      inner join employees e on e.department_id=d.department_id
      where e.employee_id=102
  );
  ```

### 3. from后面

子查询结果充当一张表，必须起别名

- 案例1

  查询每个部门的平均工资的工资等级

  ```mysql
  select ag_dep.*,g.grade_level
  from (
      select avg(salary) ag, department_id
      from employees
      group by department_id
  ) ag_dep -- 必须起别名
  inner join job_grade g
  where ag_dep.ag between g.lowest_sal and g.highest_sal;
  ```

### 4. exists后面

`exists()`语句返回是否存在的一个布尔值，`exists()`语句判断**是否至少返回一行数据**

- 案例1

  查询有员工的部门名

  ```mysql
  select department_name
  from departments d
  where exists(
      select *
      from employees e
      where e.department_id=d.department_id
  );
  ```

  通过查询表【d】（2行）中每条记录的`d.department_id`是否在exists语句中（3到6行）存在，来判断是否出现在`select department_name`（1行）中

  也可以使用`in`的方式

  ```mysql
  select department_name
  from departments d
  where department_id in (
      select department_id
      from employees
  );
  ```

- 案例2

  查询没有女朋友的男神信息

  ```mysql
  select bo.*
  from boys bo
  where not exists(
      select *
      from beauty b
      where b.boyfriend_id=bo.id
  );
  ```

  也可以提供过使用`in`语句的形式

  ```mysql
  select bo.*
  from boys bo
  where bo.id not in (
  	select b.boyfriend_id
  	from beauty b
  );
  ```

### 5. 练习题

- 案例1

  查询和Zlotkey相同部门的员工姓名和工资

  ```mysql
  select last_name,salary
  from employees
  where department_id=(
      select department_id
      from employees
      where last_name="Zlotkey"
  );
  ```

- 案例2

  查询工资比公司平均工资高的员工的员工号，姓名和工资

  ```mysql
  select employee_id,last_name,salary
  from employees
  where salary>(
      select avg(salary)
      from employees
  );
  ```

- 案例3

  查询各部门中工资比本部门平均工资高的员工的员工号，姓名和工资

  ```mysql
  select employee_id,last_name,salary
  from employees e
  where salary>(
      select avg(salary)
      from employees em
      where e.department_id=em.department_id
  );
  ```

  以上代码为本人写出，测试后符合题意，答案代码如下：

  ```mysql
  select employee_id,last_name,salary,e.department_id
  from employees e
  inner join(
      select avg(salary) ag,department_id
      from employees
      group by department_id
  ) ag_dep
  on e.department_id=ag_dep.department_id
  where salary>ag_dep.ag；
  ```

- 案例4

  查询和姓名中包含字母u的员工在相同部门的员工的员工号和姓名

  ```mysql
  SELECT employee_id,last_name
  FROM employees
  WHERE department_id IN (
      SELECT DISTINCT department_id
      FROM employees
      WHERE last_name LIKE "%u%"
  );
  ```

- 案例5

  查询在部门的location_id为1700的部门工作的员工的员工号

  ```mysql
  SELECT employee_id
  FROM employees
  WHERE department_id = ANY (
  		select department_id
  		from departments
  		where location_id=1700
  );
  ```

- 案例6

  查询管理者是King的员工姓名和工资

  ```mysql
  SELECT last_name,salary
  FROM employees
  WHERE manager_id IN (
  		SELECT employee_id
  		FROM employees
  		WHERE last_name="K_ing"
  );
  ```

- 案例7

  查询工资最高的员工的姓名，要求first_name和last_name显示为一列，列名为【姓.名】

  ```mysql
  SELECT CONCAT(first_name,last_name) "姓.名"
  FROM employees
  WHERE salary=(
  		SELECT MAX(salary)
  		FROM employees
  );
  ```

- 案例8

  查询平均工资最低的部门的信息

  ```mysql
  select d.*
  from departments d
  inner join (
      select avg(salary),department_id
      from employees 
  	GROUP BY department_id
      HAVING avg(salary)=(
          select min(ag)
          from (
              select avg(salary) ag,department_id
              from employees
              group by department_id
          ) ag_dep
      )
  ) ag_dep_id on d.department_id=ag_dep_id.department_id
  ```

  ```mysql
  select d.*
  from departments d
  where d.department_id= (
      select department_id
      from employees
      group by department_id
      having avg(salary)=(  -- avg(salary)并没有出现在select后面
          select min(ag)
          from (
              select avg(salary) ag,department_id
              from employees
              group by department_id
          ) ag_dep
      )
  )
  ```

  

## 进阶8：分页查询

```mysql
select 查询列表
from 表
[join type] join 表2
on 连接条件
where 筛选条件
group by 分组字段
having 筛选条件
order by 排序字段
limit offset,size;
```

`offset`为要显示条目的起始索引（起始索引从0开始）

`size`为要显示的条目个数

- 特点：

- `limit`语句放在查询语句的最后

  - 公式：

    - 要显示的页数page，每页的条目数size

      `limit (page-1)*size,size;`

- 案例1

  查询前五条员工信息

  `select * from employees limit 0,5;`

  0可省略，如：`select * from employees limit 5;`

- 案例2

  查询第11条至第25条

  `select * from employees limit 10,15;`

- 案例3

  有奖金的员工信息，并且工资较高的前10名显示出来

  ```mysql
  SELECT *
  FROM employees
  WHERE commission_pct IS NOT NULL
  ORDER BY salary DESC
  LIMIT 10;
  ```

- 案例4

  查询平均工资最低的部门信息

  ```mysql
  SELECT d.*
  FROM departments d
  WHERE d.department_id=(
  	SELECT department_id
  	FROM employees
      GROUP BY department_id
      ORDER BY AVG(salary)
      LIMIT 1
  );
  ```

  使用`order by`和`limit`语句将平均工资的最小值筛选出来，与**进阶7.5.案例8**形成对比

- 案例5

  查询平均工资最低的部门信息和平均工资

  ```sql
  SELECT d.*,ag_dep.ag
  FROM departments d
  INNER JOIN (
      SELECT AVG(salary) ag,e.department_id
      FROM employees e
      GROUP BY department_id
      ORDER BY ag
      LIMIT 1
  ) ag_dep
  ON d.department_id=ag_dep.department_id;
  ```

## 进阶9：联合查询

union 联合，将多条查询语句的结果合并成一个结果，可以将多个表的查询结果合并成一个，查询结果的拼接是纵向的拼接

要查询的结果来自于多个表，且多个表没有直接的连接关系，但查询的信息一致

- 特点：
  1. 要求多条查询语句的查询列数是一致的，否则报错
  2. 要求多条查询语句的查询的每一列的类型和顺序一致
  3. `union`关键字默认去重，若不需要去重，则使用`union all`

```mysql
查询语句1
union
查询语句2
union
。。。
```

- 案例1

  查询部门编号>90或邮箱包含a的员工信息

  ```mysql
  SELECT * FROM employees WHERE email LIKE "%a%"
  UNION
  SELECT * FROM employees WHERE department_id>90
  ```

