# Makefile使用(5)-make运行

最简单的就是直接在命令行下输入make命令，make命令会找当前目录的makefile来执行，一切都是自动的。但有时只想让make重编译某些文件，而不是整个工程，而又有的时候有几套编译规则，想在不同的时候使用不同的编译规则。

**make退出码**：make命令执行后有三个退出码

* 0：表示成功执行
* 1：make运行时出现错误
* 2：如果使用make的`-q`选项，并且make使得一些项目不需要更新，那么返回2

**指定makefile**：GNU make搜索默认makefile的规则是：在当前目录下依次找三个文件--`GNUmakefile`、`makefile`、`Makefile`。当然也可以给make命令指定特殊名字的`makefile`。使用`-f`或`--file`：

```makefile
make -f name.mk
```

如果make的命令是，你不只一次地使用`-f`参数，那么所有指定的makefile将会被连载一起传递给make执行。

**指定目标**：默认make的最终目标是makefile中的第一个目标，而其它目标一般是由这个目标连带出来的。任何在makefile中的目标都可以被指定为终极目标，除了以`-`开头，或是包含了`=`的目标。make有个环境变量叫做`MAKECMDGOALS`，这个变量会存放你所指定的终极目标的列表，如果在命令行没有指定目标，这个变量时空值:

```makefile
sources = foo.c bar.c
ifneq ($(MAKECMDGOALS), clean)
include $(sources:.c=.d)
endif
```

这个例子是说只要输入的命令不是`make clean`，那么makefile会自动包含`foo.d`和`bar.d`这两个makefile。

建议的规范目标：

* `all`：这个伪目标是所有目标的目标，其功能一般是编译所有目标
* `clean`：这个伪目标是删除所有被make创建的文件
* `install`：这个伪目标是安装已经编译好的程序，其实就是把目标执行文件拷贝到指定目标中去。
* `print`：这个伪目标的功能是列出改变过的源文件
* `tar`：这个伪目标功能是把源程序打包备份，也就是一个tar文件
* `dist`：这个伪目标功能就是创建一个压缩文件，一般是把tar文件压成Z文件或者gz文件
* `TAGS`：这个伪目标功能是更新所有的目标，以备完整地重编译使用
* `check`和`test`：这两个伪目标一般用来测试makefile流程。

**检查规则**：有时不想让makefile中的规则执行起来，只是要检查一下命令或者执行的序列，可以使用以下make参数：

* `-n`、`--just-print`、`--dry-run`、`--recon`不执行参数，这些参数只是打印命令，不管目标是否更新，把规则和连带规则下的命令打印出来，但不执行。
* `-t`、`--touch`这个参数的意思就是把目标文件的时间更新，但不更改目标文件。也就是说，make假装编译目标，但不是真正的编译目标，只是把目标变成已编译过的状态。
* `-q`、`--qustion`这个参数的行为是找目标，也就是说，如果目标存在，那么其什么也不输出，当然也不会执行编译，如果目标不存在，其会打印一条错误信息。
* `-W <file>`、`--what-if=<file>`、`--assume-new=<file>`这个参数需要指定一个文件，一般是源文件或者依赖文件，make会根据规则推导来运行依赖于这个文件的命令，一般来说，可以和`-n`参数一同使用，来查看这个依赖文件所发生的的规则。

**make的参数**：GNU make 3.80 版的参数定义：

* `-b`、`-m`这两个参数的作用是忽略和其它版本make的兼容性
* `-B`、`--always-make`认为所有目标都需要更新
* `-C <dir>`、`--directory=<dir>`指定读取makefile的目录，如果有多个`-C`参数，make的解释是后面的路径以前面的作为相对路径，并以最后的目录为被指定目录。
* `--debug[=<option>]`输出make的调试信息，`<option>`取值：
  * `a`：all，输出所有调试信息
  * `b`：basic，只输出简单的调试信息，输出不需要重编译的目标
  * `v`：verbose，在`b`级别之上，输出信息包括哪个makefile被解析，不需要被重编译的依赖等。
  * `i`：implicit，输出所有的隐含规则
  * `j`：jobs，输出执行规则中命令的详细信息，如命令的PID、返回码等
  * `m`：makefile，输出make读取makefile，更新makefile，执行makefile等信息。
* `-d`：相当于`--debug=a`
* `-e`、`--environment-overrides` 指明环境变量的值覆盖makefile中定义的变量的值
* `-f=<file>`、`--file=<file>`、`--makefile=<file>` 指定需要执行的makefile
* `-h`、`--help`显示帮助信息
* `-i`、`--ignore-errors`在执行时忽略所有的错误
* `-I <dir>`、`--include-dir=<dir>`指定一个被包含makefile的搜索目标。可以使用多个“-I”参数来指定多个目录
* `-j [<jobsnum>]`、`--jobs[=<jobsnum>]`指同时运行命令的个数。如果没有这个参数，make运行命令时能运行多少就运行多少。如果有一个以上的`-j`参数，那么仅最后一个`-j`才是有效的。（注意这个参数在MS-DOS中是无用的）
* `-k`、`--keep-going`出错也不停止运行。如果生成一个目标失败了，那么依赖于其上的目标就不会被执行了。
* `-l <load>`、`--load-average[=<load]`、`--max-load[=<load>]` 指定make运行命令的负载。
* `-n`、`--just-print`、`--dry-run`、`--recon`仅输出执行过程中的命令序列，但并不执行。
* `-o <file>`、`--old-file=<file>`、`--assume-old=<file>`不重新生成的指定的`<file>`，即使这个目标的依赖文件新于它。
* `-p`、`--print-data-base`输出makefile中的所有数据，包括所有的规则和变量。这个参数会让一个简单的makefile都会输出一堆信息。如果你只是想输出信息而不想执行makefile，你可以使用`make -qp`命令。如果你想查看执行makefile前的预设变量和规则，你可以使用 `make –p –f /dev/null`。这个参数输出的信息会包含着你的makefile文件的文件名和行号，所以，用这个参数来调试你的 makefile会是很有用的，特别是当你的环境变量很复杂的时候。
* `-q`、`--question`不运行命令，也不输出。仅仅是检查所指定的目标是否需要更新。如果是0则说明要更新，如果是2则说明有错误发生。
* `-r`、`--no-builtin-rules`禁止make使用任何隐含规则。
* `-R`、`--no-builtin-variabes`禁止make使用任何作用于变量上的隐含规则。
* `-s`、`--silent`、`--quiet`在命令运行时不输出命令的输出。
* `-S`、`--no-keep-going`、`--stop`取消`-k`选项的作用。因为有些时候，make的选项是从环境变量`MAKEFLAGS`中继承下来的。所以你可以在命令行中使用这个参数来让环境变量中的`-k`选项失效。
* `-t`、`--touch`相当于UNIX的`touch`命令，只是把目标的修改日期变成最新的，也就是阻止生成目标的命令运行。
* `-v`、`--version` 输出make程序的版本、版权等关于make的信息。
* `-w`、`--print-directory`输出运行makefile之前和之后的信息。这个参数对于跟踪嵌套式调用make时很有用。
* `--no-print-directory`禁止`-w`选项。
* `-W <file>`、`--what-if=<file>`、`--new-file=<file>`、`--assume-file=<file>`假定目标`<file>`需要更新，如果和`-n`选项使用，那么这个参数会输出该目标更新时的运行动作。如果没有`-n`那么就像运行UNIX的`touch`命令一样，使得`<file>`的修改时间为当前时间。
* `--warn-undefined-variables` 只要make发现有未定义的变量，那么就输出警告信息。
