# Makefile使用(4)-使用函数

**条件判断**：条件表达式可以是比较变量的值，或是比较变量和常量的值。

```makefile
libs_for_gcc = -lgnu
normal_libs =
foo: $(objects)
ifeq ($(CC), gcc)
	$(CC) -o foo $(objects) $(libs_for_gcc)
else
	$(CC) -o foo $(objects) $(normal_libs)
endif
```

关键字`ifeq`指定一个条件表达式，包含两参数，参数还可以是make函数，以逗号隔开，表达式以圆括号括起来；`else`表示条件判断为假的情况；`endif`表示结束。

```makefile
ifeq ($(strip $(foo)),)
<text-if-empty>
endif
```

这个示例中使用了`strip`函数，如果这个函数返回空，那么条件表达式为真。

`ifeq`和`ifneq`语法：

```makefile
ifeq (<arg1>, <arg2>)
ifeq '<arg1>' '<arg2>'
ifeq "<arg1>" "<arg2>"
ifeq "<arg1>" '<arg2>'
ifeq '<arg1>' "<arg2>"
ifneq (<arg1>, <arg2>)
ifneq '<arg1>' '<arg2>'
ifneq "<arg1>" "<arg2>"
ifneq "<arg1>" '<arg2>'
ifneq '<arg1>' "<arg2>"
```

还有一个关键字`ifdef`，语法为：

```makefile
ifdef <variable-name>
```

如果`variable-name`的值非空，那表达式为真，这个同样也可以是函数返回值。`ifdef`只是测试一个变量是否有值，并不会把变量扩展到当前位置。

```makefile
# example 1.
bar =
foo = $(bar)
ifdef foo # true
...
endif
# example 2.
bar =
ifdef bar # false
endif
```

当然也存在`ifndef`关键字。特别要注意的是，make在读取makefile时就计算条件表达式的值，并根据条件表达式的值来选择语句，所以最好不要把自动化变量放入条件表达式中，因为自动化变量在运行时才有。为了避免混乱，make不允许把整个条件语句分成两部分放在不同文件中。

---

---

**函数**：makefile中可以使用函数处理变量，返回值可以当做变量来使用。调用语法如下：

```makefile
$(<function> <arguments>)
# or
${<function> <arguments>}
```

这里，`<function>`表示函数名，`<arguments>`表示参数，参数间以逗号分隔，而函数名和参数间以空格隔开。

**字符串处理函数**：

```makefile
$(subst <from>,<to>,<text>)
```

* 名称：字符串替换函数`subst`
* 功能：把字串`<text>`中的`<from>`字符串替换成`<to>`
* 返回：函数返回被替换过后的字符串
* 示例：`$(subst ee, EE, feet on the street)`返回`fEEt on the strEEt`

```makefile
$(patsubst <pattern>,<replacement>,<text>)
```

* 名称：模式字符串替换函数`patsubst`
* 功能：查找`<text>`中单词（单词以空格、tab、回车、换行分隔）是否符合模式`<pattern>`，如果匹配的话，则以`<replacement>`替换。这里`<pattern>`可以包括通配符`%`，表示任意长度字串，如果`<replacement>`中也包含`%`，那么这个`%`将会是`<pattern>`中的`%`所代表的字串
* 返回：函数返回被替换后的字符串
* 示例：`$(patsubst %.c,%.o,x.c.c bar.c)`返回`x.c.o bar.o`

```makefile
$(strip <string>)
```

* 名称：去空格函数`strip`
* 功能：去掉`<string>`字串中开头和结尾的空字符
* 返回：返回被去掉空格的字符串
* 示例：`$(strip a b c )`返回`a b c`

```makefile
$(findstring <find>,<in>)
```

* 名称：查找字符串函数`findstring`
* 功能：在字串`<in>`中查找`<find>`字串
* 返回：如果找到返回`<find>`，否则返回空字符串
* 示例：`$(findstring a, a b c)`返回`a`

```makefile
$(filter <pattern...>,<text>)
```

* 名称：过滤函数`filter`
* 功能：以`<pattern>`模式过滤`<text>`字符串中的单词，保留符合模式`<pattern>`的单词。可以有多个模式。
* 返回：返回符合模式`<pattern>`的字串
* 示例：`$(filter %.c %.s, foo.c bar.s ugh.h)`返回 `foo.c bar.s`

```makefile
$(filter-out <pattern...>,<text>)
```

