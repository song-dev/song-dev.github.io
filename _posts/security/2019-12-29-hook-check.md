---
layout: post
title: "Android Hook 检测"
subtitle: ' Android 安全检测 '
author: "Song"
header-style: text
tags:
  - hook
  - check
  - Android
---

## Android Hook 检测

检测是否被 `hook`  [https://github.com/song-dev/security-check-android](https://github.com/song-dev/security-check-android)

## 方案

### 主流 hook 框架检测

检测 `maps` 中包含的 `frida`、`substrate`、`Xposed` 框架部分

```c
/**
 * 检测主流 hook 框架: frida、Xposed、substrate
 * @return
 */
int frameCheck() {
    char path[BUF_SIZE_32];
    sprintf(path, "/proc/%d/maps", getpid());
    // 读取数据
    FILE *f = NULL;
    char buf[BUF_SIZE_512];
    f = fopen(path, "r");
    if (f != NULL) {
        while (fgets(buf, BUF_SIZE_512, f)) {
            // fgets 当读取 (n-1) 个字符时，或者读取到换行符时，或者到达文件末尾时，它会停止，具体视情况而定。
            LOGI("maps: %s", buf);
            if (strstr(buf, "frida")
                || strstr(buf, "com.saurik.substrate")
                || strstr(buf, "XposedBridge.jar")) {
                fclose(f);
                return 1;
            }
        }
    }
    fclose(f);
    return 0;
}
```

> 可靠

### 包名检测

检测三大框架对应包名或者文件

```c
/**
 * 检查核心文件，若存在则为危险环境。不一定正在被 hook，但是有风险
 * @return
 */
int pkgCheck() {
    if (exists("/data/data/de.robv.android.xposed.installer")
        || exists("/data/data/com.saurik.substrate")
        || exists("/data/local/tmp/frida-server")) {
        return 1;
    }
    return 0;
}
```

### `Xposed` 类检测

尝试加载 `Xposed` 公共类，若可以加载则为 `Xposed` 环境中

```java
/**
 * 尝试加载 Xposed 类
 *
 * @return
 */
public static boolean classCheck() {
    try {
        Class.forName("de.robv.android.xposed.XC_MethodHook");
        return true;
    } catch (Exception e) {
    }
    try {
        Class.forName("de.robv.android.xposed.XposedHelpers");
        return true;
    } catch (Exception e) {
    }
    return false;
}
```

### 异常栈检测

检测异常栈是否有可疑框架

```java
/**
 * 检测调用栈中的可疑方法
 */
public static boolean exceptionCheck() {
    try {
        throw new Exception("Deteck hook");
    } catch (Exception e) {
        int zygoteInitCallCount = 0;
        for (StackTraceElement item : e.getStackTrace()) {
            // 检测"com.android.internal.os.ZygoteInit"是否出现两次，如果出现两次，则表明Substrate框架已经安装
            if ("com.android.internal.os.ZygoteInit".equals(item.getClassName())) {
                zygoteInitCallCount++;
                if (zygoteInitCallCount == 2) {
                    LogUtils.i("Substrate is active on the device.");
                    return true;
                }
            }
            if ("com.saurik.substrate.MS$2".equals(item.getClassName()) && "invoke".equals(item.getMethodName())) {
                LogUtils.i("A method on the stack trace has been hooked using Substrate.");
                return true;
            }
            if ("de.robv.android.xposed.XposedBridge".equals(item.getClassName())
                    && "main".equals(item.getMethodName())) {
                LogUtils.i("Xposed is active on the device.");
                return true;
            }
            if ("de.robv.android.xposed.XposedBridge".equals(item.getClassName())
                    && "handleHookedMethod".equals(item.getMethodName())) {
                LogUtils.i("A method on the stack trace has been hooked using Xposed.");
                return true;
            }
        }
    }

    return false;
}
```

### 核心 `so` 检测

检测 `substrate` 和 `xhook` 的 `so` 文件

```c
/**
 * 检测已加载到内存的 Substrate 核心文件
 * @return
 */
int substrateSoCheck() {
    // 直接加载危险的so
    void *imagehandle = dlopen("libsubstrate-dvm.so", RTLD_GLOBAL | RTLD_NOW);
    if (imagehandle != NULL) {
        void *sym = dlsym(imagehandle, "MSJavaHookMethod");
        if (sym != NULL) {
            dlclose(imagehandle);
            LOGE("find Cydia Substrate");
            //发现Cydia Substrate相关模块
            return 1;
        }
    }
    LOGE("not find Cydia Substrate");
    return 0;
}

/**
 * 检测已加载到内存的 xhook 核心文件
 * @return
 */
int xhookCheck() {
    // 直接加载危险的so
    void *imagehandle = dlopen("libxhook.so", RTLD_GLOBAL | RTLD_NOW);
    if (imagehandle != NULL) {
        void *sym = dlsym(imagehandle, "xhook_register");
        if (sym != NULL) {
            dlclose(imagehandle);
            LOGE("find xhook");
            return 1;
        }
    }
    LOGE("not find xhook");
    return 0;
}
```

## 参考

- https://www.jianshu.com/p/c37b1bdb4757
