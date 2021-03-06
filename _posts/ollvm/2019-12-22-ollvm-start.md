---
layout: post
title: "OLLVM 快速学习"
subtitle: ' OLLVM 快速学习 '
author: "Song"
header-style: text
tags:
  - gradle
  - build
  - compress
---

## OLLVM 简介（什么是 OLLVM）

OLLVM（Obfuscator-LLVM）是瑞士西北应用科技大学安全实验室于2010年6月份发起的一个项目，该项目旨在提供一套开源的针对LLVM的代码混淆工具，以增加对逆向工程的难度。目前，OLLVM已经支持LLVM-4.0版本。

LLVM 支持 C、C++、OC等语言，x86 、arm 架构。故可以支持 Android 和 iOS。

## OLLVM 有哪些能力

### 控制流扁平化

控制流打平的目的是将程序的控制流图完全地扁平化。

> 将原先的控制流 if else 变为 switch

### 指令替换

所谓指令替换仅仅是对标准二进制运算（比如加、减、位运算）使用更复杂的指令序列进行功能等价替换，当存在多种等价指令序列时，随机选择一种。

这种混淆并不直截了当而且并没有增加更多的安全性，因为通过重新优化可以很容易地把替换的等价指令序列变回去。然而，提供一个伪随机数，就可以使指令替换给二进制文件带来多样性。

目前，只有在整数上的操作可用，因为在浮点数上的运算替换会带来四舍五入的错误以及不必要的数值不准确。

> 如 a = b + c，替换为 a = b + random() - c - random() + c + c

### 虚假控制流

这种方式通过在当前基本块之前添加一个基本块，来修改函数调用流程图。新添加的基本块包含一个不透明的谓语，然后再跳转到原来的基本块。

原始的基本块会被克隆，并充满了随机的垃圾指令。

> 增加虚假函数控制，也就是增加虚假指令


### 字符串混淆

## OLLVM 原理是什么

https://bbs.pediy.com/thread-225756.htm
