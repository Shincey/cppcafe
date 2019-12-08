# Makefile使用(2)-使用命令

**显示命令**：通常make会把其要执行的命令行在命令执行前输出到屏幕上，当我们用`@`字符在命令行前，这个命令将不会被输出。

```makefile
echo something...
# echo something...
# something...
@echo something...
# something...
```

**命令执行**：当依赖更新需要重新生成时，make会一条一条执行其后命令。如果想让上一条命令的结果应用在下一条命令时，应该将两个命令写在一行上并使用分号分隔这两个命令。

```makefile
# example 1.
exec:
	cd /home/shincey
	pwd
# example 2.
exec:
	cd /home/shincey; pwd
```

第一个例子中`cd`命令没有作用，`pwd`命令还是会打印当前`makefile`目录；第二个例子`cd`命令打印出`/home/shincey`。

make一般是使用环境变量SHELL中所定义的系统Shell来执行命令，默认情况下UNIX的标准Shell--`/bin/sh`来执行命令。但是MS-DOS下没有SHELL环境变量，可以指定，如果指定UNIX风格的目录形式，make首先会在SHELL所指定的路径中搜索命令解释器，如果找不到会在其当前盘符中的当前目录中搜索，如果再找不到，会在PATH环境变量中所定义的所有路径中搜索。MS-DOS中，如果你定义的命令解释器没有找到，其会给你的命令解释器加上诸如`.exe`, `.com`, `.bat`, `.sh`等后缀。

**命令出错**：一个命令运行完会返回成功，make检测到命令返回成功才会执行下一条命令，当规则所有命令成功返回后，这个规则算是成功完成了。如果一个规则中的某个命令出错（命令退出码非0），make会终止当前规则，这将可能终止所有规则执行。但是有时某些命令出错不表示就是错误的，如`mkdir`执行时，如果目录不存在，那么就会执行成功，如果目录已经存在了，就会出错。如果不希望`mkdir`出错终止规则的运行，可以忽略命令的出错。在命令行前加一个减号，标记为不管命令出错与否，都认为是成功的。

```makefile
clean:
	-rm -f *.o
```

还有一个全局的方法，给make加上`-i`或`--ignore-erros`参数，那么makefile中所有命令都会忽略错误。make还有一个参数`-k`或`--keep-going`，这个参数的意思是，如果规则中的命令出错了，那么就终止该规则的执行，但继续执行后续规则。

**嵌套执行make**：在大工程中，可能会把不同模块或不同功能源文件放在不同的目录中，可以为每一个目录都书写一个makefile文件，这有利于makefile文件更加简洁。如果在子目录subdir下也有个makefile，那么总控makefile文件可以这么写：

```makefile
subsystem:
	cd subdir && $(MAKE)
# 等价于：
subsystem:
	$(MAKE) -C subdir
```

以上两个例子都是先进入subdir目录，然后执行make命令。总控makefile变量可以传递到下级的makefile中（如果显示的声明），但是不会覆盖下层makefile定义的变量，除非指定`-e`参数。如果要传递和不传递变量到下级makefile中，可以这样声明：

```makefile
export <variable ...>;
unexport <variable ...>;
```

如果想要传递所有变量，只要一个`export`就可以，后面什么也不跟。有两个变量，`SHELL`和`MAKEFILES`不管你是否`export`都会被传递到下一层makefile中。`MAKEFILES`包含了make的参数信息，但是make中有几个命令并不往下传递：`-C`、`-f`、`-h`、`-o`、`-W`。如果不想往下传递参数，可以这样：

```makefile
subsystem:
	cd subdir && $(MAKE) MAKEFLAGS=
```

make的`-w`参数会在make的过程中输出一些信息，让你看到当前的工作目录的进入和离开。如果使用`-C`参数来指定make下层makefile时，`-w`会自动打开。如果参数有`-s`或`--slient`或`--no-print-directory`，那么`-w`总是失效的。

**定义命令包**：如果makefile中出现一些相同命令序列，可以将这些命令序列定义成一个变量。

```makefile
define run-yacc
yacc $(firstword $^)
mv y.tab.c $@
endef
```

在`define`和`endef`中的两行就是命令序列。第一行是运行Yacc程序，因为Yacc程序总是生成`y.tab.c`文件；第二行把这个文件改名字。使用命令包：

```makefile
foo.c : foo.y
	$(run-yacc)
```

在调用`run-yass`中，`$^`就是`foo.y`，`$@`就是`foo.c`。
