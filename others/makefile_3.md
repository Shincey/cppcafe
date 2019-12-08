# Makefile使用(3)-使用变量

makefile中定义的变量就像C/C++中的宏一样，在调用出原封不动的展开，不同的是可以在makefile中改变其值。变量命令和C/C++类似，不过可以数字开头。

**变量基础**：变量在声明的时候需要给予初值。使用形式如`$(VAR)`或`${VAR}`，如果需要使用真实的`$`字符，需要用`$$`来表示。

```makefile
foo = c
prog.o : prog.$(foo)
	$(foo)$(foo) -$(foo) prog.$(foo)
# 展开后
pro.o : prog.c
	cc -c prog.c
```

以上例子只是展示变量的替换效果。

**变量中的变量**：可以使用变量来定义变量。有两种方式，第一种简单的使用`=`，等号右侧的变量不一定是已定义的好的值，也可以是后面才定义的值：

```makefile
foo = $(bar)
bar = $(ugh)
ugh = Huh?
all:
	echo $(foo)
```

执行`make all`会打印`Huh?`。这种方法要避免产生循环递归。第二种方法是使用`:=`：

```makefile
x := foo
y := $(x) bar
x := later
# 等价于
y := foo bar
x := later
```

这种方法只能使用已经定义好的变量。若定义一个变量等于空格应该这样：

```makefile
nullstring :=
space := $(nullstring) # end of the line
```

`nullstring`是一个Empty变量，其中什么也没有，而`space`的值是一个空格。因为在操作符的右边是很难描述一个空格的，这里采用的技术很管用，先用一个Empty变量来标明变量的值开始了，而后面采用“#”注释符来表示变量定义的终止，这样，我们可以定义出其值是一个空格的变量。请注意这里关于“#”的使用，注释符“#”的这种特性值得我们注意，如果我们这样定义一个变量：

```makefile
dir := /foo/bar    # directory to put the frobs in
```

`dir`这个变量的值是`/foo/bar`后面还跟了4个空格，如果使用这样的变量来指定别的目录`$(dir)/file`就完蛋了。还有一个比较有用的操作符`?=`:

```makefile
FOO ?= bar
# 等价于
ifeq ($(origin FOO), undefined)
	FOO = bar
endif
```

其含义是，如果FOO没有被定义，那么FOO的值就是`bar`，如果被定义过了，则什么也不做。

看下面一段复杂例子，其中包括了make的函数、条件表达式和系统变量`MAKELEVEL`：

```makefile
ifeq (0, ${MAKELEVEL})
cur-dir		:= $(shell pwd)
whoami		:= $(shell whoami)
host-type	:= ${MAKE} host-type=${host-type} whoami=${whoami}
endif
```

`MAKELEVEL`意思是，如果我们的make有一个嵌套执行的动作，那么这个变量会记录当前makefile的调用层数。

**变量高级用法**：介绍两种高级用法，一是变量值的替换，可以替换变量中共有的部分，形式如`$(var:a=b)`或`${var:a=b}`，意思是把变量`var`中所有以`a`字串结尾的`a`替换成`b`字串，这里的结尾指空格或者结束符：

```makefile
foo := a.o b.o c.o
bar := $(foo:.o=.c)
```

这个示例中第二行的意思就是把`$(foo)`中所有以`.o`字串结尾替换为`.c`，所以`$(bar)`的值就是`a.c b.c c.c`。

二是把变量的值再当成变量：

```
x = y
y = z
a := $($(x))
```

`$(a)`的值就是`z`，很好理解，因为变量只是简单的替换。

**追加变量**：使用`+=`增加变量内容：

```makefile
objects = main.o foo.o bar.o utils.o
objects += another.o
# 和下面效果一样
objects = main.o foo.o bar.o utils.o
objects := $(ojbects) another.o
```

`$(objects)`的内容就变成了`main.o foo.o bar.o utils.o another.o`。如果变量之前没有定义`+=`会自动变成`=`，如果前面有定义变量，那么`+=`会继承于前次操作的赋值符。如果前一次是`:=`，那么`:=`作为其赋值符：

```makefile
variable := value
variable += more
# 等价于：
variable := value
variable := $(variable) more
```

