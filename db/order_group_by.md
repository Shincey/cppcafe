# Order By / Group By

### Order By

`order by`关键字可以将结果集按照升序还是降序排列，默认时升序，可以通过指定`DESC`关键字来降序排列。

```sql
select column_1, column_2, column_3 ...
from table_name
order by column_1, column_2, column_3 ... ASC | DESC;
```

### Group By

`group by`语句将有相同值的元组聚类，经常配合一些聚合函数使用，如：`avg(), max(), count(), min()`等。需要知道的是，元组分组是根据元组的属性的相似性进行分组的。

```sql
select function_name(column_1), column_2
from table_name
where condition
group by column_1, column_2
order by column_1, column_2;
```

### Oder By 和 Group By 区别

| Group By                                 | Oder By                           |
| ---------------------------------------- | --------------------------------- |
| 将具有相同值的元组分成一组               | 将结果集按照升序或降序排列        |
| 可以允许在`create view`语句中            | 不在`create view`中使用           |
| 在`select`语句中，通常放到`order by`前面 | `select`中通常放到`group by` 后面 |
| `select`不能包含`group by`没有引用的属性 | 可以                              |


