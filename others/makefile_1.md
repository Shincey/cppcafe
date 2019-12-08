# Makefile使用(1)-基本规则

```makefile
targets : prerequisites
	command
	...
# or
targets : prerequisites ; command
	command
	...
```

* targets 是目标文件，以空格分开，可以使用通配符。一般是一个文件，也可以是多个文件。
* command是命令行，如果不与`targets:prerequisites`在一行，必须以`tab`键开头，如果在一行可以用分号分隔。
* `prerequisites`是目标所依赖的文件，如果其中某个文件要比目标文件日期更加新的话，目标就会被认为过时，需要重新生成。可以使用`\`换行。

**通配符**：`wildcard`扩展通配符，`notdir`去除路径，`patsubst`替换通配符：

* `~`：`~/test`表示当前用户的$HOME目录下的test目录，`~user/test`表示用户user的宿主目录下的test目录。windows没有宿主目录，`~`根据环境变量HOME设定。
* `*`：匹配任意长度字符串，`*.c`表示所有后缀为c的文件。
* `%`：匹配大于等于0个若干字符，和`*`的区别我觉得在于`%.c`若匹配到`filename.c`则`%.c`使用过程就被替换成`filename.c`。
* `?`：仅代表单个字符。

```makefile
# a.列出一确定文件夹下的所有 .c 文件
objects := $(wildcard *.c)
# b.列出(a)中所有文件对应的 .o 文件
$(patsubst %.c, %.o, $(wildcard *c))
# c.由(a)(b)两步可以写出编译并链接所有 .c .o 文件
objects := $(patsubst %.c, %.o, $(wildcard *.c))
foo : $(objects)
	cc -o foo $(objects)
```

**文件搜寻**：makefile中特殊变量`VPATH`如果没有定义，make只会在当前目录中搜寻依赖文件；如果定义了，在当前目录找不到回到指定目录搜寻。指定的顺序就是搜索顺序，不过，当前目录永远是最高优先搜索的地方。

```makefile
# 到 src 和 ../headers 下搜索， 目录由 ： 分隔
VPATH = src : ../headers
```

另一个设置文件搜索路径的方法是使用`vpath`关键字，更加灵活，可以指定不同的文件在不同的搜索目录中。

* `vpath <pattern> <directories>`：为符合模式的文件指定搜索目录。
* `vpath <pattern>`：清除符合模式的文件的搜索目录。
* `vpath`：清除所有被设置的文件搜索目录。

`vpath`使用中的`<pattern>`需要包含`%`字符

```makefile
# 在 ../headers 下搜索所有 .h 结尾的文件，如果在当前目录下没有找到的话
vpath %.h ../headers
```

**伪目标**：是一个标签，并不是一个文件，make无法生成它的依赖关系和决定它是否要执行。可以用特殊标记`.PHONY`来显示指明一个目标是伪目标，避免和文件重名。常用比如写一个伪目标`clean`来清除生成的文件。不过也可以利用伪目标来一次生成多个目标。

```makefile
all : prog1 prog2 prog3
.PHONY : all
prog1 : prog1.o utils.o
	cc -o prog1 prog1.o utils.o
prog2 : prog2.o
	cc -o prog2 prog2.o
prog3 : prog3.o sort.o utils.o
	cc -o prog3 prog3.o sort.o utils.o
```

makefile中第一个目标别认为默认目标，默认目标的特性就是总会被执行。由于`all`是一个伪目标，不会生成文件，但是其他三个目标会被生成。

**多目标**：利用一些自动化变量的知识可以同时生成多个目标。

```makefile
aoutput boutput : text.g
	generate text.g -$(subst output,,$@) > $@
```

`-$(subst output,,$@)`表示执行一个makefile函数，函数名为`subst`，后面为参数。`$@`表示目标的集合，`$@`依次取出并执行目标。

**静态模式**：可以更加灵活地定义多目标规则

```makefile
<targets ...> : <target-pattern> : <prereq-pattern ...>
	<commands>
	...
```

* `targets`定义一系列目标文件，可以有通配符。
* `target-pattern`指明了`targets`的模式，也就是目标集模式。
* `prereq-patterns`是目标的依赖模式，它对`target-pattern`形成的模式再进行一次依赖目标的定义。例如：`target-pattern`为`%.o`，意思是目标集都是以`.o`结尾。而如果`<prereq-pattern>`定义成`%.c`，意思是对`target-pattern`所形成的目标集进行二次定义，计算方法是取`target-pattern`模式中的`%`(也就是去掉`.o`)，并为其加上`.c`结尾，形成的新集合。

```makefile
# example 1
bjects = foo.o bar.o
all : $(objects)
$(objects) : %.o: %.c
	$(CC) -c $(CFLAGS) $< $@
```

**自动生成依赖**：C/C++编译器都支持`-M`，即自动找寻源文件中包含的头文件，并生成依赖关系。

```makefile
# main.o : main.c def.h
cc -M main.c
```

如果使用的是GNU的C/C++编译器，得用`-MM`参数，不然`-M`会把一些标准的头文件也包含进来。但是要想在makefile里完成自动依赖生成，可以通过其它手段来实现。GNU组织建议把编译器为每个源文件自动生成的依赖关系放到一个文件中，为每一个`name.c`文件生成一个`name.d`的makefile文件，`.d`就存放对应`.c`文件的依赖关系。所以可以让make自动更新`.d`文件，并把其包含在主makefile文件中。例如：

```makefile
%.d: %.c
	@set -e; rm -f $@; \
	 $(CC) -M $(CPPFLAGS) $< > $@.$$$$; \
	 sed 's,\($*\)\.o[ :]*,\1.o $@ :, g' < $@.$$$$ > $@; \
	 rm -f $@.$$$$
```

这个规则意思就是所有的`.d`文件依赖于`.c`文件。 `rm -f $@`的意思是删除所有的目标，也就是`.d`文件；第二行的意思是为每个依赖文件`$<`，也就是`.c`文件生成依赖文件，`$@`表示模式`%.d`文件，`$*`表示`%`的内容，如果有一个C文件是`name.c`，那么`%`就是`name`， `$$$$`是一个随机编号，第二行生成的文件可能是`name.d.12345`；第三行使用`sed`命令做了一个替换，一般形式是`sed 's/pattern/new/g'`，这里用了逗号做基本语法的分隔，其中g表示所有匹配都替换，没有g只替换第一个匹配结果，/分隔符也可以换成其它符号，如`sed 's:/usr/local:/usr:g' mytest.txt`用`:`替换`/`；第四行就是删除临时文件。

以上规则做的事情就是在编译器生成的依赖关系中加入`.d`文件的依赖，即把依赖关系`main.o : main.c defs.h`转换成`main.o mian.d : main.c defs.h`。这样，`.d`文件就会自动更新，并会自动生成了。当然，在`.d`文件中加入的不只是依赖关系，包括生成的命令也可以一并加入，让每个`.d`文件都包含一个完整的规则。接下来把自动生成的规则放到我们的主makefile中。可以使用makefile的`include`命令引入别的makefile文件。

```makefile
sources = foo.c bar.c
include $(sources:.c=.d)
```

`$(sources:.c=.d)`的意思是把`$(sources)`中所有`.c`替换为`.d`。需要注意次序，最先载入的`.d`文件中的目标会成为默认目标。
