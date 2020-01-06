---
layout: post
title: "Android 多开检测"
subtitle: ' Android 安全检测 '
author: "Song"
header-style: text
tags:
  - 多开
  - Android
  - check
---

## Android 多开原理

通过在宿主容器上面新建一个进程供插件APK寄宿，然后通过hook一些系统接口欺骗应用——让虚拟化后应用以为自己是正常运行的独立APP，欺骗系统——让系统认为此虚拟化应用是一个已正常安装在系统的应用。

## Android 多开检测方案

### 权限访问检测

`shell` 命令执行在单独进程，不受宿主控制。故可以通过 `shell` 命令访问内部存储目录，若可以访问则正常，否则为多开环境。

```c
/**
 * 0. 多开检测 false
 * 1. 多开检测 true
 * 2. 检测失败（$unknown）
 * 检测多开, 若可访问规定目录则为正常，否则为多开环境
 * @return
 */
int moreOpenCheck() {
    // 判断是否支持ls命令
    if (exists("/system/bin/ls")) {
        char packageName[BUF_SIZE_64] = UNKNOWN;
        if (getPackageName(packageName) != 0) {
            return 2;
        }
        char path[BUF_SIZE_128];
        sprintf(path, "ls /data/data/%s", packageName);
        FILE *f = NULL;
        f = popen(path, "r");
        if (f == NULL) {
            LOGD("file pointer is null.");
            return 2;
        } else {
            // 读取 shell 命令内容
            char buff[BUF_SIZE_32];
            if (fgets(buff, BUF_SIZE_32, f) == NULL) {
                LOGD("ls data error: %s", strerror(errno));
                pclose(f);
                return 1;
            }
            LOGD("ls data: %s", buff);
            if (strlen(buff) == 0) {
                pclose(f);
                return 1;
            } else {
                pclose(f);
                return 0;
            }
        }
    } else {
        return 2;
    }
}
```

> 可靠性 100%

### 端口检测

1. app运行后，先做发送端，在合适的时候去连接本地端口并发送一段密文消息，如果有端口连接且密文匹配，则认为之前已经有app在运行了（广义多开），接收端进行处理；
2. app再成为接收端，接收可能到来连接；
3. 后续若有app启动（无论本体or克隆体），则重复 1 & 2 步骤，达到『同一时间只有一个app在运行』的目的，解决广义多开的问题。

[https://github.com/lamster2018/EasyProtector/blob/master/library/src/main/java/com/lahm/library/VirtualApkCheckUtil.java](https://github.com/lamster2018/EasyProtector/blob/master/library/src/main/java/com/lahm/library/VirtualApkCheckUtil.java)

> 比较可靠，但是有缺陷，无法检测删除原 APP 情况

### 内部存储目录路径检测

检测当前内部存储目录路径是否是标准路径

```java
/**
 * 判断当前私有路径是否是标准路径
 *
 * @param context
 * @return
 */
public static boolean pathCheck(Context context) {
		// 获取内部存储目录路径
    String filesDir = context.getFilesDir().getAbsolutePath();
    String packageName = context.getPackageName();
    String normalPath_one = "/data/data/" + packageName + "/files";
    String normalPath_two = "/data/user/0/" + packageName + "/files";
    // 当前存储目录路径和正常存储目录路径比对
    if (!normalPath_one.equals(filesDir) && !normalPath_two.equals(filesDir)) {
        return true;
    }
    return false;
}
```

> 比较可靠，目前只有 360 分身大师绕过

### 包名检测

```java
/**
 * 若 applist 存在两个当前包名则为多开
 *
 * @param context
 * @return
 * @deprecated 大部分多开软件已经绕过
 */
public static boolean packageCheck(Context context) {
    try {
        if (context == null) {
            return false;
        }
        int count = 0;
        String packageName = context.getPackageName();
        PackageManager pm = context.getPackageManager();
        List<PackageInfo> pkgs = pm.getInstalledPackages(0);
        for (PackageInfo info : pkgs) {
            if (packageName.equals(info.packageName)) {
                count++;
            }
        }
        return count > 1;
    } catch (Exception ignore) {
    }
    return false;
}
```

> 不可靠，大部分多开软件都已绕过

### 进程检测

```java
/**
 * 进程检测，若出现同一个 uid 下出现的进程名对应 /data/data/pkg 私有目录，超出 1 个则为多开
 * 需要排除当前进程名存在多个情况
 *
 * @return
 * @deprecated 6.0 以上系统只能获取仅有当前进程
 */
public static boolean processCheck() {
    String filter = CommandUtils.getUidStrFormat();
    String result = CommandUtils.execute("ps");
    if (result == null || result.isEmpty()) {
        return false;
    }
    String[] lines = result.split("\n");
    if (lines == null || lines.length <= 0) {
        return false;
    }
    int exitDirCount = 0;
    for (int i = 0; i < lines.length; i++) {
        if (lines[i].contains(filter)) {
            int pkgStartIndex = lines[i].lastIndexOf(" ");
            String processName = lines[i].substring(pkgStartIndex <= 0
                    ? 0 : pkgStartIndex + 1, lines[i].length());
            File dataFile = new File(String.format("/data/data/%s",
                    processName, Locale.CHINA));
            if (dataFile.exists()) {
                exitDirCount++;
            }
        }
    }
    return exitDirCount > 1;
}
```

> 不可靠，6.0 以上系统只能获取仅有当前进程。

### maps 检测

```java
/**
 * maps检测, 若 maps 文件包含多开包名则为多开环境
 *
 * @return
 * @deprecated 无法普适所有多开软件, 且部分软件 maps 不依赖当前路径下 so
 */
public static boolean mapsCheck() {
    BufferedReader bufr = null;
    try {
        bufr = new BufferedReader(new FileReader("/proc/self/maps"));
        String line;
        while ((line = bufr.readLine()) != null) {
        		// 需要维护多开 APP 列表
            for (String pkg : virtualPkgs) {
                if (line.contains(pkg)) {
                    return true;
                }
            }
        }
    } catch (Exception ignore) {
        //忽略异常
    } finally {
        if (bufr != null) {
            try {
                bufr.close();
            } catch (IOException e) {
                //忽略异常
            }
        }
    }
    return false;
}
```

> 不可靠，大部分多开软件都已绕过

## 参考

- [https://blog.darkness463.top/2018/05/04/Android-Virtual-Check/](https://blog.darkness463.top/2018/05/04/Android-Virtual-Check/)
- [https://www.jianshu.com/p/216d65d9971e](https://www.jianshu.com/p/216d65d9971e)
- [https://juejin.im/post/5b484b9a5188251aae328c9b](https://juejin.im/post/5b484b9a5188251aae328c9b)
