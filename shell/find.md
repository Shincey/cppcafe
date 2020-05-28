# find

`find`可以遍历当前目录甚至整个文件系统来查找某些文件或目录。其一般格式为：

```shell
find pathname -options [-print -exec -ok]
```

`pathname` - 查找的目录路径，例如用`.`表示当前目录，`/`表示系统根目录。

`-print` - 将匹配的文件输出到标准输出。

`-exec` - 对匹配的文件执行该参数所给出的shell命令。相应的命令格式为`'comman' {} \;`，注意`{}`和`\;`之间的空格。

`-ok` - 和`-exec`作用相同，只不过以一种更安全模式来执行该参数给出的shell命令，每执行一个命令都需要用户确认。

`find`命令给出跟多选项或表达式：

`-name` - 按照文件名称查找文件。

`-perm` - 按照文件权限查找文件。

`-prune` - 使`find`不在当前指定目录查找，如果同时使用了`-depth`，则`-prune`被忽略。

`-user` - 按照文件属主来查找文件。

`-group` - 按照文件所属的组来查文件。

`-mtime -n +n` - 按照文件的更改时间来查找文件，`-n`表示文件更改时间距现在n天以内，+n表示文件更改时间距现在n天以前。

`-newer file1 ! file2` - 查找比file1新但比file2旧的文件。

`-type` - 查找某一类型的文件，如`b/d/c/p/l/f`。



```shell
find ~ -name "*.txt" -print
find . -name "*.txt" -print
find . -name "[A-Z]*" -print
```


