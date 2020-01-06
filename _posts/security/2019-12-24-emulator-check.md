---
layout: post
title: "Android 模拟器检测"
subtitle: ' Android 安全检测 '
author: "Song"
header-style: text
tags:
  - emulator
  - check
  - Android
---

## Android 模拟器检测

检测环境是否运行在模拟器上 [https://github.com/song-dev/security-check-android](https://github.com/song-dev/security-check-android)

## 方案

### build 特征检测

检测核心 `Build` 配置属性是否包含可疑字段

```
Build.PRODUCT
Build.MANUFACTURER
Build.BRAND
Build.DEVICE
Build.MODEL
Build.HARDWARE
Build.FINGERPRINT
```
> 不可靠

### 特殊文件检测

- 支持架构包含 `x86`
- `root` 软件版本 `16 com.android.settings`
- 声卡版本 `I82801AAICH`
- 针对模拟器厂商特殊文件检测
    ```
    /system/lib/libdroid4x.so
    /system/bin/windroyed
    /system/bin/microvirtd
    /system/bin/nox-prop
    /system/bin/ttVM-prop
    /system/lib/libc_malloc_debug_qemu.so
    /system/lib/libbluetooth_jni.so
    ```

### cpuinfo 信息检测

`/proc/cpuinfo` 包含 `intel`、`amd` 信息

### qemu 特征检测

针对特殊文件检测是否存在 qemu 特征

```
/proc/tty/drivers
/proc/cpuinfo
ro.kernel.qemu
```

## 参考

- https://mp.weixin.qq.com/s/sl33d2pnyLMJ-fUY_DfBDw
- https://github.com/Labmem003/anti-counterfeit-android
- https://github.com/lamster2018/EasyProtector
- https://github.com/happylishang/CacheEmulatorChecker
