---
layout: post
title: "crash: 'input is not valid Modified UTF-8' 问题分析"
subtitle: ' crash 问题总结 '
author: "Song"
header-style: text
tags:
  - crash
  - native
  - jni
---

## 日志详细信息

```
A/DEBUG: *** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
A/DEBUG: Build fingerprint: 'google/crosshatch/crosshatch:10/QP1A.191005.007/5878874:user/release-keys'
A/DEBUG: Revision: 'MP1.0'
A/DEBUG: ABI: 'arm'
A/DEBUG: Timestamp: 2019-12-10 16:26:16+0800
A/DEBUG: pid: 7939, tid: 8167, name: pool-14-thread-  >>> com.zjwh.android_wh_physicalfitness <<<
A/DEBUG: uid: 10206
A/DEBUG: signal 6 (SIGABRT), code 0 (SI_USER), fault addr --------
A/DEBUG: Abort message: 'JNI DETECTED ERROR IN APPLICATION: input is not valid Modified UTF-8: illegal start byte 0xbc
        string: 'OJ��d�'
        input: '0x18 0x4f 0x4a <0xbc> 0xfd 0x1a 0x64 0xe7 0x01'
        in call to NewStringUTF
        from byte[] com.geetest.mobinfo.GtMobHelper.getPostParamForDeepknow(com.geetest.mobinfo.ConfigInterface, org.json.JSONObject, org.json.JSONObject, android.content.Context)'
A/DEBUG:     r0  00000000  r1  00001fe7  r2  00000006  r3  bc4a49c8
A/DEBUG:     r4  bc4a49dc  r5  bc4a49c0  r6  00001f03  r7  0000016b
A/DEBUG:     r8  bc4a49d8  r9  bc4a49c8  r10 bc4a49f8  r11 bc4a49e8
A/DEBUG:     ip  00001fe7  sp  bc4a4998  lr  e997e6c3  pc  e997e6d6
A/DEBUG: backtrace:
A/DEBUG:       #00 pc 0005f6d6  /apex/com.android.runtime/lib/bionic/libc.so (abort+166) (BuildId: cc27f9179247ae9d87d4072edb0bf184)
A/DEBUG:       #01 pc 0037719d  /apex/com.android.runtime/lib/libart.so (art::Runtime::Abort(char const*)+1680) (BuildId: 256ef641a0210baa82c07de7aa3a69f8)
A/DEBUG:       #02 pc 0000856f  /system/lib/libbase.so (android::base::LogMessage::~LogMessage()+406) (BuildId: 890987f10cf1a41629431f922968bc5d)
A/DEBUG:       #03 pc 00288483  /apex/com.android.runtime/lib/libart.so (art::JavaVMExt::JniAbort(char const*, char const*)+1198) (BuildId: 256ef641a0210baa82c07de7aa3a69f8)
A/DEBUG:       #04 pc 002885bf  /apex/com.android.runtime/lib/libart.so (art::JavaVMExt::JniAbortV(char const*, char const*, std::__va_list)+54) (BuildId: 256ef641a0210baa82c07de7aa3a69f8)
A/DEBUG:       #05 pc 0027de79  /apex/com.android.runtime/lib/libart.so (art::(anonymous namespace)::ScopedCheck::AbortF(char const*, ...)+40) (BuildId: 256ef641a0210baa82c07de7aa3a69f8)
A/DEBUG:       #06 pc 0027dda1  /apex/com.android.runtime/lib/libart.so (art::(anonymous namespace)::ScopedCheck::CheckNonHeapValue(char, art::(anonymous namespace)::JniValueType)+936) (BuildId: 256ef641a0210baa82c07de7aa3a69f8)
A/DEBUG:       #07 pc 0027c345  /apex/com.android.runtime/lib/libart.so (art::(anonymous namespace)::ScopedCheck::Check(art::ScopedObjectAccess&, bool, char const*, art::(anonymous namespace)::JniValueType*)+628) (BuildId: 256ef641a0210baa82c07de7aa3a69f8)
A/DEBUG:       #08 pc 0027404d  /apex/com.android.runtime/lib/libart.so (art::(anonymous namespace)::CheckJNI::NewStringUTF(_JNIEnv*, char const*)+500) (BuildId: 256ef641a0210baa82c07de7aa3a69f8)
A/DEBUG:       #09 pc 00082da1  /data/app/com.zjwh.android_wh_physicalfitness-61-i4euIJb8aAmShgDJ_NQ==/lib/arm/libdeepknow-lib.so (BuildId: cc4cf55d8991920e136db257c1d013a9eda36880)
A/DEBUG:       #10 pc 0005a477  /data/app/com.zjwh.android_wh_physicalfitness-61-i4euIJb8aAmShgDJ_NQ==/lib/arm/libdeepknow-lib.so (BuildId: cc4cf55d8991920e136db257c1d013a9eda36880)
A/DEBUG:       #11 pc 0007e66b  /data/app/com.zjwh.android_wh_physicalfitness-61-i4euIJb8aAmShgDJ_NQ==/lib/arm/libdeepknow-lib.so (BuildId: cc4cf55d8991920e136db257c1d013a9eda36880)
A/DEBUG:       #12 pc 00035cc7  /data/app/com.zjwh.android_wh_physicalfitness-61-i4euIJb8aAmShgDJ_NQ==/lib/arm/libdeepknow-lib.so (BuildId: cc4cf55d8991920e136db257c1d013a9eda36880)
A/DEBUG:       #13 pc 00032909  /data/app/com.zjwh.android_wh_physicalfitness-61-i4euIJb8aAmShgDJ_NQ==/lib/arm/libdeepknow-lib.so (BuildId: cc4cf55d8991920e136db257c1d013a9eda36880)
A/DEBUG:       #14 pc 000dc519  /apex/com.android.runtime/lib/libart.so (art_quick_generic_jni_trampoline+40) (BuildId: 256ef641a0210baa82c07de7aa3a69f8)
A/DEBUG:       #15 pc 000d7bc5  /apex/com.android.runtime/lib/libart.so (art_quick_invoke_stub_internal+68) (BuildId: 256ef641a0210baa82c07de7aa3a69f8)
A/DEBUG:       #16 pc 0042f8af  /apex/com.android.runtime/lib/libart.so (art_quick_invoke_static_stub+246) (BuildId: 256ef641a0210baa82c07de7aa3a69f8)
```

## 原因

`jni` 函数 `NewStringUTF` 参数并非 `ASCII` 字符，导致乱码，报 `CheckJNI` 错误

## 方案

- 排查调用 `NewStringUTF` 位置，可能因为某些原因数据处理错误，导致未被初始化数据（内存随机数据）入参报错。
- 初始化所有可能位置，且检查语句处理错误原因

## 参考

- https://stackoverflow.com/questions/12127817/android-ics-4-0-ndk-newstringutf-is-crashing-down-the-app
- https://www.jianshu.com/p/333648d8a998