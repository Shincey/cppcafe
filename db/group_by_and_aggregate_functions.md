# Group By 和 聚合函数

我们经常会发现根据组的某些特征来进行分组，（如部门等），可以很方便对组的一些信息进行统计（如总值、平均值等）。如果要计算部门的平均工资，用户可以按部门将所有员工的工资进行分组，`group by`之句用于将表中元组分组，其形式为：

```sql
group by column_1 [, column_2] ...
```

配合`select`如下：

```sql
select column_1 [, column_2] ...
from table_name
[where condition]
[group by column_1 [, column_2] ...]
[order by column_1 [DESC] [, column_2] [DESC] ]...;
```

`group by`指定的列用于形成组，其把具有相同的指定的一个或多个列的属性的值的元组放在一个组中。**分组只是概念上的分组，并不是物理上的重新排列**。

`group by`通常有5个内置聚合函数使用：**`sum / avg / max / min / count`**。

聚合函数使用在`select`语句中，用来替代列名：

``` sql
function([distinct] argument)
```

`distinct`关键字指定重复的值只处理一次，比如和`count`一起使用来计算不同值的数量。

举例：计算每个部门的平均工资？

```sql
select dept, avg(salary)
from employee
group by dept;
```

可能有如下输出：

```sql
dept		avg(salary)
-----------------------
MKT			33500
MGT			45000
LIB			22000
RND			27500
FIN			42000
```

该查询语句中，表`employee`中所有行按照部门`dept`分组，聚合函数`avg`计算每个组的平均工资，最终并展示每个部门及其平均工资。

**一个包含聚合函数的select语句，不能包含任何group by没有使用的的列**，例如：

```sql
select ename, avg(salary)
from employee
group by dept;
```

该查询语句会报错，`select`使用的列在`group by`中没有使用是不被允许的。所以，**`select`只能使用`group by`指定的列以及聚合函数**。上面的例句，由于`ename`没有包含在`group by`中，所以会出错。

举例：计算MKT部门平均工资？

```sql
select count(*), avg(salary)
from employee
where dept = 'MKT';
```

可能输出：

```sql
count(*)		avg(salary)
---------------------------
2				33500
```

在这个例子中，聚合函数被使用在一个有`where`的`select`语句中。

你可以将表中的元组按照多个列来分组。例如计算每个部门的总工资，然后，每个部门按照福利分类汇总。

举例：每个部门按福利分类支付的总工资是多少？

```sql
select dept, benefits, sum(salary)
from employee
group by dept, benefits;
```

在这个例子中，我们通过部门进行分组，然后在每个部门内，通过福利进行分类。可能的输出：

```sql
dept		benefits		sum(salary)
---------------------------------------
FIN			FULL			42000
LIB			PART			22000
MGT			FULL			45000
MKT			FULL			67000
RND			FULL			30000
RND			PART			25000
```

当一个聚合函数被使用时，而没有`group by`子句，那么整个表会被当成一个组，最终显示一个单一值。

举例：计算所有员工的总工资？

```sql
select sum(salary)
from employee;
```

可能输出：

```sql
sum(salary)
-----------
231000
```

