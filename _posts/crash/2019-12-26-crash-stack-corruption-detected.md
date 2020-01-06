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

## 日志详细信息

```
I/AEE/AED: *** *** *** *** *** *** *** *** *** *** *** *** *** *** *** ***
I/AEE/AED: Build fingerprint: 'Xiaomi/hennessy/hennessy:5.0.2/LRX22G/V9.6.1.0.LHNCNFD:user/release-keys'
I/AEE/AED: Revision: '0'
I/AEE/AED: ABI: 'arm64'
I/AEE/AED: pid: 13997, tid: 14031, name: pool-3-thread-1  >>> com.geetest.deepknow.demo <<<
I/AEE/AED: signal 6 (SIGABRT), code -6 (SI_TKILL), fault addr --------
I/AEE/AED: Abort message: 'stack corruption detected'
I/AEE/AED:     x0   0000000000000000  x1   00000000000036cf  x2   0000000000000006  x3   0000007f6728d000
I/AEE/AED:     x4   0000007f6728d000  x5   0000000000000005  x6   0000000000000001  x7   0000000000000020
I/AEE/AED:     x8   0000000000000083  x9   647362647364631f  x10  7f7f7f7f7f7f7f7f  x11  0000000000000001
I/AEE/AED:     x12  0000000000000001  x13  d5e608b579d291ce  x14  0000007f66908ae1  x15  0000000000000000
I/AEE/AED:     x16  0000007f7ce5f7e0  x17  0000007f7ce2458c  x18  0000000000000000  x19  0000007f6728d000
I/AEE/AED:     x20  0000007f66909bb0  x21  0000007f7ce65000  x22  0000000000000000  x23  0000000000000006
I/AEE/AED:     x24  00000000c6957540  x25  00000000c6957540  x26  00000000185d2283  x27  00000000fb91ea47
I/AEE/AED:     x28  000000009ebd9f39  x29  0000007f66908df0  x30  0000007f7cde0b6c
I/AEE/AED:     sp   0000007f66908df0  pc   0000007f7ce24594  pstate 0000000060000000
I/AEE/AED: backtrace:
I/AEE/AED:     #00 pc 0000000000061594  /system/lib64/libc.so (tgkill+8)
I/AEE/AED:     #01 pc 000000000001db68  /system/lib64/libc.so (pthread_kill+160)
I/AEE/AED:     #02 pc 000000000001f09c  /system/lib64/libc.so (raise+28)
I/AEE/AED:     #03 pc 0000000000018abc  /system/lib64/libc.so (abort+60)
I/AEE/AED:     #04 pc 000000000001aeb8  /system/lib64/libc.so (__libc_fatal+128)
I/AEE/AED:     #05 pc 0000000000060538  /system/lib64/libc.so (__stack_chk_fail+16)
I/AEE/AED:     #06 pc 0000000000034274  /data/app/com.geetest.deepknow.demo-1/lib/arm64/libdeepknow-lib.so
I/AEE/AED:     #07 pc 000000000004dcd4  /data/app/com.geetest.deepknow.demo-1/lib/arm64/libdeepknow-lib.so
I/AEE/AED:     #08 pc 000000000000f234  /data/app/com.geetest.deepknow.demo-1/lib/arm64/libdeepknow-lib.so
I/AEE/AED:     #09 pc 0000000000009000  /data/app/com.geetest.deepknow.demo-1/lib/arm64/libdeepknow-lib.so
I/AEE/AED:     #10 pc 0000000000357200  /data/dalvik-cache/arm64/data@app@com.geetest.deepknow.demo-1@base.apk@classes.dex
```

```
#++++++++++Record By Bugly++++++++++#
# You can use Bugly(http:\\bugly.qq.com) to get more Crash Detail!
# PKG NAME: com.geetest.deepknow.demo
# APP VER: 1.0
# LAUNCH TIME: 2019-12-23 16:03:03
# CRASH TYPE: NATIVE_CRASH
# CRASH TIME: 2019-12-23 16:03:08
# CRASH PROCESS: com.geetest.deepknow.demo
# CRASH THREAD: pool-3-thread-1(16155)
# REPORT ID: 19c22148-e85d-4284-9e6b-b508250b1d93
# CRASH DEVICE: A31t UNROOT
# RUNTIME AVAIL RAM:196071424 ROM:3186294784 SD:3133865984
# RUNTIME TOTAL RAM:944201728 ROM:5444198400 SD:5391769600
# EXCEPTION FIRED BY UNKNOWN_USER com.geetest.deepknow.demo(1015)
# CRASH STACK:
SIGABRT
0x3f7
#00    pc 00022184    /system/lib/libc.so (tgkill+12) [armeabi-v7a::a649dd9547e4a94716671d6a9529b834]
#01    pc 000131d9    /system/lib/libc.so (pthread_kill+48) [armeabi-v7a::a649dd9547e4a94716671d6a9529b834]
#02    pc 000133ed    /system/lib/libc.so (raise+10) [armeabi-v7a::a649dd9547e4a94716671d6a9529b834]
#03    pc 00012123    /system/lib/libc.so [armeabi-v7a::a649dd9547e4a94716671d6a9529b834]
#04    pc 00021a38    /system/lib/libc.so (abort+4) [armeabi-v7a::a649dd9547e4a94716671d6a9529b834]
#05    pc 00012c09    /system/lib/libc.so [armeabi-v7a::a649dd9547e4a94716671d6a9529b834]
#06    pc 000120f3    /system/lib/libc.so (__stack_chk_fail+6) [armeabi-v7a::a649dd9547e4a94716671d6a9529b834]
#07    pc 00033867    /data/app-lib/com.geetest.deepknow.demo-1/libdeepknow-lib.so [armeabi-v7a::868aa2cc249df9f08958e716e23e2913]
#08    pc 00033c33    /data/app-lib/com.geetest.deepknow.demo-1/libdeepknow-lib.so [armeabi-v7a::868aa2cc249df9f08958e716e23e2913]
#09    pc 00034b8b    /data/app-lib/com.geete
#++++++++++++++++++++++++++++++++++++++++++#
```

- 信号量信息：  `signal 6 (SIGABRT), code -6 (SI_TKILL), fault addr`
- 错误描述:  `Abort message: 'stack corruption detected'`
- 测试函数调用栈:  `__stack_chk_fail`

## 原因

`stack check failed` 的根本原因在于栈内存被覆盖，编译器会在函数进入和退出前后插入一个 `stack check` 的东西用于检测内存越界这种错误。

## 方案

排查内存可能越界位置

- 操作超出数组范围内存，给超出数组范围内存赋值
- `jni` 函数操作未有结束符的数组

## 参考

https://www.jianshu.com/p/e879ce7e0e79
