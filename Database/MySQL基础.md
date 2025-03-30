# MySQL

- 常用可视化工具
  - navicat
  - workbench



## DDL 数据定义语言

- 用来定义数据库对象，如数据库、表、列

- 关键字：create、drop、alter、truncate

### `create`

创建一个数据库

```mysql
create database game; 
#使用数据库/进入数据库
use game;
```
创建一个表

```mysql
create table player(
	id int,
	name varchar(100),
	level int,
);
#mysql的数据类型
#char 定长字符串	varchar变长字符串 text文本 blob二进制数据
#int 整型
#decimal(10,2) 长度为10保留两位小数的十进制数值
#datetime 时间日期

#描述表的结构
desc player;
```
---

### `drop`
删除数据库

```mysql
drop database game;
```

删除表

```mysql
drop table player;
```

---

### `alter`

修改表的结构

```mysql
alter player modify column name varchar(200);

#修改默认值
alter player modify level int default 1;

alter player rename column name to newName;

#sql通常不区分大小写，所以最好用驼峰命名法
alter player add column last_login datetime;

alter player drop column last_login;
```

---

## DML 数据操作语言

- 对数据库中记录进行增删改

- 关键字：insert、update、delete、call

### `insert`

```mysql
#insert into + 表名 + (列名) + values (对应值),(),() ... （可以一次插入多个数据
insert into player(id, name, level) values (1, 'shen', 1);
#列名不需要全写，写只需要添加的内容，其他使用默认值
#如果values按顺序写入，列名可以忽略
insert into player values (1, 'yao', 1);
```

### `update`

```mysql
#如果不指定where，则表明对player的所有值修改
update player set level = 2 where name = 'shen';
```

### `delete`

```mysql
delete from player where name = 'yao';
```

### `call`



---

## DQL 数据查询语言

- 用于查询数据库中的记录
- 关键字：select、where

### `select`

- select 返回集合（可以选择输出对应的属性）from 表 where 选择对象的条件

```mysql
#从表中查询对应行，*表示展示出来的行对应的列，可以替换具体
select * from player; 
```

### `where`

```mysql
#通过where获得满足要求的行
select * from player where name = 'yao';

#where条件可以使用逻辑运算，优先级 () > not > and > or
select * from player where level > 1 and level < 5;

#使用in来指定多个值
select * from player where level in (1,3,5);

#使用between and查询一个范围，等价与a =< x <= b
select * from player where level between 1 and 10;

#not可以在任何一个条件语句，或者比较运算符前面
select * from player where level not between 1 and 10;

#is null/is not null来查询是否空的行，使用 = 在mysql中不行
#但空字符串不是null，需要用''
select * from player where email is not null;

#通过order by 列名 来将内容升序排列
#通过order by 列名 desc 来将内容降序排列
#通过order by 列名1 desc, 列名2 asc 来进行多个条件排序
select * from player order by level asc, exp desc;

#distinct去除重复数据
select distinct sex from player;

```

### 模糊查询

like模糊查询，获得所有匹配的行，可用于与查询简单的表达式，需要配合%_来使用

- %表示任意长度字符
  - '姚%'表示姚开头的任意字符串
  - '%姚'表示姚结尾的任意字符串
  - '%姚%'表示含有姚的任意字符串
- _表示一个字符
  - '姚_'表示姚x的所有字符串
  - '姚__'表示姚xx的所有字符串

```mysql
select * from player where name like '姚%';
```

regexp正则表达式，来查询复杂的表达式，用正则表达式的通配符构造正则表达式

- . 表示任意一个字符
- ^ 表示开头 $ 表示结尾  
  - '^姚' 姚开头的任意长度的字符串
- [abc] 表示匹配[]内任意字符
- [a-z] 表示匹配范围内任意字符
- A|B 表示匹配A|B

```mysql
select * from player where name regexp '^姚.$'; #表示姚x
```

### 聚合函数

常用聚合函数

- AVG() 返回集合的平均值
- COUNT() 返回集合中项目数量
- MAX() 返回最大值
- MIN() 返回最小值
- SUM() 求和
- SUBSTR(string, begin, len) 截取字符串
- ROUND() 浮点数四舍五入返回
- EXISITS() 返回查询是否有结果

```mysql
#输出player中所有项目
select count(*) from player;
select count(*) from player where level = 5;

#输出level平均值
select avg(level) from player;
```

### 分组查询

```mysql
#使用group by进行分组
select sex,count(*) from player group by sex;
select level,count(level) from player group by level; #不会考虑null的level行

#配合having使用，having筛选分组后的数据
select level,count(level) from player 
group by level 
having count(level) > 4;
#再配合order by排序
select level,count(level) from player 
group by level 
having count(level) > 4 
order by count(level) desc, level asc;
#使用limit限制数量
select level,count(level) from player 
group by level 
having count(level) > 4 
order by count(level) desc, level asc
limit 3;#limit可以是偏移量如limit 1,3; 1表示偏移量，3表示数量

#上述可以构成分页查询的原理
```

### 并交集

```mysql
#使用union查询集合的并集
select * from player where level > 3
union
select * from player where exp > 3;
#使用union all不会去除重复元素

#使用intersect查询集合的交集
select * from player where level > 3
intersect
select * from player where exp > 3;
```

---

### 子查询

- 一个查询结果作为另外查询结果的条件
- 嵌套式的查询
- 子查询可以放在任何可以放值得地方，如select where create等

```mysql
select avg(level) from player;
#使用子查询查询所有大于平均值的玩家
select * from player where level > (select avg(level) from player);


```

---

### 表关联

- 可以使用join on关键字进行
- 也可以直接使用where
- 本质使用笛卡尔积然后过滤出来

```mysql
#inner jion //只返回关联的内容
#left jion	//返回左表所有内容，没有连接内容为空
#right jion	//返回右表所有内容，没有内容为空
select * from player
inner join equip
on player.id = equip.player_id;

#使用where
select * from player , equip
where player.id = equip.player_id;
#使用别名
select * from player p, equip q
where p.id = q.player_id;

```

---

### 索引

#### 创建添加索引

- create [unique|fulltext|spatial] index index_name on table_name (type); type表示在哪些字段上创建索引
- unique 唯一索引，fulltext 全局索引，spatial 空间索引

```mysql
create index email_index on fast (email);

#也可以使用alter添加索引
alter table fast add index name_index(name);

```

#### 查看索引

- 查看这个表上所含有的索引表

```mysql
show index from fast;
```

#### 索引查询

```mysql
#没有索引查询，速度慢
select * from slow where email like 'abcd%' order by id;

#索引查询
select * from fast where email like 'abcd%' order by id;
```

#### 删除索引

```mysql
drop index email_index on fast;

```

---

### 视图

- 一种虚拟存在的表，本身不包含数据，作为一种查询语句保存

```mysql
#创建
create view top10
as
select * from player order by level desc limit 10;

#查询
select * from top10;

#修改
alter view top10
as
select * from player order by level asc limit 10;

#删除
drop view top10;
```





## DCL 数据控制语言

- 用来定义数据库的访问权限和安全级别
- 关键字：grant、revoke



## 数据库常用命令

### 数据库导入导出

```shell
#导出语句，导出后的内容便是一条一条的sql语句，导入时mysql会执行该文件中所有sql语句进行创建添加
mysqldump -uroot -p030326 game player(表名可以省略) > game.sql (最后就是shell的重定位)

#导入语句
mysql -u -p game < game.sql
```

### 别名

```
#select中可以用as起别名，方便查看
select s_name as name from student;
```

