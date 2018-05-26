# Git 操作文档
Git 是一个十分流行的版本控制系统，Git 和 SVN 区别在于，`SVN`使用**增量文件系统**，存储每次提交之间的差异。而 `git` 使用**全量文件系统**，存储每次提交的文件的全部内容(snapshot)。

Git [文档地址](https://git-scm.com/book/zh/v2)

## 目录
* [安装](#安装)
* [配置](#配置)
* [初始化仓库](#初始化仓库)
* [获取项目](#获取项目)
* [更新代码](#更新代码)
* [分支管理](#分支管理)
* [查看状态](#查看状态)
* [查看历史](#查看历史)
* [标签管理](#标签管理)
* [变基操作](#变基)

## 安装
linux 下 git 安装很简单，apt-get 和 yum 直接装即可.
```
$ yum install -y git
```

## 配置
使用`git config`进行信息配置, 其实是针对 config 文件进行配置
```
$ cat config
[core]
        repositoryformatversion = 0
        ...
[remote "origin"]
        url = https://...
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
        remote = origin
        merge = refs/heads/master
```

若加上`--global`, 则是全局配置, 比如配置 [user] 域下的 name 和 email 信息。
```
$ git config --global user.name "bascker"
$ git config --global user.email "xxx@xx.com"

# 查看 git 配置信息
$ git config --list
```

## 初始化仓库
使用`git init`可以创建一个 git 仓库。一般而言，纯粹用于版本控制的仓库，以 `.git`结尾。
```
# 初始化一个 git 仓库
$ git init sample.git
```

一般创建完仓库后，我们需要执行如下命令来允许 push 操作.
```
$ git config --gloabl receive.denyCurrentBranch ignore

# 若不执行, 可能会导致 push 失败, 报错
$ git push
remote: error: refusing to update checked out branch: refs/heads/master
remote: error: By default, updating the current branch in a non-bare repository
remote: error: is denied, because it will make the index and work tree inconsistent
remote: error: with what you pushed, and will require 'git reset --hard' to match
remote: error: the work tree to HEAD.
remote: error:
remote: error: You can set 'receive.denyCurrentBranch' configuration variable to
remote: error: 'ignore' or 'warn' in the remote repository to allow pushing into
remote: error: its current branch; however, this is not recommended unless you
remote: error: arranged to update its work tree to match what you pushed in some
remote: error: other way.
remote: error: To squelch this message and still keep the default behaviour, set
remote: error: 'receive.denyCurrentBranch' configuration variable to 'refuse'.
```

## 获取项目
git 支持 ssh/http(s)/ftp(s) 方式来获取远端代码. 通过`git clone`命令即可.
```
$ git clone https://USER_NAME:USER_PASS@IP/sample.git
```
命令选项:
* `-b`: 获取指定分支，若不存在则报错
* `--depth`: 获取代码的深度。若指定为 1，则只会获取到远程仓库的 master 分支

## 更新代码
最常用/基本操作，就是在本地进行代码/文本编辑后，提交代码入库.
```
# 跟踪当前目录下所有文件的变更
$ git add .
# 提交变更
$ git commit -m "描述信息"
# 推送到远端代码库
$ git push
```

对于 commit 而言, 注释最好以一行短句子(小于 50 个字符)作为开头, 然后空一行再把进行详细的注释,这样可以很方便的用工具把 commit 注释变成 email 通知，第一行作为标题，剩下的部分就作 email 正文.

## 分支管理
一般使用 `git branch` 和 `git checkout` 进行基础的分支管理操作.当然更进一步就是使用`git merge/fetch/rebase`命令了(这些命令的使用, 容后再议).

```
# 查看当前仓库的所有分支
$ git branch -a
* master
  remotes/origin/master

# 切换分支: -b 表示若无指定分支，则新建一个分支
$ git checkout -b bascker
Switch to a new branch 'bascker'

# 删除分支: 若使用 -D，则强制删除
$ git branch -d bascker
```

第一次 push 本地分支代码到远端时, 需要加 --set-upstream, 从而在远端创建一个分支.
```
$ git push --set-upstream origin bascker
```

## 查看状态
使用 git status 可以查看当前工作目录的状态，查看索引内容,哪些文件被暂存了(在Git索引中), 哪些文件被修改了但是没有暂存, 哪些文件没有被跟踪(untracked)。git 十分人性化的一点就在于每一个时间点，使用 git status 都有提示，告诉你下一步怎么做.
```
$ echo aaa > a.txt
$ git add a.txt
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   a.txt

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        .idea/
```

## 查看历史
使用`git log`可以看到历史提交信息，搭配`git show`使用效果更佳.
```
$ git commit -m "add a.txt"
$ git log
commit 3da5d8691146288de84a466cbfb123fed5171329
Author: bascker <xxx@aliyun.com>
Date:   Sat May 26 22:16:15 2018 +0800

    add a.txt

# 查看 commit 对象信息
$ git show 3da5d8691146288de84a466cbfb123fed5171329
commit 3da5d8691146288de84a466cbfb123fed5171329
Author: bascker <jp2317015793@aliyun.com>
Date:   Sat May 26 22:16:15 2018 +0800

    add a.txt

diff --git a/a.txt b/a.txt
new file mode 100644
index 0000000..3c8969b
--- /dev/null
+++ b/a.txt
@@ -0,0 +1 @@
+aaa
```
> git show 用于查看 git 对象信息, 每一次提交都是一个 commit 对象，包括分支、tag 也是对象.

git log 常见选项
* `--stat`: 显示在每个提交(commit)中哪些文件被修改了, 这些文件分别添加或删除了多少行内容
* `--pretty`: 格式化日志输出
* `--graph`: 图形化显示


## 标签管理
标签即 tag，通过`git tag`来进行标签管理，用于版本发布. 标签分为 2 种：
* 轻量标签：指向 commit 提交对象的引用
* 附注标签：生成独立的 tag 对象，拥有完整信息，建议使用

```
# 轻量标签
$ git tag v1.0.light

# 附注标签: 最简单方式就是使用 -a -m 参数
$ git tag -a v1.1 -m "附注标签"

$ git tag
v1.0.light
v1.1

# 查看标签对象
$ git show v1.1
Tagger: bascker <xxx@aliyun.com>
Date:   Sat May 26 22:31:35 2018 +0800

<E9><99><84><E6><B3><A8><E6><A0><87><E7><AD><BE>

commit 3da5d8691146288de84a466cbfb123fed5171329
Author: bascker <xxx@aliyun.com>
Date:   Sat May 26 22:16:15 2018 +0800

    add a.txt

diff --git a/a.txt b/a.txt
new file mode 100644
index 0000000..3c8969b
--- /dev/null
+++ b/a.txt
@@ -0,0 +1 @@
+aaa

# 使用 git checkout 可以切换到指定标签
$ git checkout v1.1
```

## 变基操作
一般都是使用`git merge`进行分支合并, 但还有一种更棒的方式就是`git rebase`变基操作.所谓变基，就是变更基线.
![rebase案例](assert/rebase.png)
如图所示，我们基于 master 创建了 merge 和 rebase 分支, 在 merge 和 rebase 分支上分别前进了 2 次, 与此同时, master 也前进了, 这时候我们分别在 merge 分支执行 `git merge master` 和在 rebasr 分支执行 `git rebase master` 利用 `gitk` 查看结果，可以看到 rebase 操作的分支历史比 merge 干净整洁很多.
> Note: gitk 是一个以图形的方式显示项目历史的命令

```
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: add r1.txt
Applying: add r2.txt
```
git rebase 操作会将本地的提交历史，由远及近逐一跟 master 进行合并。若发送冲突，则需要我们手工解决冲突后，执行 `git rebase --continue` 继续之前的操作即可.

git rebase 还拥有将多个 commit 合并成一个 commit 的逆天功能，这个功能十分有用.
```
# 将前 2 次提交进行 rebase
$ git rebase -i HEAD~2
```
这样就会进入命令模式, 然后根据提示进行操作即可
```
pick f8f5714 add b1.txt
pick aea2a1b add b2.txt

# Rebase ba6bea9..aea2a1b onto ba6bea9 (2 command(s))
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```