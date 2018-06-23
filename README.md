Git 是一个十分流行的版本控制系统，Git 和 SVN 区别在于，`SVN`使用**增量文件系统**，存储每次提交之间的差异。而 `git` 使用**全量文件系统**，存储每次提交的文件的全部内容(snapshot)。

git 保存的不是文件的变化或者差异，而是**一系列不同时刻的文件快照**。

	* [别名](#别名)
* [查看状态](#查看状态)
* [查看历史](#查看历史)
* [标签管理](#标签管理)
* [变基操作](#变基操作)
* [暂存变更](#暂存变更)
* [获取指定分支代码](#获取指定分支代码)
* [附录A 本地版本库](#appendix_a)
* [附录B 支持 http 方式 clone](#appendix_b)
* [附录C 将 git 上的项目 push 到自己的 repo](#appendix_c)
```
```
```
### 别名
跟 linux 别名命令 `alias` 一样，git 也可以设置别名。
```
# 给 checkout 设置别名 ck，git ck 等价于 git checkout
$ git config --global alias.ck checkout
$ git config -l | grep alias
alias.ck=checkout
```
```
```
remote: error:
上面这个配置比较麻烦，可以在创建 git 仓库时一次性解决。使用如下命令即可。
```
# --shared 参数
$ git init --bare --shared sample.git
```

```
```
```

# 删除分支: 若使用 -D，则强制删除
$ git branch -d bascker
```
**git 分支本质就是指向提交对象（校验和文件）的指针**，每次 commit，指针都会移动，指向最后一个 commit 对象。commit 流程图如下所示。
![commit_flow](assert/commit_flow.png)

特殊 HEAD 指针，指向当前分支。若当前在 master 分支，则 HEAD 指向 master 分支。若在 bascker 分支，则指向 bascker。
```
# 使用 --decorate 查看分支各分支指针所指向的对象
$ git log --oneline --decorate
aa403d4 (HEAD -> master, bascker) add b2.txt
...
```
如上，HEAD 指向 master 分支，master & bascker 均执行 aa403d4 提交对象。

### 分支合并
当在分支 bascker 上进行开发后，想把代码合入主干 master，就可以执行合并操作 `git merge`.
```
# master 分支执行
$ git merge bascker
```
若出现冲突，则解决完冲突，在继续 merge.
> Note：除第一次 commit 外，每次 commit 对象都会有一个父对象，多分支合并产生的 commit 对象有多个父对象。

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

# 使用 --oneline 可以将各个 commit 信息简单化
$ git log --oneline
aa403d4 add b2.txt
e4bf2fe add b1.txt
...
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

## 暂存变更
有时候我们还未开发代码完毕，需要切到其他分支，但又不想执行 commit 提交变更时，可通过`git stash`命令来暂存变更.
```
$ echo ccc > c.txt
$ git stash				# 暂存变更
$ git stash list
stash@{0}: WIP on bascker: aea2a1b add b2.txt

# 切换到其他分支, 执行某些操作
$ git checkout master
$ ...

# 切换回后，取回缓存
$ git stash pop

# 再执行 git stash list 显示没有暂存了
```

## 获取指定分支代码
有时候我们需要获取指定分支的代码，怎么办呢？可以通过`git fetch & git checkout` 来实现.
```
# 1.fetch 指定分支到本地
$ git fetch origin ORIGIN_BRANCE_NAME

# 2.将远程分支映射到本地
$ git checkout -b LOCAL_BRANCH_NAME origin/ORIGIN_BRANCE_NAME
```

<b id="appendix_a"></b>
## 附录A 本地版本库
若没有远程 git 仓库，没法很好的练习 git 操作咋办呢？没问题，我们可以使用本地版本库。通过在本地创建一个 git 仓库，可以将其 clone 到另一路径，然后进行 git 操作.
```
$ cd repo
$ git init sample.git

$ cd ..
$ git clone repo/sample.git
```

<b id="appendix_b"></b>
## 附录B 支持 http 方式 clone
想通过 HTTP 方式来进行 git clone 操作，是需要配置 httpd 服务的.
### 1 安装软件
为了使用 git-http-backend(支持 git 的 CGI 程序), 需要按照 git-core，apache 支持 git 就靠它
```
$ yum install -y git git-core httpd
```

### 2 配置 http 账户
```
# 修改仓库所有者，使得 apache 可以访问
$ chown -R apache:apache /workspace/gitrepo

# 创建 apache 账户 tanlang.
# -m：使用 MD5 算法对密码进行加密
# -c：创建一个加密文件
$ htpasswd -m -c /etc/httpd/conf.d/git.htpassd tanlang
New password:
Re-type new password:
Adding password for user tanlang

# 修改 git.htpasswd 文件的所有者与所属群组, 以及权限
$ chown apache:apache /etc/httpd/conf.d/git.htpassd
$ chmod 640 /etc/httpd/conf.d/git.htpassd
```
### 3 配置 httpd
修改 `/etc/httpd/conf/httpd.conf` 配置文件，末尾添加如下内容.
```
<VirtualHost *:80>
        ServerName 106.xxxx.xxx                                     # 服务器地址/域名
        SetEnv GIT_HTTP_EXPORT_ALL
        SetEnv GIT_PROJECT_ROOT /workspace/gitrepo/                 # 代码存放路径
        ScriptAlias /git/ /usr/libexec/git-core/git-http-backend/   # 以/git/开头的访问路径映射至git的CGI程序git-http-backend
        <Location />
                AuthType Basic
                AuthName "Git"
                AuthUserFile /etc/httpd/conf.d/git.htpasswd         # 验证用户帐户的文件
                Require valid-user
        </Location>
</VirtualHost>
```

### 4 启动服务
```
$ systemctl enable httpd
$ systemctl start httpd
```

### 5 测试
通过以上步骤，就可以使用 http 方式进行 clone 了.
```language
$ git clone http://***/git/sample.git
Cloning into 'sample'...
```

<b id="appendix_c"></b>
## 附录C 将 git 上的项目 push 到自己的 repo
以 spring-framework 为例，当 clone 下这个项目后，改动代码，提交变更, 然后 push 的话会 push 到 github 上去。那么如果我们只想将代码 push 到自己的 git 仓库呢？其实很简单，只需要添加自己的远程仓库地址，在 push 是选择 push 到我们自己的仓库就行.
```language
$ git remote add bascker ssh://IP/root/workspace/gitrepo/spring-framework.git
$ git remote -v
origin  https://github.com/spring-projects/spring-framework.git (fetch)
origin  https://github.com/spring-projects/spring-framework.git (push)
bascker ssh://IP/root/workspace/gitrepo/spring-framework.git (fetch)
bascker ssh://IP/root/workspace/gitrepo/spring-framework.git (push)
```
用小乌龟 push 时，选择目标 bascker 分支即可.
![appendix_c](assert/appendix_c.png)