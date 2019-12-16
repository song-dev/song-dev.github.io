---
layout: post
title: "git 分支命名规范 & commit 描述规范"
subtitle: '标准化 git flow 流程分支命名和提交描述'
author: "Song"
header-style: text
tags:
  - git
  - git-flow
  - commit
  - branch
---

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