但是如果这这种情况：

```makefile
variable = value
variable += more
```

由于前面使用的是`+`，所以`+=`也会以`=`来做赋值，可能会发生变量递归定义，但是make会自动为我们解决这个问题。

**override指示符**：如果有变量通过make的命令行参数设置的，那么makefile中对这个变量的赋值会被忽略。如果你想在makefile中设置这些参数，那么使用`override`指示符：

```makefile
override <variable>; = <value>;
override <variable>; := <value>;
# 还可以追加
override <variable>; += <more text>;
```

对于用`define`指示符定义多行变量，在`define`指示符前同样可以使用`override`指示符：

```makefile
override define foo
bar
endef
```

**多行变量**：使用`define`关键字定义的变量可以有换行，`define`后跟的是变量的名字，而重起一行定义变量的值。因为命令要以`tab`键开头，如果`define`定义的命令变量没有以`tab`键开头，那么make就不会当成命令。

```makefile
define two-lines
	echo foo
	echo $(bar)
endef
```

变量的值通常在一行通过赋值语句完成，但是在`define`指令中，中间的行都是变量值的一部分。上面等同于`two-lines = echo foo; echo $(bar)`，因为两个命令之间用分号隔开，其行为很接近两个分离的shell命令。然而，使用两个分离的行，意味着make请求shell两次，每次都在独立的子shell中运行。

**环境变量**：make运行时的系统环境变量可以在make开始运行时被载入到Makefile文件中，但是如果Makefile中已定义了这个变量，或是这个变量由make命令行带入，那么系统的环境变量的值将被覆盖。（如果make指定了`-e`参数，那么，系统环境变量将覆盖Makefile中定义的变量）

因此，如果在环境变量中设置了`CFLAGS`环境变量，那么我们就可以在所有的Makefile中使用这个变量了。这对于我们使用统一的编译参数有比较大的好处。如果Makefile中定义了`CFLAGS`，那么则会使用Makefile中的这个变量，如果没有定义则使用系统环境变量的值，一个共性和个性的统一，很像“全局变量”和“局部变量”的特性。

当make嵌套调用时，上层Makefile中定义的变量会以系统环境变量的方式传递到下层的Makefile 中。当然，默认情况下，只有通过命令行设置的变量会被传递。而定义在文件中的变量，如果要向下层Makefile传递，则需要使用export关键字来声明。

**目标变量**：前面我们所讲的在Makefile中定义的变量都是“全局变量”，在整个文件，我们都可以访问这些变量。当然，“自动化变量”除外，如“$<”等这种类量的自动化变量就属于“规则型变量”，这种变量的值依赖于规则的目标和依赖目标的定义。

当然，同样可以为某个目标设置局部变量，这种变量被称为“Target-specific Variable”，它可以和“全局变量”同名，因为它的作用范围只在这条规则以及连带规则中，所以其值也只在作用范围内有效。而不会影响规则链以外的全局变量的值：

```makefile
<target ...> : <variable-assignment>;
<target ...> : overide <variable-assignment>
```

`<variable-assignment>；`可以是各种赋值表达式。

```makefile
prog : CFLAGS = -g
prog : prog.o foo.o bar.o
        $(CC) $(CFLAGS) prog.o foo.o bar.o
prog.o : prog.c
        $(CC) $(CFLAGS) prog.c
foo.o : foo.c
        $(CC) $(CFLAGS) foo.c
bar.o : bar.c
        $(CC) $(CFLAGS) bar.c
```

 在这个示例中，不管全局的`$(CFLAGS)`的值是什么，在`prog`目标，以及其所引发的所有规则中（prog.o foo.o bar.o的规则），`$(CFLAGS)`的值都是`-g`。

 **模式变量**：在GNU的make中，还支持模式变量（Pattern-specific Variable），通过上面的目标变量中，我们知道，变量可以定义在某个目标上。模式变量的好处就是，我们可以给定一种“模式”，可以把变量定义在符合这种模式的所有目标上。make的“模式”一般是至少含有一个`%`的，所以，我们可以以如下方式给所有以`.o`结尾的目标定义目标变量：

```makefile
%.o : CFLAGS = -O
```

 同样，模式变量的语法和“目标变量”一样 。