* 名称：反过滤函数`filter-out`
* 功能：以`<pattern>`模式过滤`<text>`字符串中的单词，去除符合`<pattern>`的单词。可以有多个模式
* 返回：返回不符合模式`<pattern>`的字串
* 示例：`$(filter-out main1.o main2.o, main1.o foo.o main2.o bar.o)`返回`foo.o bar.o`

```makefile
$(sort <list>)
```

* 名称：排序函数`sort`
* 功能：给字符串`<list>`中单词排序(升序)
* 返回：返回排序后的字符串
* 备注：`sort`函数会去除`<list>`中相同的单词
* 示例：`$(sort foo bar lose)`返回`bar foo lose`

```makefile
$(word <n>,<text>)
```

* 名称：取单词函数`word`
* 功能：取字符串`<text>`中第`<n>`个单词，从1开始索引
* 返回：返回字符串`<text>`中第`<n>`个单词，如果`<n>`大于单词个数就返回空字符串
* 示例：`$(word 2, foo bar baz)`返回`bar`

```makefile
$(wordlist <ss>,<e>,<text>)
```

* 名称：取单词函数`wordlist`
* 功能：从字符串`<text>`中取`<ss>`开始到`<e>`的单词串。`<ss>`和`<e>`是索引，闭区间
* 返回：返回字符串`<text>`中从`<ss>`到`<e>`的单词字符串。如果`ss`大于`e`返回空，如果`e`大于单词个数，就返回`ss`到`text`结束字符串。
* 示例：`$(wordlist 2, 3, foo bar baz)`返回`bar baz`

```makefile
$(words <text>)
```

* 名称：单词个数统计函数`words`
* 功能：统计`<text>`字符串中单词个数
* 返回：返回单词数
* 示例：`$(words foo bar baz)`返回3

```makefile
$(firstword <text>)
```

* 名称：首单词函数`firstword`
* 功能：取字符串`<text>`中第一个单词
* 返回：返回字符串`<text>`第一个单词
* 示例：`$(firstword foo bar)`返回`foo`

**文件名操作函数**：

```makefile
$(dir <names...>)
```

* 名称：取目录函数`dir`
* 功能：从文件名序列`<names>`中取出目录部分。目录部分是指最后一个反斜杠`/`之前的部分，如果没有反斜杠就返回`./`
* 返回：返回文件名序列`<names>`目录部分
* 示例：`$(dir src/foo.c hacks)`返回`src/ ./`

```makefile
$(notdir <names...>)
```

* 名称：取文件函数`notdir`
* 功能：从文件名序列`<names>`中取出非目录部分，指最后一个反斜杠之后的部分
* 返回：返回文件名序列`<names>`的非目录部分
* 示例：`$(notdir src/foo.c hacks)`返回`foo.c hacks`

```makefile
$(suffix <names...>)
```

* 名称：取后缀函数`suffix`
* 功能：从文件名序列`<names>`中取出各个文件名的后缀
* 返回：返回文件名序列`<names>`的后缀序列，如果文件没有后缀，则返回空字符串
* 示例：`$(suffix src/foo.c src-1.0/bar.c hacks)`返回`.c .c`

```makefile
$(basename <names...>)
```

* 名称：取前缀函数`basename`
* 功能：从文件名序列`<names>`中取出各个文件名的前缀部分
* 返回：返回文件名序列`<names>`的前缀序列，如果文件没有前缀，返回空字符串
* 示例：`$(basename src/foo.c src-1.0/bar.c hacks)`返回`src/foo src-1.0/bar hacks`

```makefile
$(addsuffix <suffix>,<names...>)
```

* 名称：加后缀函数`addsuffix`
* 功能：把后缀`<suffix>`加到`<names>`中每个单词后面
* 返回：返回加过后缀的文件名序列
* 示例：`$(addsuffix .c, foo bar)`返回`foo.c bar.c`

```makefile
$(addprefix <prefix>,<names...>)
```

* 名称：加前缀函数`addprefix`
* 功能：把前缀`<prefix>`加到`<names>`中每个单词后面
* 返回：返回加过前缀文件名序列
* 示例：`$(addprefix src/, foo bar)`返回`src/foo src/bar`

```makefile
$(join <list1>,<list2>)
```

* 名称：连接函数`join`
* 功能：把`<list2>`中的单词对应的加到`<list1>`的单词后面，如果`<list1>`的单词个数要比`<list2>`中的多，那么，`<list1>`多出来的单词部分保持原样；如果`<list2>`中单词个数要多，那么，`<list2>`多出来的单词将被复制到`<list1>`中。
* 返回：返回连接后的字符串。
* 示例：`$(join a, b, 1, 2, 3)`返回`a1 b2 3`

