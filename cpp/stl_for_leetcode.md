# 刷题该了解的STL

### 容器

容器几乎都有`empty`函数判断容器是否为空，`size`函数返回容器存储元素的数量，`begin / end`或`rbegin / rend`配合迭代器。

#### std::vector

头文件：`<vector>`

成员函数：

* `operator[]` 或 `at`：都可以通过下标索引访问特定元素。
* `front` / `back`：返回第一个元素 / 最后一个元素。
* `data`：返回指向底层数组的指针。
* `insert`：在迭代器指向的地方插入一个或一段数据。
* `push_back`：在容器最后插入一个元素。
* `pop_back`：弹出容器最后一个元素。
* `erase`：将迭代器指向的数据删除。若传递两个迭代器则删除一段数据。
* `clear`：清空容器。



#### std::stack

头文件：`<stack>`

成员函数：

* `top`：返回栈顶元素，不弹出。
* `pop`：弹出栈顶元素，不返回任何值。
* `push`：数据进栈。



#### std::queue

头文件：`<queue>`

成员函数：

* `front` / `back`：返回第一个 / 最后一个元素。
* `push` / `pop`：队尾入队 / 队首出队。都不返回任何值。



#### std::set

头文件：`<set>`

成员函数：

* `insert`：插入一个值
* `earse`：删除一个值
* `count`：返回元素出现的次数，集合中元素最多也就出现1次，可以用来判断在不在问题。
* `find`：返回给定元素的迭代器。
* `lower_bound` / `upper_bound`：返回第一个不小于 / 最后一个大于 给定元素的迭代器。



#### std::map

头文件：`<map>`

成员函数：

* `operator[]`：更新或插入一个元素，根据给定的键。
* `insert`：插入一个键值对。
* `erase`：删除迭代器指向的键值对。
* `count`：返回给定键的元素数量，只有0和1，可以用来判断在不在问题。
* `find`：返回给定键的迭代器。
* `lower_bound` / `upper_bound`：和set的作用一样。



***

### 算法

头文件：`<algorithm>`

函数：

* `all_of`：所有都为真返回真。容器空时返回真。
* `any_of`：一个为真就返回真。容器为空返回假。
* `none_of`：全假的时候返回真。容器为空返回真。（以上三个都可以自定义条件函数）
* `for_each`：将两个迭代器之间的元素，每个作为参数执行一遍给定的函数。
* `count`：统计给定元素出现的次数。
* `count_if`：统计满足条件的元素出现的次数。
* `find`：在给定范围内找到第一个和给定值相等的元素。
* `find_if`：在给定范围内找到第一个满足给定条件的元素。
* `find_if_not`：在给定范围内找到第一个不满足给定条件的元素。
* `find_end`：在给定范围内找到最后一个和给定值相等或满足给定条件的元素。
* `search`：在给定范围内寻找给定range。
* `copy` / `copy_if`：复制或复制满足条件的元素
* `transform`：每个元素都传递给指定函数并将结果存储在给定区域。
* `remove`：在给定范围内删除和给定值相等或满足给定条件的元素。
* `is_partitioned`：判断在给定范围内满足条件的元素和不满足的是否是分开的。
* `partion`：按给定条件将元素分为两组，返回分界处的迭代器。
* `is_sorted`：判断是否是升序，可指定判断条件。
* `sort` / `stable_sort`：按照给定条件排序。
* `partial_sort`：排序前N个元素。
* `binary_search`：二分查找。
* `merge`：合并两个有序的序列。
* `max` / `min`：返回给定值中最大最小的值。可指定条件函数。
* `max_element` / `min_element`：返回容器中最大最小元素

头文件：`<numeric>`

函数：

* `accumulate`：累加给定范围内的元素。可指定初始值，可指定运算符。
* `inner_product`：内积。
* `adjacent_difference`：相连的相减，可指定运算符。
* `partial_sum`：每个元素是前面的累积，默认加法，可指定运算法。



***

### 其它

#### 运算符

头文件：`<functional>`

* `plus`：加，通过`std::plus<int>`指定整型加法，其它类似。
* `minus`：减
* `multiplies`：乘
* `divides`：除
* `modules`：取余
* `negate`：-x
* `equal_to`：相等
* `not_equal_to`：不相等
* `greater`：大于
* `less`：小于
* `greater_equal`：大于等于
* `less_equal`：小于等于
* `logical_and`：逻辑与
* `logical_or`：逻辑或
* `logical_not`：逻辑非
* `bit_and`/`bit_or`/`bit_xor`/`bit_not`：按位与/或/异或/非



#### 最大最小值

头文件：`<limits>`

`std::numeric_limits<type>::min() / max()`
