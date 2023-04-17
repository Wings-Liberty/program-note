#还没有复习 

此笔记不对Git进行介绍，而是记录git的命令行操作（分必须掌握和非必须掌握两部分）

# git中的常用名词

## 工作区，暂存区

被 git 管理的文件将分为以下几个区域

`工作区` 本地文件所处的状态

`暂存区`git 创建的一个临时保存文件的区域

`本地库`git 创建的本地仓库

`远程库`git 创建的远程仓库

代码提交流程：`工作区`--`缓存区`--`本地库`--`远程库`

`工作区`和`暂存区`只有一个状态

`本地库`有多个状态（多次提交记录，每次提交就是一个状态），每个分支都有自己独立`commit`记录，就好像每个分支都有一个独立的 “小本地库”，这些 “小本地库” 组成了本地库。

`工作区`会随着切换分支而改变其内容

`暂存区`只有一个，不会因为切换到不同的分支上而改变

在`master`分支下 add 的文件，也可以切换到`dev`分支后`commit`



> 分支只是指向某个commit对象的引用



## 远程分支和远程跟踪分支

`远程分支`远程库的分支

`远程跟踪分支`此分支在本地，是远程分支状态的引用

当进行一些对远程分支的操作时，会先检查本地的远程分支。

比如，使用 push 时先检查本地有没有对应的远程分支，而不是检查真正的远程库有没有此分支。因此需要不时 pull 或 fetch

## 标签

标签，给指定的`commit`绑定一个标签

##  远程库

git 可连接==多个远程库==，为此需要在添加远程库时为远程库起名字。默认是 'origin'

在`push`时，默认推送到 'origin'，也可用远程库的名字指定推送的目标远程库

# 必须掌握的指令

## --help

```shell
$ git --help
$ git add --help
```

前者会在终端打印帮助信息，后者自动打开浏览器打印所选命令的详细帮助信息（其html文件位置在`Git/mingw64/share/doc/git-doc/`）

## git管理文件夹

```shell
$ git init
```

## 工作区文件提交至缓存区

```shell
$ git add readme.txt
$ git add .
```

```shell
# 把指定文件从缓存区撤销
$ git rm --cached <file>...
```

## 缓存区文件提交至本地库

```shell
$ git commit -m'这里是本次提交的说明'
```

## 查看各区的提交情况

```shell
$ git status
```

红色表示没有add的文件，绿色表示没有`commit`的文件

## 查看工作区和暂存区的区别

```shell
$ git diff
```

## 查看commit日志

```shell
$ git log
```

## 工作区回退至到本地库的状态

抛弃对工作区，暂存区的修改。同时设置 push 时不上传超前的 commit（逻辑上丢弃了指定 commit 后的所有 commit）

```shell
$ git reset --hard head^
```

有三种回退模式，`--hard`，`--soft`，`--mixed`，后加 `commit id`或`head^`，`head`。有几个 ^ 就回到几次`commit`前

三种模式共同点为：`本地库`将丢掉指定版本前的所有`commit`记录。但不是真的丢失，只要记得`commit id`就还能回到指定版本。以下是不同点：

`--hard`					`工作区`，`暂存区`回到指定的版本

`--mixed(默认)`		`工作区`不变，`暂存区`回到指定版本

`--soft`						`工作区`，`暂存区`不变

## 工作区回到缓存区的状态

即：抛弃对工作区的修改

```shell
git checkout .
```

`git checkout <filename>`后面的参数是文件名，.表示当前目录，将工作区的当前目录和当前目录下所有文件回到缓存区的状态

==`git reset`和`git checkout `的区别是，前者重在操作`本地库`，后者重在操作`暂存区`==

## 连接远程库

分两种连接方式。1.克隆远程库到本地。2.本地 push 到远程库，以达到连接目的。这里说第2种

```shell
$ git remote add origin git@gitee.com:wingsofliberty/gitstudy.git
$ git push -u origin "master"
```

`git push`的选项`-u`的作用是本次 push 后，以后直接使用`git push`即可。命令后不需要再加`origin master`

如果远程库不存在此分支，远程库会自动创建此分支

## 创建分支

```shell
$ git checkout -b dev
```

```shell
$ git branch dev
$ git checkout dev
```

前者的一条命令等于后者执行的两条命令

切换分支不会引起工作区和缓存区内容的改变

## 合并分支

```shell
$ git merge dev
```

合并指定分支到当前分支。这里是将`dev`分支的东西合并到`master`分支

## stash 保存当前工作区状态

```shell
$ git stash
```

将`工作区`和`暂存区`的修改（不同于上次提交状态的部分）隐藏起来，工作树状态变为clean。工作区状态恢复至上次 commit

## 恢复工作现场

```shell
$ git stash pop
```

恢复的同时，删除`stash`

## 显示stash列表

```shell
$ git stash list
```

## 获取远程库的新分支

```shell
$ git fetch
$ git checkout -b feature origin/feature
```

## 拉取远程库代码

```shell
$ git pull	
```

更新`远程跟踪分支`并合并到当前分支

工作树不干净 pull 直接被会被拒绝（即 pull 前，先把工作树清理干净）

工作树是干净的

