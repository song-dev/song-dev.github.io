---
layout: post
title: "Android root 检测"
subtitle: ' Android 安全检测 '
author: "Song"
header-style: text
tags:
  - root
  - check
  - Android
---

## Android root 检测

检测是否在 root 环境 [https://github.com/song-dev/security-check-android](https://github.com/song-dev/security-check-android)

## 方案

### 执行 `su` 命令

`root` 设备都支持 `su` 命令，直接执行 `su -v` 获取 `root` 应用版本号。若返回为空则为非 `root` 设备，否则为 `root` 设备

```c
/**
 * root 检测
 * @return 0: false 1: true 2: not support
 */
int rootCheck(char *dest) {
    FILE *f = NULL;
    f = popen("su -v", "r");
    if (f != NULL) {
        if (fgets(dest, BUF_SIZE_256, f) == NULL) {
            LOGD("read popen error: %s", strerror(errno));
            pclose(f);
            return 0;
        }
        LOGD("su -v: %s", dest);
        if (strlen(dest) == 0) {
            pclose(f);
            LOGD("this not is root.");
            return 0;
        } else {
            pclose(f);
            LOGD("this is rooted.");
            return 1;
        }
    } else {
        LOGD("file pointer is null.");
        return 2;
    }
}
```
> 100% 可靠

### 特殊文件检测

检测特殊路径下特有 `su` 文件，若存在则为 `root` 设备

```java
public static boolean fileCheck() {
    List<String> filesFound = new ArrayList<>();
    for (String path : SU_PATHS) {
        if (new File(path).exists()) {
            filesFound.add(path);
        }
    }
    LogUtils.d(filesFound.toString());
    return filesFound.size() > 0;
}
```

> 不可靠，只能检测已有列表设备

### root 应用检测

`root` 设备都有管理应用，检测常见应用即可

```java
public static boolean rootPackagesCheck(Context context) {
    ArrayList<String> packages = new ArrayList<>();
    packages.addAll(Arrays.asList(KNOWN_ROOT_APPS_PACKAGES));
    packages.addAll(Arrays.asList(KNOWN_DANGEROUS_APPS_PACKAGES));
    packages.addAll(Arrays.asList(KNOWN_ROOT_CLOAKING_PACKAGES));
    PackageManager pm = context.getPackageManager();
    List<String> packagesFound = new ArrayList<>();
    for (String packageName : packages) {
        try {
            // Root app detected
            pm.getPackageInfo(packageName, 0);
            packagesFound.add(packageName);
        } catch (PackageManager.NameNotFoundException e) {
        }
    }
    LogUtils.d(packagesFound.toString());
    return packagesFound.size() > 0;
}
```

> 不可靠，只能检测已有列表设备

### 特殊属性检测

`root` 设备 `[ro.debuggable]` 和 `[ro.secure]` 分别表现为 `[1]`、`[0]`。检测当前属性即可。

```java
public static boolean dangerousPropertiesCheck() {
    DANGEROUS_PROPERTIES.put("[ro.debuggable]", "[1]");
    DANGEROUS_PROPERTIES.put("[ro.secure]", "[0]");
    String[] lines = propertiesReader();
    List<String> propertiesFound = new ArrayList<>();
    assert lines != null;
    for (String line : lines) {
        for (String key : DANGEROUS_PROPERTIES.keySet()) {
            if (line.contains(key) && line.contains(DANGEROUS_PROPERTIES.get(key))) {
                propertiesFound.add(line);
            }
        }
    }
    LogUtils.d(propertiesFound.toString());
    return propertiesFound.size() > 0;
}
```

> `[ro.debuggable]` 属性为 `[1]` 不一定表现为 `root` 可能是在 `debug` 状态

## 参考

- https://www.jianshu.com/p/c37b1bdb4757
