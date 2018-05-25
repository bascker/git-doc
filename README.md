# Git 操作文档
Git [文档地址](https://git-scm.com/book/zh/v2)

## 目录
* [安装](#安装)
* [配置](#配置)
* [初始化仓库](#初始化仓库)
* [获取项目](#获取项目)
* [更新代码](#更新代码)
* [分支管理](#分支管理)

## 安装
linux 下 git 安装很简单，apt-get 和 yum 直接装即可.
```language
$ yum install -y git
```

## 配置
使用`git config`进行信息配置, 其实是针对 config 文件进行配置
```language
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
```language
$ git config --global user.name "bascker"
$ git config --global user.email "xxx@xx.com"

# 查看 git 配置信息
$ git config --list
```

## 初始化仓库
使用`git init`可以创建一个 git 仓库。一般而言，纯粹用于版本控制的仓库，以 `.git`结尾。
```language
# 初始化一个 git 仓库
$ git init sample.git
```

一般创建完仓库后，我们需要执行如下命令来允许 push 操作.
```language
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
```language
$ git clone https://USER_NAME:USER_PASS@IP/sample.git
```
命令选项:
* `-b`: 获取指定分支，若不存在则报错
* `--depth`: 获取代码的深度。若指定为 1，则只会获取到远程仓库的 master 分支

## 更新代码
最常用/基本操作，就是在本地进行代码/文本编辑后，提交代码入库.
```language
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

```language
# 查看当前仓库的所有分支
$ git branch -a
* master
  remotes/origin/master

# 切换分支: -b 表示若无指定分支，则新建一个分支
$ git checkout -b bascker
Switch to a new branch 'bascker'
```

第一次 push 本地分支代码到远端时, 需要加 --set-upstream, 从而在远端创建一个分支.
```language
$ git push --set-upstream origin bascker
```
