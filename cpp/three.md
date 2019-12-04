# 第三节

正则表达式的使用

```c++
bool regex_search(seq, m, r, mft); //子串符合返回true
bool regex_match(seq, r, mft); //整个字符串符合返回true
//seq查找的字符串，可以是string，char *或迭代器
//m是一个match对象，保存结果，m和seq必须具有兼容类型
//mft是一个可选的regex_constants::match_flag_type值
```

