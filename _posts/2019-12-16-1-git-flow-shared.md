---
layout: post
title: "git flow 分享"
subtitle: '标准化 git flow 流程'
author: "Song"
header-style: text
tags:
  - git
  - git-flow
---

## 介绍
git flow 是一种工作流，核心是分支策略和发布管理，作者 Vincent Driessen

<img src="https://song-dev.github.io//img/in-post/post-git/gitflow.png" alt="gitflow" style="zoom:60%;" />

[https://nvie.com/posts/a-successful-git-branching-model/](https://nvie.com/posts/a-successful-git-branching-model/) <br>
[http://www.ruanyifeng.com/blog/2015/12/git-workflow.html](http://www.ruanyifeng.com/blog/2015/12/git-workflow.html)

## 主要分支
中央仓库拥有两个主要分支，其具有无限生命周期

- master
- develop

<img src="https://song-dev.github.io//img/in-post/post-git/master-develop.png" alt="master-develop" style="zoom:60%;" />

origin/master是源代码反映生产就绪状态的主要分支 。

origin/develop也是主要的分支，其始终反映了下一版本中最新交付的开发更改状态。有些人称之为“整合分支”。

当develop分支中的源代码开发完成，应该合并回master，然后使用版本号进行标记。因此，将更改合并回master时，生成一个新的生产版本。对此应该处理流程非常严格，因此从理论上讲，可以使用脚本在每次提交时自动构建和推出我们的软件到我们的生产服务器 master。

## 支持分支

与主要分支不同，支持分支的寿命有限，因为最终会被删除。可能使用的支持分支有：

- feature
- release
- hotfix

这些分支中都有特定的目的，并且必须遵守哪些分支是其起始分支，以及哪些分支是其合并目标分支的严格规定。

### 功能分支(feature)

用于为即将发布或将来的版本开发新功能。在开始开发新功能时，此功能的目标版本可能不确定。feature 分支的本质是，只要功能处于开发阶段，它就会存在，但最终会被合并回develop（以便将新功能添加到即将发布的版本中）或丢弃（在实验令人失望的情况下)。功能分支通常仅存在于开发人员本地仓库中，而不存在于origin(存疑)。

1. 可能从develop checkout
2. 必须合并回 develop 分支
3. 分支命名 feature/*


#### 创建功能分支
在开始处理新功能时，从develop分支checkout。

```shell
$ git checkout -b feature/myfeature develop
 切换到新分支“feature/myfeature”，基于develop分支
```

#### 在开发中加入完成的功能
完成的功能可能会合并到develop分支中，以确保将其添加到即将发布的版本中：

```shell
$ git checkout develop
 切换到分支'develop'
$ git merge --no-ff feature/myfeature
 更新ea1b82a..05e9557 
（更改摘要）
$ git branch -d feature/myfeature
 已删除分支feature/myfeature（为05e9557）。
$ git push origin develop
```

<img src="https://song-dev.github.io//img/in-post/post-git/gitflow-merge.png" alt="master-develop" style="zoom: 60%;" />

该--no-ff标志使合并始终创建新的提交对象，即使可以使用快进执行合并。这样可以避免丢失有关功能分支历史存在的信息，并将所有一起添加功能的提交组合在一起。

### 发布分支(release)

release 准备新的生产版本，修复小错误并为发布准备元数据（版本号，构建日期等）。develop反映开发中所有状态，故发布后必须要合并回develop分支中。即将发布的版本被分配了一个版本号，直到此刻，develop分支反映“下一个版本”的变化，但不清楚“下一个版本”最终是否会变为0.3或1.0，直到发布上线。

1. 可能从develop checkout
2. 必须合并回 develop 和 master 分支
3. 分支命名 release/*

#### 创建发布分支
发布分支是从develop分支创建的。例如，版本1.1.5是当前的生产版本，我们即将推出一个大版本。develop分支为“下一个版本”做好了准备，我们已经决定这将成为版本1.2（而不是1.1.6或2.0）。

```shell
$ git checkout -b release/1.2 develop
 切换到新分支“release/1.2” 
$ ./bump-version.sh 1.2
 文件修改成功，版本提升到1.2。
$ git commit -a -m “Bumped version number to 1.2” 
[release/1.2 74d9424] Bumped version number改为1.2 
1个文件，1个插入（+），1个删除（ - ）
```

在创建新分支并切换到它后，我们会修改版本号，然后提交。这个新的分支可能存在一段时间，直到发布后。在此期间，可以在此分支中错误修复（而不是在develop分支上）。严禁在此分支添加大型新功能。它们必须合并develop，然后再等待下一个大版本。

#### 完成发布分支
当release分支准备发布时，首先合并到master（因为每次提交master都是按照定义的新版本）。接下来，master必须标记该提交，以便将来参考此历史版本。最后，需要将发布分支上所做的更改合并回develop，以便将来的版本也包含这些错误修复。

Git的前两个步骤：
```shell
$ git checkout master
 切换到分支'master' 
$ git merge --no-ff release/1.2
 由递归合并而成。
（更改摘要）
$ git tag -a 1.2
```

> 该版本现已完成，并标记以供将来参考。

将更改合并到develop分支。

```shell
$ git checkout develop
 切换到分支'develop' 
$ git merge --no-ff release/1.2
 由递归合并而成。
（变更摘要）
```

已经完成了，删除release分支

```shell
$ git branch -d release/1.2
 删除了分支版本1.2（ff452fe）。
```

### 修补程序分支(hotfix)

hotfix 分支非常像release分支，因为也是准备新的生产版本。当必须立即解决生产版本重大问题时候，从对应Tag checkout 修补。

<img src="https://song-dev.github.io//img/in-post/post-git/gitflow-hotfix.png" alt="gitflow-hotfix" style="zoom:60%;" />

> 重大分支版本需要预留hotfix维护

1. 可能从 master(对应的tag节点) checkout 分支
2. 必须合并回 develop 和 master 分支
3. 分支命名 hotfix/*

> 非重大bug，可以直接在 release 分支或者 feature 分支提交

#### 创建修补程序分支
从master分支创建hotfix分支。例如，假设版本1.2是当前正在运行的生产版本，并且由于严重的错误而导致麻烦。但是develop分支仍在开发阶段，我们就可在hotfix分支修复问题。

```shell
$ git checkout -b hotfix/1.2.1 master
 切换到新分支“hotfix/1.2.1” 
$ ./bump-version.sh 1.2.1
 文件修改成功，版本提升到1.2.1。
$ git commit -a -m “Bumped version number to 1.2.1” 
[hotfix-1.2.1 41e61bb] Bumped version number to 1.2.1 
1个文件改变了，1个插入（+），1个删除（ - ）
```
> 切记版本号碰撞


然后修复错误，并在一个或多个单独的提交中提交

```shell
$ git commit -m “修复了严重生产问题” 
[hotfix-1.2.1 abbe5d6]修复了严重生产问题
5个文件发生了变化，32个插入（+），17个删除（ - ）
```

#### 完成hotfix分支
完成后，需要将错误修复合并回master，但也需要合并回develop，以保证错误修复程序也包含在下一个版本中。这与release分支的完成方式完全相似。
```shell
合并且tag
$ git checkout master
 切换到分支'master' 
$ git merge --no-ff hotfix/1.2.1
 由递归合并。
（更改摘要）
$ git tag -a 1.2.1

删除临时分支
$ git branch -d hotfix/1.2.1
 删除了分支修补程序-12.2.1（是abbe5d6）。
```

## 插件使用

[https://github.com/nvie/gitflow](https://github.com/nvie/gitflow)<br>
[https://jeffkreeftmeijer.com/git-flow/](https://jeffkreeftmeijer.com/git-flow/)

初始化仓库

```shell
➜  TestGitFlow git flow init -d
Initialized empty Git repository in /Users/chensongsong/git_work/TestGitFlow/.git/
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

创建 feature/0.1 分支

```shell
➜  TestGitFlow git:(develop) git flow feature start 0.1
Switched to a new branch 'feature/0.1'

Summary of actions:
- A new branch 'feature/0.1' was created, based on 'develop'
- You are now on branch 'feature/0.1'

Now, start committing on your feature. When done, use:

     git flow feature finish 0.1
```

```shell
➜  TestGitFlow git:(feature/0.1) git flow feature finish 0.1
Switched to branch 'develop'
Updating cc65b36..3e88704
Fast-forward
 feature-0.1.txt | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 feature-0.1.txt
Deleted branch feature/0.1 (was 3e88704).

Summary of actions:
- The feature branch 'feature/0.1' was merged into 'develop'
- Feature branch 'feature/0.1' has been removed
- You are now on branch 'develop'
```

## 总结

- git-flow 插件只是 git 命令的集合，并不创建新的命令
- 主要分支生命周期无限长，支持分支最终都会被删除
- git-flow 命令只适合在创建支持分支，push，最后发布使用，开发过程中的提交需要用原生的git 命令
- master 和 develop 分支只接受合并，不接受提交，故可锁定
- 私有化分支和重大分支版本(如v1升级到v2)，切勿使用 git-flow 命令
- 务必规范测试流程，在release分支提测，修改bug
- 规范构建脚本，配合自动打包流程和工具使用