**foreach函数**：

```makefile
$(foreach	,<list>,<text>)
```

这个函数的意思是，把参数`<list>`中的单词逐一取出放到参数所指定的变量中，然后再执行`< text>`所包含的表达式。每一次`<text>`会返回一个字符串，循环过程中，`<text>`的所返回的每个字符串会以空格分隔，最后当整个循环结束时，`<text>`所返回的每个字符串所组成的整个字符串（以空格分隔）将会是`foreach`函数的返回值。

所以，最好是一个变量名，`<list>`可以是一个表达式，而`<text>`中一般会使用这个参数来依次枚举`<list>`中的单词。举个例子:

```makefile
names := a b c d
files := $(foreach n, $(names), $(n).o)
```

`$(names)`中的单词会被挨个取出，并存到变量`n`中，`$(n).o`每次根据`$(n)`计算出一个值，这些值以空格分隔，最后作为`foreach`函数的返回。所以`$(files)`的值是`a.o b.o c.o`。`n`是一个临时变量。

**if函数**：

```makefile
$(if <condition>,<then-part>)
# or
$(if <condition>,<then-part>,<else-part>)
```

`<condition>`参数是`if`的条件表达式，如果为非空字符串，那么这个条件就成立，函数返回值是`<then-part>`，否则`<else-part>`。如果`<else-part>`没有定义，则返回空字符串。

**call函数**：`call`函数是唯一一个可以用来创建心得参数化的函数，可以写一个非常复杂的表达式，在这个表达式中定义许多参数，然后可以用`call`函数向这个表达式传递参数，语法：

```makefile
$(call <expression>,<parm1>,<parm2>,<parm3>...)
```

当make执行这个函数时，`<expression>`参数中的变量，如`$(1) $(2) $(3)`等就会被`<parm1> <parm2> <parm3>`依次取代。而`<expression>`的返回值就是`call`函数的返回值。

```makefile
reverse = $(2) $(1)
foo = $(call reverse, a, b)
# foo 的值为 b a
```

**origin函数**：origin函数不像其它函数，它不操作变量的值，只是告诉你这个变量哪里来的：

```makefile
$(origin <variable>)
```

`variable`是变量的名字，不应该是引用，最好不要在`variable`中使用`$`字符。返回值有以下几种情况：

* `undefined`，从未被定义
* `default`，默认的定义，如`CC`变量
* `environment`，环境变量，并且当makefile被执行时，`-e`参数没有被打开
* `file`，这个变量被定义在makefile中
* `command line`，这个变量被命令行定义的
* `override`，变量被override指示符重新定义的
* `automatic`，是一个自动化变量

 这些信息对于我们编写Makefile是非常有用的，例如，假设我们有一个Makefile其包了一个定义文件Make.def，在 Make.def中定义了一个变量`bletch`，而我们的环境中也有一个环境变量`bletch`，此时，我们想判断一下，如果变量来源于环境，那么我们就把之重定义了，如果来源于Make.def或是命令行等非环境的，那么我们就不重新定义它。于是，在我们的Makefile中，我们可以这样写：

```makefile
ifdef bletch
ifeq "$(origin bletch)" "environment"
bletch = barf, gag, etc.
endif
endif
```

**shell函数**：`shell`函数的参数就是操作系统shell的命令，它把执行操作系统命令后的输出作为函数返回。于是，我们可以用操作系统命令及字符串处理命令`awk`，`sed`来生成一个变量：

```makefile
contents := $(shell cat foo)
files := $(shell echo *.c)
```

这个函数会新生成一个shell程序来执行命令，需要注意运行性能。

**控制make的函数**： make提供了一些函数来控制make的运行。通常，你需要检测一些运行Makefile时的运行时信息，并且根据这些信息来决定，你是让make继续执行，还是停止 ：

```makefile
$(error <text ...>)
```

 产生一个致命的错误，`<text ...>`是错误信息。注意，`error`函数不会在一被使用就会产生错误信息，所以如果你把其定义在某个变量中，并在后续的脚本中使用这个变量，那么也是可以的。例如：

```makefile
# example 1.
ifdef ERROR_001
$(error error is $(ERROR_001))
endif
# example 2.
ERR = $(error found an error!)
.PHONY: err
err: ; $(ERR)
```

示例1会在变量`ERROR_001`定义了后执行时产生`error`调用，示例2则在目录`err`被执行时才发生`error`调用。

```makefile
$(warning <text ...>)
```

这个函数很像`error`函数，只是它并不会让`make`退出，只是输出一段警告信息，然后make继续执行。
