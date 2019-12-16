---
layout: post
title: "git-flow 插件使用手册"
subtitle: 'git 分支流程规范 & commit 描述规范'
author: "Song"
header-style: text
tags:
  - git
  - git-flow
  - git commit
  - 插件
---

## git-flow 插件简介

git flow 规范参考 [https://nvie.com/posts/a-successful-git-branching-model/](https://nvie.com/posts/a-successful-git-branching-model/)

git-flow 插件参考 [https://github.com/nvie/gitflow](https://github.com/nvie/gitflow) 

git-flow 插件安装 [https://github.com/nvie/gitflow/wiki/Installation](https://github.com/nvie/gitflow/wiki/Installation)

> 如: Mac 使用 Homebrew 安装 brew install git-flow

## 常用命令

### git flow 命令列表

```shell
➜  gitflowplugin git flow help
usage: git flow <subcommand>

Available subcommands are:
   init      Initialize a new git repo with support for the branching model.
   feature   Manage your feature branches.
   release   Manage your release branches.
   hotfix    Manage your hotfix branches.
   support   Manage your support branches. // 忽略，未使用
   version   Shows version information.

Try 'git flow <subcommand> help' for details.
```

### 查看版本号

```shell
➜  gitflowplugin git flow version
0.4.1
```

### 初始化

#### git flow init 命名列表

```shell
➜  gitflowplugin git flow init help
usage: git flow init [-fd]
```

#### git flow init -d

使用默认分支命名配置

```shell
➜  gitflowplugin git flow init -d
Initialized empty Git repository in /Users/chensongsong/git_work/gitflowplugin/.git/
Using default branch names.
No branches exist yet. Base branches must be created now.
Branch name for production releases: [master]
Branch name for "next release" development: [develop]

How to name your supporting branch prefixes?
Feature branches? [feature/]
Release branches? [release/]
Hotfix branches? [hotfix/]
Support branches? [support/]
Version tag prefix? []
```

#### git flow init [-f]

自定义分支命名

```shell
➜  gitflowplugin git flow init -f
Initialized empty Git repository in /Users/chensongsong/git_work/gitflowplugin/.git/
No branches exist yet. Base branches must be created now.
Branch name for production releases: [master] mymaster // 命名主要分支 master 为 mymaster
Branch name for "next release" development: [develop] // 直接回车使用默认 develop 命名

How to name your supporting branch prefixes? // 定义支持分支命名前缀
Feature branches? [feature/] myfeature- // 功能分支前缀定义为 myfeature-
Release branches? [release/] myrelease- // 发布分支前缀定义为 myrelease-
Hotfix branches? [hotfix/] // 回车使用默认 hotfix/ 前缀(注意带有/)
Support branches? [support/] mysupport- // 支持分支前缀定义为 mysupport-
Version tag prefix? [] test-tag- // tag 前缀为 test-tag- (默认无前缀)
```

> 若直接回车，使用默认命名, 建议直接使用插件默认命名

示例自定义前缀命名

```shell
依次执行下列命名
➜  gitflowplugin git:(develop) git flow feature start 0.1
➜  gitflowplugin git:(myfeature-0.1) git flow release start 0.1
➜  gitflowplugin git:(myrelease-0.1) git flow hotfix start 0.1.1
➜  gitflowplugin git:(hotfix/0.1.1) git branch -a
# 可以看到创建的所有分支命名
  develop
* hotfix/0.1.1
  myfeature-0.1
  mymaster
  myrelease-0.1

依次执行下述命令
➜  gitflowplugin git:(hotfix/0.1.1) git flow hotfix finish 0.1.1

0.1.1 // 配置的tag，但是不生效
#
# Write a message for tag:
#   test-tag-0.1.1 // 根据 init tag 前缀生成的tag命名
# Lines starting with '#' will be ignored.

执行下述命名
➜  gitflowplugin git:(develop) git tag --list
test-tag-0.1
test-tag-0.1.1
test-tag-0.2
```

操作显示主要分支配置的是分支名称，支持分支配置的是分支前缀

> 插件疑问，配置的tag不生效，最终命名是建议的 tag

### feature 分支

#### git flow feature 命令列表

```shell
➜  gitflowplugin git:(develop) git flow feature help
usage: git flow feature [list] [-v]
       git flow feature start [-F] <name> [<base>]
       git flow feature finish [-rFk] <name|nameprefix>
       git flow feature publish <name>
       git flow feature track <name>
       git flow feature diff [<name|nameprefix>]
       git flow feature rebase [-i] [<name|nameprefix>]
       git flow feature checkout [<name|nameprefix>]
       git flow feature pull <remote> [<name>]
```

#### git flow feature start [-F] <name> [<base>] 命令

1. [-F] 可选参数，若配置此参数，则必选 <name> 为  commit id; 若未配置，则 <name> 为创建分支名称
2. [<base>] 可选参数，若配置 -F ,则 <base>配置不生效，<base>总是 对应的 commit id(<name>)；若不配置 -F,则 <base> 为想 checkout 起点分支最后节点

配置 [-F] 时，`<base>` 不生效，且 `<name>` 为指定 `commit id`

```shell
➜  gitflowplugin git:(develop) git flow feature start -F b38e1fc master
Switched to a new branch 'feature/b38e1fc'

Summary of actions:
- A new branch 'feature/b38e1fc' was created, based on 'b38e1fc'
- You are now on branch 'feature/b38e1fc'

Now, start committing on your feature. When done, use:

     git flow feature finish b38e1fc
```

只有未配置 [-F] 时，`<base>` 才生效

```shell
➜  gitflowplugin git:(develop) git flow feature start 0.3 master
Switched to a new branch 'feature/0.3'

Summary of actions:
- A new branch 'feature/0.3' was created, based on 'master' // 基于master分支 checkout
- You are now on branch 'feature/0.3'

Now, start committing on your feature. When done, use:

     git flow feature finish 0.3

➜  gitflowplugin git:(feature/0.3)
```

#### git flow feature [list] [-v] 命令

[list] 可选参数，列出 feature 分支列表

```shell
➜  gitflowplugin git:(feature/0.3) git flow feature list
  0.1
  0.2
* 0.3
  b38e1fc
  d2086b8
```

[-v] 可选参数，列出 feature 分支列表,且说明分支信息

```shell
➜  gitflowplugin git:(feature/0.3) git flow feature -v
  0.1       (is behind develop, may ff)
  0.2       (may be rebased)
* 0.3       (is behind develop, may ff)
  b38e1fc   (may be rebased)
  d2086b8   (is behind develop, may ff)
```

> -v 参数自带 [list] 功能，故 git flow feature list -v 效果 和 git flow feature -v 相同

#### git flow feature diff [<name|nameprefix>] 命令

[<<name|nameprefix>>] name: 指定分支，nameprefix：feature分支后缀 。显示当前分支和选定分支区别

```shell
➜  gitflowplugin git:(feature/d2086b8) git flow feature diff 0.2
diff --git a/sss.txt b/sss.txt
new file mode 100644
index 0000000..e9e6383
--- /dev/null
+++ b/sss.txt
@@ -0,0 +1 @@
+sss
```

#### git flow feature pull <remote> [<name>] 命令

从远程分支拉取

- 若设置 [<name>] ，且 <name> 远程分支存在，若本地仓库不存在 <name> 分支，则下载此远程分支，且 checkout 到此 （未跟踪此远程分支）
- 若设置 [<name>] ，且 <name> 远程分支存在，若本地仓库存在 <name> 分支，则拉取远程分支新的 commit (当前必须在对应的 <name> 分支，否则拉取失败)
- 若设置 [<name>] ，且 <name> 远程分支不存在，则报错
- 若未设置 [<name>] ，则拉取当前分支对应的远程分支新的 commit ，若当前分支对应的远程分支不存在，则报错

```shell
➜  gitflowplugin git:(feature/0.2) git flow feature pull origin 0.2 // 设置其他分支会报错
Pulled origin's changes into feature/0.2.
```

> 若要拉取的远程分支本地不存在，则下载到本地且 checkout, 但并未跟踪拉取的远程分支

#### git flow feature track <name> 命令

下载远程分支，若本地无此远程分支，则下载下来，且跟踪远程分支；若本地有此对应分支，则为无效操作

```shell
➜  gitflowplugin git:(develop) git flow feature track 0.2
Branch 'feature/0.2' set up to track remote branch 'feature/0.2' from 'origin'.
Switched to a new branch 'feature/0.2'

Summary of actions:
- A new remote tracking branch 'feature/0.2' was created
- You are now on branch 'feature/0.2'
```

> 下载的本地分支跟踪远程分支

#### git flow feature publish <name> 命令

push 指定 feature 分支到远程仓库

```shell
➜  gitflowplugin git:(feature/0.2) git flow feature publish 0.3
Total 0 (delta 0), reused 0 (delta 0)
remote:
remote: Create a pull request for 'feature/0.3' on GitHub by visiting:
remote:      https://github.com/songsongbrother/gitflowplugin/pull/new/feature/0.3
remote:
To github.com:songsongbrother/gitflowplugin.git
 * [new branch]      feature/0.3 -> feature/0.3
Switched to branch 'feature/0.3'
Your branch is up to date with 'origin/feature/0.3'.

Summary of actions:
- A new remote branch 'feature/0.3' was created
- The local branch 'feature/0.3' was configured to track the remote branch
- You are now on branch 'feature/0.3'
```

> 若远程分支存在，则当前操作无效，此命令非 push commit 到远程

#### git flow feature checkout [<name|nameprefix>] 命令

checkout 到指定的 feature 分支(必须存在)

```shell
➜  gitflowplugin git:(feature/0.2) git flow feature checkout 0.4
Switched to branch 'feature/0.4'
```

#### git flow feature finish [-rFk] <name|nameprefix> 命令

合并 <name> 到本地的 develop 分支,合并成功后删除该分支

- -r 在合并到develop分支时,使用 rebase 机制,而不是merge(慎重操作)
- -F 在执行finish操作前,先执行fetch,从远程仓库下载更新(推荐)
- -k 执行完finsh后,保留feature分支,即不删除分支

```shell
➜  gitflowplugin git:(feature/0.2) git flow feature finish 0.4
Switched to branch 'develop'
Your branch is up to date with 'origin/develop'.
Auto-merging test.txt
Merge made by the 'recursive' strategy.
 sss.txt  | 8 ++++++++
 test.txt | 2 ++
 2 files changed, 10 insertions(+)
 create mode 100644 sss.txt
Deleted branch feature/0.4 (was 1f0cbfb).

Summary of actions:
- The feature branch 'feature/0.4' was merged into 'develop'
- Feature branch 'feature/0.4' has been removed
- You are now on branch 'develop'
```

> 根据develop分支和 <name> 分支之间的提交的个数决定是否设置--no--ff(测试5个提交就执行)

#### git flow feature rebase [-i] [<name|nameprefix>] 命令

以develop分支作为upstream,对指定的feature分支执行rebase操作

- -i 等价rebase -i

```shell
➜  gitflowplugin git:(feature/0.2) git flow feature rebase -i 0.2
Will try to rebase '0.2'...
Successfully rebased and updated refs/heads/feature/0.2.
```

### release 分支

#### git flow release 命令列表

```shell
➜  gitflowplugin git:(develop) git flow release help
usage: git flow release [list] [-v]
       git flow release start [-F] <version>
       git flow release finish [-Fsumpk] <version>
       git flow release publish <name>
       git flow release track <name>
```

#### git flow release [list] [-v] 命令

同 feature 命令

```shell
➜  gitflowplugin git:(release/0.2) git flow release -v
* 0.2   (no commits yet)
```

#### git flow release start [-F] <version>

同 feature start 命令, 创建 release 分支，强制基于 develop 分支，-F 参数指定commit(必须为develop分支的节点)，默认为 develop 的 HEAD

```shell
➜  gitflowplugin git:(develop) git flow release start 0.2
Switched to a new branch 'release/0.2'

Summary of actions:
- A new branch 'release/0.2' was created, based on 'develop'
- You are now on branch 'release/0.2'

Follow-up actions:
- Bump the version number now!
- Start committing last-minute fixes in preparing your release
- When done, run:

     git flow release finish '0.2'
```

基于指定 commit，此 commit 必须为develop 分支节点

```shell
  gitflowplugin git:(develop) git flow release start -F 9655879
Switched to a new branch 'release/9655879'

Summary of actions:
- A new branch 'release/9655879' was created, based on '9655879'
- You are now on branch 'release/9655879'

Follow-up actions:
- Bump the version number now!
- Start committing last-minute fixes in preparing your release
- When done, run:

     git flow release finish '9655879'
```

> 此时此刻只能存在一个 release 分支，否则创建报错

#### git flow release publish <name> 命令

同 feature 命令，将 release 分支push到远程仓库

```shell
➜  gitflowplugin git:(release/0.2) git flow release publish 0.2
Total 0 (delta 0), reused 0 (delta 0)
remote:
remote: Create a pull request for 'release/0.2' on GitHub by visiting:
remote:      https://github.com/songsongbrother/gitflowplugin/pull/new/release/0.2
remote:
To github.com:songsongbrother/gitflowplugin.git
 * [new branch]      release/0.2 -> release/0.2
Already on 'release/0.2'
Your branch is up to date with 'origin/release/0.2'.

Summary of actions:
- A new remote branch 'release/0.2' was created
- The local branch 'release/0.2' was configured to track the remote branch
- You are now on branch 'release/0.2'
```

#### git flow release track <name> 命令
同 release 命令，下载远程分支，若本地无此远程分支，则下载，且跟踪远程分支；若本地有此对应分支，则为无效操作

```shell
➜  gitflowplugin git:(develop) git flow release track 0.2
Branch 'release/0.2' set up to track remote branch 'release/0.2' from 'origin'.
Switched to a new branch 'release/0.2'

Summary of actions:
- A new remote tracking branch 'release/0.2' was created
- You are now on branch 'release/0.2'
```

> 本地分支会跟踪远程分支

#### git flow release finish [-Fsumpk] <version> 命令
完成由version指定的release分支的开发,将其合并到develop和master分支,并为该分支创建一个tag

- -F 执行操作前先执行fetch (推荐)
- -s 对新建的tag签名
- -u 签名使用的GPG-key
- -m 使用指定的注释作为tag的注释
- -p 当操作结束后,push到远程仓库中 (推荐)
- -k 保留 release 分支
- -n 不创建tag (help命令有bug)

```shell
➜  gitflowplugin git:(release/0.4) git flow release finish 0.4
Switched to branch 'master'
Your branch is ahead of 'origin/master' by 26 commits.
  (use "git push" to publish your local commits)
Merge made by the 'recursive' strategy.
 test.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
Switched to branch 'develop'
Your branch is ahead of 'origin/develop' by 2 commits.
  (use "git push" to publish your local commits)
Merge made by the 'recursive' strategy.
 test.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
Deleted branch release/0.4 (was 9b68c08).

Summary of actions:
- Latest objects have been fetched from 'origin'
- Release branch has been merged into 'master'
- The release was tagged '0.4'
- Release branch has been back-merged into 'develop'
- Release branch 'release/0.4' has been deleted
```

### hotfix 分支

#### git flow hotfix 命令列表

```shell
➜  gitflowplugin git:(develop) git flow hotfix help
usage: git flow hotfix [list] [-v]
       git flow hotfix start [-F] <version> [<base>]
       git flow hotfix finish [-Fsumpk] <version>
```

#### git flow hotfix [list] [-v] 命令
同 feature 和 release 命令

#### git flow hotfix finish [-Fsumpk] <version> 命令
同 feature 和 release 命令

#### git flow hotfix start [-F] <version> [<base>] 命令
同 release 命令, 创建 hotfix 分支，强制基于 master 分支，-F 参数指定commit(必须为 master 分支的节点)，默认为 master 的 HEAD

```shell
➜  gitflowplugin git:(develop) git flow hotfix start 0.5.1 master // 只有master 生效，否则报错
Switched to a new branch 'hotfix/0.5.1'

Summary of actions:
- A new branch 'hotfix/0.5.1' was created, based on 'master'
- You are now on branch 'hotfix/0.5.1'

Follow-up actions:
- Bump the version number now!
- Start committing your hot fixes
- When done, run:

     git flow hotfix finish '0.5.1'
```

> 此时此刻只能存在一个 hotfix 分支，否则创建报错

## git-flow 插件使用总结

- git-flow 插件只是 git 命令的集合，并不创建新的命令
- 主要分支生命周期无限长，支持分支最终都会被删除
- git-flow 命令只适合在创建支持分支，push，最后发布使用，开发过程中的提交需要用原生的git 命令
- master 和 develop 分支只接受合并，不接受提交，故可锁定
- 私有化分支和重大分支版本(如v1升级到v2)，切勿使用 git-flow 命令

## 分支命名规范

### feature

分两种情况, 一种是功能分支声明周期无限长，作为特殊的发布版本存在, 比如私有化，命名为feature/category-description；

一种是会合并到develop,作为功能开发分支。命名为feature/versions，versions为当时分裂出来的版本号

例: 私有化 feature/privatization-taiping

例: 新功能 feature/0.9.0, feature/0.8.8

### hotfix

命名规范：hotfix/versions

例：hotfix/0.9.1

### release

命名规范：release/version

例：release/0.9.0(要更新的version)

## commit 描述规范

描述规范：[product][branch][function][description]

- [product]：项目名称，如：Onepass
- [branch]：分支名称，如：hotfix/4.1.1
- [function]：更新功能，update(功能更新)/bugfix(bug修复)/prepare(发布准备)
- [description：提交描述，如：更新日志策略 & 更新网络内核为OKhttp

例：[Sensebot][feature/4.1.0][update][更新日志策略 & 更新网络内核为OKhttp]

> 若仓库只存在一个产品，则[product]可以省略。如：onepass 和 onelogin 在一个仓库的情况，[product]说明不能省略

