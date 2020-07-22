# Where / Having

### Where

`where`子句用来过滤查询值或者同时连接多个表使用。符合条件的记录才会被抽取，可以使用在`select`、`update`、`delete`语句中。

假设有一张表`Student`

```sql
Roll_no			S_name		Age
-------------------------------
1				a			17
2				b			20
3				c			21
4				d			18
5				e			20
6				f			17
7				g			21
8				h			17
```

查询：

```sql
select S_name, Age
from Student
where Age >= 18;
```

只有年龄大于等于18的行才会返回。

### Having

`having`用于从给定条件的组中过滤记录。那些满足给定条件的组会出现最终结果中。`having`只能用在`select`中。

```sql
select Age, count(Roll_no) as No_of_Students
from Student
group by Age
having count(Roll_no) > 1;
```

输出：

```
Age			No_of_Students
--------------------------
17			3
20			2
21			1
```

### where 和 having 区别

| where                                                     | having                                           |
| --------------------------------------------------------- | ------------------------------------------------ |
| `where`子句被用于过滤符合条件的记录                       | `having`子句用于过滤组中符合条件记录             |
| `where`可以不搭配`group by`一起使用                       | `having`必须和`group by`一起使用                 |
| `where`子句不能包含聚合函数                               | `having`子句可以包含聚合函数                     |
| `where`可以和`select`、`update`、`delete`一起使用         | `having`只能和`select`一起使用                   |
| `where`在`group by`前面使用                               | `having`在`group by`后面使用                     |
| `where`子句可以和单行作用的函数一起，如`upper`、`lower`等 | `having`和多行作用的函数一起，如`sum`、`count`等 |

