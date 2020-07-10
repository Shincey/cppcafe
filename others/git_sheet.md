# Git 常见命令速查

### 帮助

```shell
$ git help <command>
```

### 创建

克隆一个仓库：

```shell
$ git clone ssh://user@domain.com/repo.git
```

创建一个新仓库：

```shell
$ git init
```

### 本地改变

查看本地工作目录下文件的改变：

```shell
$ git status
```

查看跟踪的文件的改变内容：

```shell
$ git diff
```

将所有当前改动添加到下一次提交：

```shell
$ git add .
```

将文件`<file>`的所有改变添加到下一次提交：

```shell
$ git add -p <file>
```

提交所有本地追踪的文件的改动：

```shell
$ git commit -a
```

提交先前暂存的改动：

```shell
$ git commit
```

修改最近的提交（不要修改发布的提交）：

```shell
$ git commit --amend
```

### 提交历史

打印所有的提交，从最新的开始：

```shell
$ git log
```

打印特定文件的提交信息，随时间的变化：

```shell
$ git log -p <file>
```

打印是谁何时在特定文件中改动了什么内容：

```shell
$ git blame <file>
```

### 分支和标签

列出所有存在的分支：

```shell
$ git branch -av
```

跳转到 HEAD 分支：

```shell
$ git checkout <branch>
```

基于当前HEAD创建一个新的分支：

```shell
$ git branch <new-branch>
```

基于远程HEAD创建一个新的跟踪分支：

```shell
$ git checkout --track <remote/branch>
```

删除一个本地分支：

```shell
$ git branch -d <branch>
```

给当前提交标记一个tag：

```shell
$ git tag <tag-name>
```

### 更新和发布

列出所有远程连接：

```shell
$ git remote -v
```

打印一个远程分支的信息：

```shell
$ git remote show <remote>
```

新增一个新的远程仓库：

```shell
$ git remote add <shortname> <url>
```

从`<remote>`拉取所有改变，但不合并到当前HEAD中：

```shell
$ git fetch <remote>
```

拉取所有改变并合并到HEAD中：

```shell
$ git pull <remote> <branch>
```

发布本地改变到远程分支：

```shell
$ git push <remote> <branch>
```

删除一个远程仓库的分支：

```shell
$ git branch -dr <remote/branch>
```

发布标签：

```shell
$ git push --tags
```

### 合并和变基

合并分支`<branch>`到HEAD上：

```shell
$ git merge <branch>
```

将当前HEAD变基到`<branch>`上：

```shell
$ git rebase <branch>
```

中止一个变基：

```shell
$ git rebase --abort
```

在解决冲突后继续变基：

```shell
$ git rebase --continue
```

使用配置好的合并工具来解决冲突：

```shell
$ git mergetool
```

使用编辑器手动解决冲突，之后标记这些文件被解决：

```shell
$ git add <resolved-file>
$ git rm <resolved-file>
```

### 撤销

放弃本地工作目录的所有改变：

```shell
$ git reset --hard HEAD
```

放弃某特定文件的改变：

```shell
$ git checkout HEAD <file>
```

还原一个提交（通过提交之前的提交）：

```shell
$ git revert <commit>
```

重新设置HEAD指针到以特定commit，然后放弃那之后的所有改变：

```shell
$ git reset --hard <commit>
```

将所有改变保留为非暂存更改：

```shell
$ git reset <commit>
```

保留本地未提交的改动：

```shell
$ git reset --keep <commit>
```
