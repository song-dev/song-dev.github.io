---
layout: post
title: "crash: 'stack corruption detected' 问题分析"
subtitle: ' crash 问题总结 '
author: "Song"
header-style: text
tags:
  - crash
  - native
  - jni
---

## OLLVM 简介（什么是 OLLVM）

```
2019-12-10 16:26:16.466 8344-8344/? A/DEBUG: *** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
2019-12-10 16:26:16.466 8344-8344/? A/DEBUG: Build fingerprint: 'google/crosshatch/crosshatch:10/QP1A.191005.007/5878874:user/release-keys'
2019-12-10 16:26:16.466 8344-8344/? A/DEBUG: Revision: 'MP1.0'
2019-12-10 16:26:16.466 8344-8344/? A/DEBUG: ABI: 'arm'
2019-12-10 16:26:16.466 8344-8344/? A/DEBUG: Timestamp: 2019-12-10 16:26:16+0800
2019-12-10 16:26:16.466 8344-8344/? A/DEBUG: pid: 7939, tid: 8167, name: pool-14-thread-  >>> com.zjwh.android_wh_physicalfitness <<<
2019-12-10 16:26:16.466 8344-8344/? A/DEBUG: uid: 10206
2019-12-10 16:26:16.466 8344-8344/? A/DEBUG: signal 6 (SIGABRT), code 0 (SI_USER), fault addr --------
2019-12-10 16:26:16.466 8344-8344/? A/DEBUG: Abort message: 'JNI DETECTED ERROR IN APPLICATION: input is not valid Modified UTF-8: illegal start byte 0xbc
        string: 'OJ��d�'
        input: '0x18 0x4f 0x4a <0xbc> 0xfd 0x1a 0x64 0xe7 0x01'
        in call to NewStringUTF
        from byte[] com.geetest.mobinfo.GtMobHelper.getPostParamForDeepknow(com.geetest.mobinfo.ConfigInterface, org.json.JSONObject, org.json.JSONObject, android.content.Context)'
2019-12-10 16:26:16.466 8344-8344/? A/DEBUG:     r0  00000000  r1  00001fe7  r2  00000006  r3  bc4a49c8
2019-12-10 16:26:16.466 8344-8344/? A/DEBUG:     r4  bc4a49dc  r5  bc4a49c0  r6  00001f03  r7  0000016b
2019-12-10 16:26:16.466 8344-8344/? A/DEBUG:     r8  bc4a49d8  r9  bc4a49c8  r10 bc4a49f8  r11 bc4a49e8
2019-12-10 16:26:16.466 8344-8344/? A/DEBUG:     ip  00001fe7  sp  bc4a4998  lr  e997e6c3  pc  e997e6d6
2019-12-10 16:26:16.620 8344-8344/? A/DEBUG: backtrace:
2019-12-10 16:26:16.621 8344-8344/? A/DEBUG:       #00 pc 0005f6d6  /apex/com.android.runtime/lib/bionic/libc.so (abort+166) (BuildId: cc27f9179247ae9d87d4072edb0bf184)
2019-12-10 16:26:16.621 8344-8344/? A/DEBUG:       #01 pc 0037719d  /apex/com.android.runtime/lib/libart.so (art::Runtime::Abort(char const*)+1680) (BuildId: 256ef641a0210baa82c07de7aa3a69f8)
2019-12-10 16:26:16.621 8344-8344/? A/DEBUG:       #02 pc 0000856f  /system/lib/libbase.so (android::base::LogMessage::~LogMessage()+406) (BuildId: 890987f10cf1a41629431f922968bc5d)
2019-12-10 16:26:16.621 8344-8344/? A/DEBUG:       #03 pc 00288483  /apex/com.android.runtime/lib/libart.so (art::JavaVMExt::JniAbort(char const*, char const*)+1198) (BuildId: 256ef641a0210baa82c07de7aa3a69f8)
2019-12-10 16:26:16.621 8344-8344/? A/DEBUG:       #04 pc 002885bf  /apex/com.android.runtime/lib/libart.so (art::JavaVMExt::JniAbortV(char const*, char const*, std::__va_list)+54) (BuildId: 256ef641a0210baa82c07de7aa3a69f8)
2019-12-10 16:26:16.621 8344-8344/? A/DEBUG:       #05 pc 0027de79  /apex/com.android.runtime/lib/libart.so (art::(anonymous namespace)::ScopedCheck::AbortF(char const*, ...)+40) (BuildId: 256ef641a0210baa82c07de7aa3a69f8)
2019-12-10 16:26:16.621 8344-8344/? A/DEBUG:       #06 pc 0027dda1  /apex/com.android.runtime/lib/libart.so (art::(anonymous namespace)::ScopedCheck::CheckNonHeapValue(char, art::(anonymous namespace)::JniValueType)+936) (BuildId: 256ef641a0210baa82c07de7aa3a69f8)
2019-12-10 16:26:16.621 8344-8344/? A/DEBUG:       #07 pc 0027c345  /apex/com.android.runtime/lib/libart.so (art::(anonymous namespace)::ScopedCheck::Check(art::ScopedObjectAccess&, bool, char const*, art::(anonymous namespace)::JniValueType*)+628) (BuildId: 256ef641a0210baa82c07de7aa3a69f8)
2019-12-10 16:26:16.621 8344-8344/? A/DEBUG:       #08 pc 0027404d  /apex/com.android.runtime/lib/libart.so (art::(anonymous namespace)::CheckJNI::NewStringUTF(_JNIEnv*, char const*)+500) (BuildId: 256ef641a0210baa82c07de7aa3a69f8)
2019-12-10 16:26:16.621 8344-8344/? A/DEBUG:       #09 pc 00082da1  /data/app/com.zjwh.android_wh_physicalfitness-61-i4euIJb8aAmShgDJ_NQ==/lib/arm/libdeepknow-lib.so (BuildId: cc4cf55d8991920e136db257c1d013a9eda36880)
2019-12-10 16:26:16.621 8344-8344/? A/DEBUG:       #10 pc 0005a477  /data/app/com.zjwh.android_wh_physicalfitness-61-i4euIJb8aAmShgDJ_NQ==/lib/arm/libdeepknow-lib.so (BuildId: cc4cf55d8991920e136db257c1d013a9eda36880)
2019-12-10 16:26:16.621 8344-8344/? A/DEBUG:       #11 pc 0007e66b  /data/app/com.zjwh.android_wh_physicalfitness-61-i4euIJb8aAmShgDJ_NQ==/lib/arm/libdeepknow-lib.so (BuildId: cc4cf55d8991920e136db257c1d013a9eda36880)
2019-12-10 16:26:16.621 8344-8344/? A/DEBUG:       #12 pc 00035cc7  /data/app/com.zjwh.android_wh_physicalfitness-61-i4euIJb8aAmShgDJ_NQ==/lib/arm/libdeepknow-lib.so (BuildId: cc4cf55d8991920e136db257c1d013a9eda36880)
2019-12-10 16:26:16.621 8344-8344/? A/DEBUG:       #13 pc 00032909  /data/app/com.zjwh.android_wh_physicalfitness-61-i4euIJb8aAmShgDJ_NQ==/lib/arm/libdeepknow-lib.so (BuildId: cc4cf55d8991920e136db257c1d013a9eda36880)
2019-12-10 16:26:16.621 8344-8344/? A/DEBUG:       #14 pc 000dc519  /apex/com.android.runtime/lib/libart.so (art_quick_generic_jni_trampoline+40) (BuildId: 256ef641a0210baa82c07de7aa3a69f8)
2019-12-10 16:26:16.621 8344-8344/? A/DEBUG:       #15 pc 000d7bc5  /apex/com.android.runtime/lib/libart.so (art_quick_invoke_stub_internal+68) (BuildId: 256ef641a0210baa82c07de7aa3a69f8)
2019-12-10 16:26:16.621 8344-8344/? A/DEBUG:       #16 pc 0042f8af  /apex/com.android.runtime/lib/libart.so (art_quick_invoke_static_stub+246) (BuildId: 256ef641a0210baa82c07de7aa3a69f8)
```

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
