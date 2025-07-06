# 引言

传统的[[文件处理系统]]过于依赖手工处理，并且层次式的数据形式，不利于查询

因此产生数据库系统设计的需要

数据库结构以 **[[数据模型]]（data model）** 为基础

本书主要讲 **[[关系模型]]**

出于降低系统复杂度的目的，我们需要引入 **[[数据抽象]]** 以屏蔽底层细节

---

数据库系统使用 [[DDL]]、[[DML]] 管理

DML 只是与数据库进行操作，要执行复杂命令，还是要用高级语言编程，称为*宿主(host)*

host 通过 预留好的接口 与数据库交互

---
数据库设计流程：
1. 刻画预期的用户需求，完成需求文档
2. 设计高层数据模型，完成概念设计
3. 规范化数据对象间的联系，可用E-R模型
4. 落实到具体 database 系统的逻辑设计模型
5. 最后执行数据存储的物理设计
---

数据库引擎
三个主要模块：
1. [[存储管理器]]
2. [[查询处理器]]
3. [[事务管理部件]]
---
现代数据库应用采用三层体系结构：
- 【用户—应用客户】—【应用服务器】—【数据库系统】
- 客户作为前端，不直接与数据库进行交互
- 提供了更好的安全性与性能
---
数据库用户
四类：
1. 初学者用户：傻瓜，门外汉
2. 应用程序员：开发者
3. 老练用户：分析师等
4. 数据库管理员（[[DBA]]）

---
# 关系语言

关系模型采用以下术语：
1. **关系**：表示整张表
2. **元组**：表示表中的一行
3. **属性**：表示一列
	1. 每个属性有其值域，称为 **域**

关系数据库就采用这种表形式存储数据，并通过属性间的关联，描述数据间的联系

关系中的元素必须是*原子的*

---

要区分不同 tuple，必须有能唯一辨识 tuple 的 属性 或 属性集合

术语：
- 超码：能唯一标识，可以有冗余
- 候选码：唯一标识，最小超码
- **主码**：设计者指定的候选码

在同一个关系中，使用主码区分 tuple

**主码是针对特定关系的概念**

术语：
- **主码约束**：主码必须互不相同
- **外码约束**：必须有一个属性的值被引用
	- 否则该关系就不能与其他关系建立联系，成为一个孤立的关系
> 外码约束 与 引用完整性约束 的区别
> - 后者是设计原则上的概念
> - 前者是系统具体实现的机制
> - 前者是后者的具体落实


---

关系查询语言为用户提供服务，分为
- 命令式
- 函数式
- 声明式
---
# [[SQL]] 介绍

## 初级 SQL

本章介绍 SQL DML 与 DDL

---
### 基本语法
#### SQL 类型
- `char(n)`、`varchar(n)`：字符串
- `int`、`smallint`：整数
-  `numeric(p, d)` 定点数
- `real`、`double precision` 浮点数
- `float(n)` 浮点数
---

#### SQL 关系定义
```sql
create table department
	(dept_name varchar(20),
	building   varchar(15) not null,
	budget     numeric(12, 2),
	primary key (dept_name, ...),
	foreign key (dept_name, ...) references someone);
```

---

#### 增删语句
```sql
drop table r;  // 连关系模式也删除
delete from r; // 保留关系模式

alter table r add A D; // 添加属性
alter table r drop A;  // 删除属性
```

---
#### 查询语句

`select` 返回一个只有指定属性的关系
- `distinct` 指示结果应去重
- `all` 指示结果不要去重
`from` 指名涉及到的关系
`where` 从笛卡尔积中筛选满足特定条件的 tuple

 ```sql
 select x, y
 from A, B
 where A.name=B.name;
```
---
#### 更名运算
然而不同关系可能具有同名属性，必须加上关系名`A.x, B.x` 作区分

为防止 `select`的结果中，属性名称过长，SQL提供了 `as` 重命名运算，在结果中更改属性名

也可用于 `from` 中，简化关系名或者避免歧义
```sql
select x as A.x 
from A;

select distinct T.name
from instructor as T, instruct as S
where T.salary > S.salary and S.dept_name = 'Biology';
```

---
#### 字符串运算

