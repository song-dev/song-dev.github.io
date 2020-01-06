---
layout: post
title: "Android debug 检测"
subtitle: ' Android 安全检测 '
author: "Song"
header-style: text
tags:
  - debug
  - android
  - check
---

## Android debug 检测

检测当前环境是否被调试

## 方案

### TracerPid 读取检测

```c
/**
 * 检测 TracerPid 若不为 0 则为debug 状态
 * @return
 */
int tracerPidCheck() {
    FILE *f = NULL;
    char path[BUF_SIZE_64];
    sprintf(path, "/proc/%d/status", getpid());
    char line[BUF_SIZE_512];
    f = fopen(path, "r");
    while (fgets(line, BUF_SIZE_512, f)) {
        if (strstr(line, "TracerPid")) {
            LOGI("TracerPid line: %s", line);
            int statue = atoi(&line[10]);
            LOGW("TracerPid: %d", statue);
            if (statue != 0) {
                return 1;
            } else {
                return 0;
            }
        }
    }
    return 0;
}
```

> 可靠

### 系统函数调用检测

```java
/**
 * 判斷是debug版本
 *
 * @param context
 * @return
 */
public static boolean debugVersionCheck(Context context) {
    try {
        return (context.getApplicationInfo().flags & ApplicationInfo.FLAG_DEBUGGABLE) != 0;
    } catch (Exception e) {
        //忽略异常
    }
    return false;
}

/**
 * 是否正在调试
 *
 * @return
 */
public static boolean connectedCheck() {
    try {
        return android.os.Debug.isDebuggerConnected();
    } catch (Exception e) {
        //忽略异常
    }
    return false;
}
```

> 可靠，但是可能被 hook 绕过

## 参考

- https://gtoad.github.io/2017/06/25/Android-Anti-Debug/
- https://bbs.pediy.com/thread-223460.htm
