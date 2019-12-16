---
layout: post
title: "同一个终端配置多个 ssh key"
subtitle: ' SSH 链接多个 GitHub 账号 '
author: "Song"
header-style: text
tags:
  - git
  - ssh
---

## 问题场景

自己有两个 GitHub 账号，想在同一终端使用 `SSH key` 账户校验，两者互不干扰。

## 解决方案

管理两个 `SSH key`。

### 生成额外的 `SSH key`

```shell
.ssh ssh-keygen -t rsa -C "test@gmail.com"
```
> 生成 RSA 密钥对，"test@gmail.com" 可以自定义

公私钥文件命名

```shell
Enter file in which to save the key (/Users/chensongsong/.ssh/id_rsa): 
test
```
> 这里给文件命名为 `test`

```
(base) ➜  .ssh ls
id_rsa              id_rsa.pub          known_hosts         test                test.pub
```
> 生成文件命名为 `test` 秘钥对

命名执行示例：

```shell
(base) ➜  .ssh ssh-keygen -t rsa -C "test@gmail.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/xxx/.ssh/id_rsa): test
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in test.
Your public key has been saved in test.pub.
The key fingerprint is:
SHA256:Gh8Fdgjq/RAvAvmUjb9VTWz+T3wt9oFZ8bYnSQTN1wk test@gmail.com
The key's randomart image is:
+---[RSA 2048]----+
|      ..o...oE. o|
|   . = ..o oo +oo|
|  o = o   oo.. .o|
|   = o o o  . ..o|
|    + * S    o++o|
|     . X .   o*o*|
|      o o    . *+|
|                o|
|                 |
+----[SHA256]-----+
```

### 配置 `SSH key`

- 将 `test.pub` 配置到 GitHub 
- 将私钥 `test` 添加到 `ssh-agent` 中
	
	> `ssh-add test`

### 配置 `config` 文件

- 执行 `touch config` 命令生成 `config` 文件
- `config` 文件配置示例

  ```vim
  Host github.com # 强制配置为 github.com
    HostName github.com # 强制配置为 github.com
    User songsongbrother # 其中一个 GitHub 账号名
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa

  Host test # Host 名强制和私钥文件名称一致
    HostName github.com
    User song-dev # 另外一个 GitHub 账号名
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/test
    UseKeychain yes
    AddKeysToAgent yes
  ```

- `Host` 名切记和私钥文件名称 `test` 一致

#### `config` 配置文件字段解释

- `Host test`

用来指定该 key 的 Host 名字，此处必须使用设置的私钥文件名称。

- `Hostname github.com`

此处指定 Host 对应的具体域名，这里跟 Host 保持一致。

- `User song-dev`

说明该配置的用户

- `IdentityFile ~/.ssh/test`

这行最为关键，指定了该使用哪个 ssh key 文件，这里的 key 文件一定指的是私钥文件。之前我们生成了新的私钥文件 ~/.ssh/test。

- `IdentitiesOnly yes`

请配置为yes。

### 更新仓库名称

仓库名称需要根据配置 Host 修改，如

```shell
// 原
git@github.com:song-dev/song-dev.github.io.git
// 改
git@test:song-dev/song-dev.github.io.git
```

### 测试

```shell
(base) ➜  .ssh ssh -T git@test
Hi song-dev! You've successfully authenticated, but GitHub does not provide shell access.
```