- `%`匹配任意字串，`_`匹配单个任意字符
- 用 `like` 作为关键字，`escape`转义字符
```sql
select A
from B
where C like '%__aadfa_\%' escape '\'
```

---
- `*` 可代表一个关系的所有属性
- `order by` 规定属性顺序
- `where` 支持元组比较

---
#### 集合运算

```sql
A union B;
A intersect B;
A except B;
```

---
#### 空值
包含 `null` 的逻辑运算结果被认为是 `unknown`

---
### [[聚集函数]]

```sql
select x, avg(y) as z
from A
group by k
having avg(x) > 100
```
`group by` 提供分组聚集功能
> `group by`  中没有出现的属性，不能被 `select`，只能作为聚集函数的参数

`having` 提供分组筛选功能，类似于`where`
聚集函数一般忽略`null`空值

---
### 嵌套子查询

- `in` `not in` 包含
- `some` `all` 全称量词
- `exists` `not exists` 是否存在
- `unique` `not unique` 是否去重
- `with` 临时定义一个关系
- 只有一行一列的关系可以作为 **标量子查询**
---
### 修改

`delete`，`insert`，`update`对元组进行删增
`case` 条件判断

---
## 中级 SQL

### 连接
1. `natural join` 
2. `join ... using`
3. `join ... on`
4. `outer join` 填充空值
	1. `left` / `right` / `full`
5. `inner join`
---
### 视图
 ```sql
 create view faculty(x, y) as
	 select ID, name, dept_name
	 from instructor;
```

视图只定义查询过程，在查看时才由系统执行该过程，惰性求值

一般不允许对 view 进行修改，除非满足以下条件：
1. `from` 单个关系
2. `select` 不包括表达式
3. 筛掉的属性可以为`null`
4. 没有 `group by` `having`

插入时检查 tuple 是否满足`WHERE`条件

---
### 事务

保证逻辑操作的原子性
- `commit`/`rollback`

---
### 完整性约束

防止用户对数据的意外破坏（保护一致性）

`alter table <x> add <y>` 为现有关系添加约束

1. `not null`
2. `unique`
3. `check <something>`
4. `foreign key (x) references r(x)`
5. `on delete cascade` 级联删除，将外码关联的元组一并进行处理，可递归

通过 `constraint <name> <x>` 为约束命名，一个关系可以通过 `drop` 删除约束命名

一个原子事务在执行过程中，可以允许暂时不满足约束，这一点通过将约束 `set` 为 `deferred` 实现

定义断言 `assertion`，数据库认为该约束应永远为真，修改时必检查

---
### 数据类型和模式

类型：
1. `date`
2. `time`
3. `timestamp`

使用 `cast(e as t)`实现类型转换

`coalesce(salary, 0)`指明将 `salary` 中的空值以  `0` 显示

`clob`、`blob`为大对象类型，声明时用大小初始化`clob(10KB)`

大对象一般不直接读写，那样太慢了，我们只存入其定位器（类似地址），由宿主语言完成访问操作

`create type` 提供类似 `typedef` 的功能，自定义类型

`create domain` 类似，不过可以为类型加入约束，语义上为值域

`ID number(5) generated always as identity`，用于生成唯一编码，不能被覆盖

`create table like/as` 用于创建副本

SQL 通过多层目录实现用户隔离与全局环境

### 索引

`create index`，索引不是必要的

### 授权

四种权限：
1. 读取
2. 插入
3. 更新
4. 删除
授权 `grant on to` 收回`revoke on from`

`public` 用户代表全局用户权限

`create role` 角色、权限组

SQL 提供**引用(refereences)** 权限，限制外码的定义

`with grant option` 为用户提供转移权限的能力

收权时，要注意级联收权的问题
`restrict` 用于阻止级联收权

`granted by <current_role>` 将角色本身作为授权者，而不是具体用户

可以将授权操作的细粒度具体到一个关系中的 tuple

---

## 高级 SQL

SQL 是一种声明式语言
### 宿主语言【已跳过】
---
### 函数和过程

SQL 允许定义函数、过程、方法

`declare` 用于声明变量
`set` 用于赋值
循环、条件判断、异常处理

---
### 触发器

```sql
create trigger <name> after <option> on <x>
referencing new row as now
```
---
### 递归查询

---
### 高级聚集特性

- `rank()`
-  window
- `pivot`
- 

---