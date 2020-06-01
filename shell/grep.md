# grep

`grep`是UNIX和LINUX中使用最广泛的命令之一。`grep`支持正则表达式，也支持其扩展集。其有三种变形，即：

1. Grep：标准`grep`命令，以下主要介绍这个
2. Egrep：扩展`grep`，支持基本及扩展的正则表达式，但不支持`\q`模式范围的应用
3. Fgrep：快速`grep`，允许查找字符串而不是一个模式。



`grep`的一般格式为：

```shell
$ grep [options] 基本正则表达式 [file]
```

这里的基本正则表达式可为字符串。

在命令中输入参数时，最好用双引号将其括起来，这样做一方面防止误解为shell命令，一方面可以用来查找多个单词组成的字符串，如“hello world”，如果不括起来，可以hello会被认为一个文件。在调用变量时，也应该使用双引号，如`grep "$VAR"`，如果不这样，奖没有返回结果。

在调用匹配模式时，应该使用单引号。

`grep`的常用选项有：

* `-c` - 只输出匹配行的计数
* `-i` - 不区分大小写（只适用于单字符）
* `-h` - 查询多文件时不显示文件名
* `-l` - 查询多文件时只输出包含匹配字符的文件名
* `-n` - 显示匹配行及行号
* `-s` - 不显示不存在或无匹配文本的错误信息
* `-v` - 显示不包括匹配文本的所有行



#### 查询多文件

```shell
$ grep "sou" date.txt #在date.txt文件中查找字符串"sou"
$ grep "sou" *.txt #在当前目录下所有.txt文件中查找字符串"sou"
$ grep "hello world" *.txt #在当前目录下所有.txt文件中查找单词"hello world"
```



#### 精确查找

假设data.txt中包含两行：hello world、hellow world。那么使用`grep "hello" data.txt`则会匹配到这两行，`grep`抽取精确匹配的一种更有效方式是在抽取字符串后加`\>`：

```shell
$ grep 'hello\>' data.txt
```



#### grep和正则表达式

使用正则表达式最好使用单引号括起来，这样可以防止`grep`中使用的专有模式与一些shell命令的特殊方式相混淆。

```shell
$ grep 'hel\{2\}o' data.txt
```

`grep`命令加`-E`参数允许使用扩展模式匹配：

```shell
$ grep -E '32|23' data.txt
```



#### 类名

`grep`允许使用国际字符模式匹配或匹配模式的类名形式:

| 类          | 等价的正则表达式 |
| ----------- | ---------------- |
| [[:upper:]] | [A-Z]            |
| [[:lower:]] | [a-z]            |
| [[:digit:]] | [0-9]            |
| [[:alnum:]] | [0-9a-zA-Z]      |
| [[:space:]] | 空格或tab键      |
| [[:alpha:]] | [a-zA-Z]         |



#### 获取系统信息

```shell
$ ls -l | grep '^d' #查询目录列表中的目录
```

如果`grep`不支持`-s`选项，可重定向输出或错误（2>&1）到系统池（/etc/null）：

```shell
$ grep "tom" /etc/password >/dev/null 2>&1
```

使用带有`ps x`命令的`grep`可查询系统上运行的进程。`ps x`命令是显示系统上运行的所有进程列表。如，查看DNS服务器是否在运行：

```shell
$ ps ax | grep "named"
```