1. 远程库在本地库数据的基础上追加了一些数据

   这样就没有冲突，pull 将拉取代码并合并，还会自动提交到本地（`git log`中会有记录）

2. 远程库和本地库的数据不同，pull 后，工作树是干净的，但是`工作区`已为合并后的数据，需`手动处理冲突`--`git add `--`git commit`--`git push`

```shell
$ git fetch
```

更新`远程跟踪分支`。比如，别人在远程库添加了新的`commit`、添加了新的分支。fetch 都能获取到这些信息

## 提交代码到远程库

```shell
$ git push
```

前提：本地库远程库同步，然后发生一下几种情况

1. 其他人 push 了若干次，此时需更新`远程跟踪分支`
2. 自己修改原数据后 push（不光追加有新数据，还修改了原数据）

## 添加标签

```shell
$ git tag v1.0
$ git tag v1.0 1094adb
$ git tag -a v0.1 -m "version 0.1 released" 1094adb
```

默认给当前分支的最新的一次`commit`打上标签

也可指定给某次`commit`打标签，如：`$ git tag v0.9 f52c633`

常用选项：`-a`指定`commit id`，`-m`提交信息，`-d`指定要删除的本地标签

打完标签后，工作树不会有空变为忙



且 git push 不会主动上传标签，需要显式声明，如：

- 上传单个 tag `git push origin [tagname]`
- 上传所有 tag `git push [origin] --tags`

## 打印标签列表

```shell
$ git tag
```

## 恢复

```shell
$ git revert 12ljk
```

`git revert -n <commit id>`，不同于`git reset`

个人认为，`git revert`应该和`git cherry-pick`做对比，而不是和`git reset`做对比

在执行命令后，都会进入合并的操作中。在合并的对比新旧版本的流程中

后者显示的旧版本的数据是指定`commit`的提交信息和数据

前者显示的旧版本的数据是指定`commit`的提交信息和上一个`commit`的数据



# 非必须掌握的指令

**查看执行过的命令历史**

```shell
$ git reflog
```

**带有参数的提交日志**

```shell
$ git log --graph --pretty=oneline --abbrev-commit
```

**文件删除**

```shell
git rm readme.txt
```

好像没毛用。因为直接手动删除文件效果好像是一样的。也可能是我理解错误

如果误删文件，可用`git reset`回到文件没有被删除的版本

**查看所有分支和当前使用的分支**

```shell
$ git branch
```

**删除分支**

```shell
$ git branch -d dev
```

**切换分支**

```shell
$ git switch dev
$ git checkout dev
```

两者均可起到切换分支的作用，但前者语法上更为合理

**创建加切换分支**

```shell
$ git switch -c dev
$ git checkout -b dev
```

**合并策略**

```shell
$ git merge --no-ff -m "merge with no-ff" dev
```

**恢复工作区**

```shell
$ git stash apply
$ git stash apply stash@{1} 
```

默认打开`stash@{0} `此处可以指定。下述的`git stash drop`同理，也可指定要删除的`stash`

**删除隐藏部分**

```shell
$ git stash drop
```

**将某次提交合并到分支上**

```shell
$ git cherry-pick 4c805e2
```

`cherry-pick`后面的参数是`commit id`

此命令用于，修改分支 bug 时，多个分支都有此 bug。这条命令方便把修改 bug 的 commit 合并到各个分支上，而不是切换到不同分支上手动修复

**强行删除分支**

```shell
$ git branch -D feature01
```

上行删除分支，并丢掉分支上的修改

**显示远程库的详细信息**

```shell
$ git remote -v
```

**fetch**

```shell
$ git fetch
```

获取远程库的最新信息。例如：远程库创建了一个新的分支，想在本地创建并关联远程分支时，由于本地不知道远程库有了新的分支而无法关联。此时执行此命令后本地即可获取远程库的最新信息（例如远程库创建了一个新的分支）

**查看所有分支**

```shell
$ git branch -a
```

`-a`选项表示查看所有分支，不加`-a`看不到远程分支

**变基**

```shell
$ git rebase
```

解决使用`git log`打印提交线图有分叉显示不友好的问题。使用`git rebase`命令后，`git log`打印的数据将变成一条线

```shell
$ git rebase --abort
```

关闭变基模式

**展示某个标签的提交信息**

```shell
$ git show v1.0
```

`$ git show <tagname>`控制台将打印指定标签所绑定的`commit`的提交信息

**将标签推送至远程库**

```shell
$ git push origin --tags
```

默认`push`的时候不会提交标签，`--tags`表示推送所有未推送的标签

- 命令`git push origin `可以推送一个本地标签
- 命令`git push origin --tags`可以推送全部未推送过的本地标签
- 命令`git tag -d `可以删除一个本地标签
- 命令`git push origin :refs/tags/`可以删除一个远程标签



# Git 小知识



## 自动替换回车换行符

在 Windows 下，如果文本文件中每行的回车换行格式不是 CRLF，git 将在执行 add 操作时自动为其完成替换工作，将非 CRLF 格式换为 CRLF 格式，标兵给出以下提示

```
The file will have its original line endings in your working directory
warning: LF will be replaced by CRLF in xxx.xxx
```

可通过以下命令开关此自动转换

```shell
$ git config --global core.autocrlf [true|false]
```



