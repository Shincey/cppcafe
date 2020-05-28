# 输入输出

#### echo

`echo`命令可以显示文本行或变量，或者把字符串输入到文件。一般格式为：

```shell
$ echo string
```

`echo`命令有很多功能，其中最常用的是下面几个：

`\c` - 不换行

`\f` - 进纸

`\t` - 跳格

`\n` - 换行

如果希望提示符出现在输出的字符串之后，可以用：

```shell
$ echo "what's your name: \c" #输出字符串后跟一个光标
```

如果让光标移到下一行：

```shell
$ echo "what's your name:"
```

输出转义符及变量：

```shell
$ echo "\007your home directory is $HOME, you are connected `tty`"
your home directory is /Users/xxx, you are connected /dev/ttys000
```

如果是LINUXx系统必须使用`-n` 选项禁止输出后换行，`-e`选项使转义符生效：

```shell
$ echo -n "what's your name:"
$ echo "\007your home directory is $HOME, you are connected `tty`"
```

如果想把一个字符串输出到文件中，使用重定向符号`>`：

```shell
$ echo "Hello World" > file.txt
```

如果想输出引号，则需要通过转义字符`\`：

```shell
$ echo \"
```



#### read

可以使用`read`从键盘或文件的某一行读入信息，并将其赋值给一个变量。如果只指定一个比变量，则会把所有输入赋给该变量，直到遇到第一个文件结束符或回车，一般格式为：

```shell
$ read var1 var2 ...
```

在下面的例子中，只指定一个变量，它将被赋值直到回车之前的所有内容：

```shell
$ raed name
I am Ironman
$ echo $name
I am Ironman
```

指定两个变量：

```shell
$ read name surname
John Lemon Doe
$ echo $name
John
$ echo Doe
Lemon Doe
```



#### cat

`cat`是一个简单而通用的命令，可以用它显示文件内容，创建文件，还可以用来显示控制字符。`cat`会显示整个文件内容，可以配合管道及`more`命令分页显示：

```shell
$ cat myfile | more
```

其一般格式为：

```shell
$ cat [options] file1 ... file2 ...
```

`-v`选项显示控制字符。显示多个文件：

```shell
$ cat file1 file2 file3
```

可以利用重定向将内容输入一个新文件：

```shell
$ cat file1 file2 > newfile
```

或者：

```shell
$ cat > newfile
```

`cat`命令的输入是标准输入(键盘)，输入一些文字后，按`<CTRL-D>`结束输入。



#### 管道

可以通过管道把一个命令的输出传递给另一个命令的输入。管道用 `|` 表示。一般格式为：

```shell
$ command1 | command2
```

例如：

```shell
$ ls | grep hello.txt
hello.txt
```

`sed`、`awk`和`grep`都很适合用管道，特别是在简单的一行命令中，在下面例子，`who`命令的输出通过管道传递给`awk`命令，以便只显示用户名和所在的终端。

```shell
$ who | awk '{print $1"\t"$2}'
```

如果希望李处系统中所有文件系统，可以使用管道把`df`命令的输出传递给`awk`命令，`awk`显示其中的一列。还可以继续使用管道把`awk`的结果传递给`grep`命令，去掉最上面的题头filesystem：

```shell
$ df -k | awk '{print $1}' | grep -v "Filesystem"
```



#### tee

`tee`命令把输出的一个副本送到标准输出，另一个副本拷贝到相应的文件中。一般格式为：

```shell
$ tee -a files
```

`-a`表示追加到文件末尾。下面的例子，把结果输出到屏幕上，同时保存到`who.out`文件中：

```shell
$ who | tee who.out
```



#### 标准输入、输出和错误

当我们在Shell中执行命令的时候，每个进程都和三个打开的文件相联系，并用文件描述符来引用这些文件。即标准输入、输出和错误，对应的文件描述符分别是0、1、2。



#### 文件重定向

在执行命令时，可以通过文件重定向指定命令的标准输入、输出和错误。对标准错误进行重定向时必须使用文件描述符，对标准输入输出不是必须的。下表列出了常用的重定向组合：

| 命令                            | 解释                                                       |
| ------------------------------- | ---------------------------------------------------------- |
| `command > filename`            | 把标准输出重定向到一个新文件中                             |
| `command >> filename`           | 把标准输出重定向到一个文件中（追加）                       |
| `command 1 > filename`          | 把标准输出重定向到一个文件中                               |
| `command > filename 2>&1`       | 把标准输出和错误一起重定向到一个文件中                     |
| `command 2 > filename`          | 把标准输出重定向到一个文件中                               |
| `command 2 >> filename`         | 把标准输出重定向到一个文件中（追加）                       |
| `command >> filename 2>&1`      | 把标准输出和错误一起重定向到一个文件中（追加）             |
| `command < filename >filename2` | command命令以filename作为标准输入，以filename2作为标准输出 |
| `command < filename`            | command命令以filename作为标准输入                          |
| `command << delimiter`          | 从标准输入中读入，直到遇到`delimiter`分界符                |
| `command <&m`                   | 把文件描述符m作为标准输入                                  |
| `command >&m`                   | 把标准输出重定向到文件描述符m中                            |
| `command <&-`                   | 关闭标准输入                                               |



#### exec

`exec`命令可以用来替代当前shell，换句话说，并没有启动子shell。使用这一命令时任何现有环境都将会诶清除，并重新启动一个shell。其一般形式为：

```shell
$ extc command
```

其中`command`通常是一个shell脚本。


