# 学习使用 git-svn

[英文版官方文档：https://git-scm.com/docs/git-svn](https://git-scm.com/docs/git-svn)

## 名称

`git-svn` - 可以在 `Subversion`和`Git`仓库之间进行双向操作。

## 梗概

`git svn <命令> [选项] [参数]`

## 描述

`git svn` 是一种可以在 `Subversion` 和 `Git` 之间进行互操作的简单方式。是 `Subversion` 和 `Git` 之间操作的桥梁。

`git svn` 可以使用选项 `--stdlayout` 对遵循 `"trunk/branches/tags/"` 目录结构的标准Subversion仓库进行跟踪。它也可以分别使用选项 `-T`、`-t`、`-b` 来分别跟踪 `trunk`、`tags`、`branches`目录(参见下面对选项`init`和`clone`的相关介绍)

如果使用上面所描述的方式跟踪了一个Subversion仓库，就可以使用`fetch`命令从Subversion仓库中拉取更新，并且使用`dcommit`命令将git仓库的变更提交到Subversion仓库中去。

## 命令

### init

为 `git svn` 初始化一个带有附加元数据目录(`.git/`)的空Git仓库。这个命令需要后面跟上Subversion仓库的URL地址，或者是对应于Subverison仓库目录 `trunk(-T)/tags(-t)/branches(-b)` 的完整URL。

所要初始化的目标目录作为命令的第二个参数是可选的，如果不提供，默认初始化当前目录。


`-T<trunk_subdir> 或 --trunk=<trunk_subdir>`

`-t<tags_subdir> 或 --tags=<tags_subdir>`

`-b<branches_subdir> 或 --branches = <branches_subdir>`

`-s 或 --stdlayout` （该命令默认选项)

上面列出的是 `init` 命令的可选项，每一个都可以赋予指向仓库目录的相对路径 `(--tags=project/tags)` 或完整URL `(--tags=https://foo.org/project/tags)`。如果你的Subversion仓库把`tags`或`branches`放在多个路径下时，也可指定多个 `--tags` 或 `--branches` 选项。`--stdlayout` 表示你的Subversion仓库目录结构是标准的 `trunk/tags/branches` 形式，与另外的选项一起使用时，`--stdlayout`选项的优先级最低。

`--username=<user>`

对于通过 `http` `https` 或 `svn` 协议传输且需要身份认证的SVN仓库, 通过该选项指明用户名。对于通过 `svn+ssh` 协议传输的SVN仓库，需要把用户名信息拼接到URL上，例如: `svn+ssh://foo@svn.bar.com/project`


### fetch

仅从Subvesion仓库中取回其他人所做的修改的信息，但不对当前工作目录产生任何影响

`--log-window-size=<n>`

扫描Subversion历史时，每次取回的历史条目数，默认每次请求取回100条历史。

### clone

相当于运行了 `init` 和 `fetch` 命令。它会根据所提供的仓库URL地址的仓库名称，自动创建目录，如果该命令有第二个参数，则按照第二个参数指定的名称创建目录并在其中处理相关版本控制任务。它可以接受除了 `--fetch-all` 和 `--parent`这两个选项之外的任何其它可用于 `init` 和 `fetch`命令的选项。项目被从仓库克隆到本地后，可以使用 `fetch` 命令获得最近更新信息，但这不会影响当前工作目录，你需要使用 `rebase`命令把最新获取的更新信息应用到当前工作目录中。

### rebase

从 SVN 仓库中获得最新更新信息，并应用于当前本地工作目录。 它与 `svn update` 或 `git pull` 的不同之处在于，它会保证提交历史的线性特征，以便于之后使用命令 `git svn dcommit` 提交。这个命令使用时要求当前工作目录没有待提交的更改，也就是当前工作区必须是干净的。

`-l 或 --local`

不拉取仓库的更新数据，仅仅对本地仓库作 `rebase` 操作, 以此来线性化提交历史。

### dcommit

把本地修改推送到SVN仓库中去， 相当于纯svn环境下的 `svn commit` 的效果。

### branch

在SVN仓库中创建一个新分支

`-m 或 --message`

指定创建分支时的提交信息

`-t 或 --tags`
创创建一个tag版本。


### tag

相当于 `git svn branch -t`，即创建一个tag版本


### log

以svn的形式查看仓库历史提交日志。

### blame

显示一个文件内每一行最近一次被修改的版本号和修改人的名字。它不会对未提交过后文件内容起作用。

`--git-format` 
以 `git blame` 命令的形式显示相关信息。

### info

与纯svn环境下的 `svn info` 使用效果一样

### 基本使用示例：

跟踪并对一个Subversion托管仓库的 `trunk` 目录贡献，忽略 `tags` 和 `branches` 目录：

```
#克隆一个仓库, 类似于 git clone 命令：
	git svn clone http://svn.example/com/project/trunk
# 进入新克隆的本地仓库目录：
	cd trunk
# 现在你应该是master分支上，使用 git branch 来确认一下
	git branch
# 之后你可以像常使用 git 那样在本地仓库中进行更改或提交，注意这里的操作都只是对本地仓库来说我
	git commit ...
	
# 当你想要将本地的更改提交远程仓库时， 需要先使用 rebase 命令来同步其它你的工作到你本地仓库中：
	git svn rebase
	
# 同步远程仓库后可能会出现一些冲突，像正常使用git那样解决冲突后，再使用 git svn rebase 同步仓库，直到没有冲突后，再使用 git svn dcommit 提交你的更改到远程仓库中：
	git svn dcommit
# 你还可以把远程SVN仓库中设定的忽略文件设定导入到本地git仓库的相应设置文件中：
	git svn show-ignore >> ,git/info/exclude

```

另一个例子是跟踪并对一个完整的标准SVN仓库(包括 `trunk` `tags` 和 `branches`目录)贡献。

```
# 首先是克隆远程SVN仓库到本地git仓库:
	git svn clone http://svn.example.com/project --stdlayout --prefix svn/
# 如果远程SVN仓库使用的目录结构不是标准的 trunk/tags/branches 形式时，使用：
	git svn clone http://svn.example.com/project -T trunk_dir -b branch_dir -t tag_dir --prefix svn/
# 查看你所克隆的仓库的所有分支(branches)和标记（tags):
	git branch -r
# 在SVN远程仓库中创建一个新的分支：
	git svn branch new-branch-name
# 把git 的 master 分支(或其它分支)设定到 SVN 的 trunk 目录上：
	git reset --hard svn/trunk
#
# 一次你只能向 branch/tag/trunk 提交一次，使用 dcommit命令
#
# 之后你可以像常使用 git 那样在本地仓库中进行更改或提交，注意这里的操作都只是对本地仓库来说我
	git commit ...
	
# 当你想要将本地的更改提交远程仓库时， 需要先使用 rebase 命令来同步其它你的工作到你本地仓库中：
	git svn rebase
	
# 同步远程仓库后可能会出现一些冲突，像正常使用git那样解决冲突后，再使用 git svn rebase 同步仓库，直到没有冲突后，再使用 git svn dcommit 提交你的更改到远程仓库中：
	git svn dcommit
# 你还可以把远程SVN仓库中设定的忽略文件设定导入到本地git仓库的相应设置文件中：
	git svn show-ignore >> ,git/info/exclude
```

当远程SVN项目非常大时， 使用 `git svn clone` 来克隆项目会很耗时, 如果多个人想都使用 `git svn clone` 去克隆同一个远程仓库时，可以先认其中一人使用 `git svn clone` 把仓库克隆成git仓库放在服务器上，然后其它人就可以像克隆git远程仓库那样进行仓库克隆了。

```
# 由一个人把SVN远程仓库克隆到服务器上：
	ssh server "cd /pub && git svn clone http://svn.example.com/project [options ...]
	
# 然后其它人就可以克隆仓库到本地了：
	mkdir project 
	cd project
	git init
	git remote add origin server:/pub/project
	git config --replace-all remote.origin.fetch '+ref/remotes/*:refs/remotes/*'
	git fetch
# 防止之后再从 Git 服务器上取数据， 因为克隆下来后，我们之后都是要使用 git svn 来操作远程SVN仓库了， 不是之前的那个临时Git仓库服务器了
	git config --remove-section remote.origin
# 从之前获取的远程分支中创建一个对应本地分支进行跟踪：
	git checkout -b master FETCH_HEAD
# 本地初始化 git svn 仓库就和之前的两个示例一样了：
	git svn init http://svn.example.com/project [options ...]
# 从SVN远程仓库上拉下最新的数据：
	git svn rebase
```
