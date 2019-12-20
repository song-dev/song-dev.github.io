---
layout: post
title: "gradle 常用配置"
subtitle: ' 参数设置、打包、签名、依赖等配置 '
author: "Song"
header-style: text
tags:
  - gradle
  - config
---

## 简介

总结下gradle配置方法，避免经常忘记和不断查阅，且不断更新

> 参考资料
[https://juejin.im/post/582d606767f3560063320b21](https://juejin.im/post/582d606767f3560063320b21)
[https://blog.csdn.net/whitley_gong/article/details/55272353](https://blog.csdn.net/whitley_gong/article/details/55272353)

> [gradle官网](https://docs.gradle.org/current/userguide/build_environment.html)

> [android studio 配置gradle官网](https://developer.android.com/studio/build/dependencies)

## 配置
### 1. 全局设置
根目录的 build.gradle 中配置：

```java
allprojects {
  repositories {
    jcenter()
  }
  tasks.withType(JavaCompile) {
    sourceCompatibility = JavaVersion.VERSION_1_8
    targetCompatibility = JavaVersion.VERSION_1_8
    options.encoding = "UTF-8"
  }
}
```

### 2. 局部设置
如果想在某个 module 设置，可以在其 build.gradle 中配置：

```java
android {
  compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
  }
}
```

### 3. 全局变量配置
根目录下的 build.gradle 定义全局变量:
```java
ext {
    compileSdkVersion = 25
    buildToolsVersion = "25.0.2"
    minSdkVersion = 14
    targetSdkVersion = 25
}
```
引用方式如：

```java
compileSdkVersion rootProject.ext.compileSdkVersion
buildToolsVersion rootProject.ext.buildToolsVersion
```
### 4. 局部变量配置
某个module的build.gradle中配置：

```java
ext {
    versionCode = 1
    versionName = "1.0"
}
```
> 可配置在最外层

引用：
```java
android {
    defaultConfig {
        versionCode versionCode
        versionName versionName
    }
}

// 字符串中引用需要$
def SDK_NAME = "geetest_onepass_android_v$versionName"
```
### 5. 在BuildConfig中添加静态变量

编译时在BuildConfig中添加静态变量

```java
buildTypes {
        release {
            buildConfigField "boolean", "LOG_DEBUG", "false"
            // 值必须双引号包括
            buildConfigField "String", "VERSION", '"' + versionName + '"'
        }
        debug {
            buildConfigField "boolean", "LOG_DEBUG", "true"
            // 值必须双引号包括
            buildConfigField "String", "VERSION", '"' + versionName + '"'
        }
    }
}

// 或者用双引号转义
buildConfigField "String", "VERSION", "\"1.0\""

// 引用示例
LOG_DEBUG = BuildConfig.LOG_DEBUG
```

### 6. 配置签名信息

```java
android {
    signingConfigs {
        // test签名文件配置
        test {
            keyAlias 'test'
            keyPassword 'test'
            // 根目录下test.jks
            storeFile file('../test.jks')
            storePassword 'test'
        }
    }
    
    buildTypes {
        release {
            // 引用签名文件
            signingConfigs.test
        }
    }
}
```

### 7. 多渠道打包

替换AndroiManifest中的channel

```java
android {
  productFlavors {
    fir {
      manifestPlaceholders = [UMENG_CHANNEL_VALUE: "fir"]
      // 替换res/values下的string文件
      resValue "string", "AppName", "fir"

    }
    GooglePlay {
      manifestPlaceholders = [UMENG_CHANNEL_VALUE: "GooglePlay"]
      resValue "string", "AppName", "GooglePlay"
    }
    Umeng {
      manifestPlaceholders = [UMENG_CHANNEL_VALUE: "Umeng"]
      resValue "string", "AppName", "Umeng"
    }
  }
}

// AndroiManifest中的字段
<meta-data
    android:name="UMENG_CHANNEL"
    android:value="${UMENG_CHANNEL_VALUE}"/>
```

### 8. 减少编译错误和忽略 lint 检查

```java
android {
    packagingOptions {
      exclude 'META-INF/DEPENDENCIES.txt'
      exclude 'META-INF/LICENSE.txt'
      exclude 'META-INF/NOTICE.txt'
      exclude 'META-INF/NOTICE'
      exclude 'META-INF/LICENSE'
      exclude 'META-INF/DEPENDENCIES'
      exclude 'META-INF/notice.txt'
      exclude 'META-INF/license.txt'
      exclude 'META-INF/dependencies.txt'
      exclude 'META-INF/LGPL2.1'
    }

    lintOptions {
      abortOnError false
    }
}
```
### 9. 生成自定义 App 名称

```java
//生成自定义App名称
def generateAppName(variant) {
    variant.outputs.each { output ->
        def file = output.outputFile
        output.outputFile = new File(file.parent, "Gank.IO-V" +           android.defaultConfig.versionName + ".apk")
    }
}
```

### 10. 指定资源目录

适用Eclipse项目转Android Studio项目

```java
android {
    sourceSets {
        main {
            manifest.srcFile 'AndroidManifest.xml'
            java.srcDirs = ['src']
            resources.srcDirs = ['src']
            aidl.srcDirs = ['src']
            renderscript.srcDirs = ['src']
            assets.srcDirs = ['assets']
            if (!IS_USE_DATABINDING) { // 如果用了databinding
                jniLibs.srcDirs = ['libs']
                res.srcDirs = ['res', 'res-vm'] // 多加了databinding的资源目录
            } else {
                res.srcDirs = ['res']
            }
        }

        test {
            java.srcDirs = ['test']
        }

        androidTest {
            java.srcDirs = ['androidTest']
        }
    }
}
```
### 11. 本地依赖

将aar添加某module的libs文件夹下，且在某module的build.gradle配置：

```java
android {
    repositories {
        flatDir {
            dirs 'libs' //this way we can find the .aar file in libs folder
        }
    }
}

dependencies {
    compile(name:'aar name', ext:'aar') // 依赖本地libs下aar
    compile fileTree(include: ['*.jar'], dir: 'libs') // 依赖本地libs下jar
    // 或者
    compile files('libs/foo.jar', 'libs/bar.jar')
    // 依赖本地library module
    compile project(':mylibraryModule')
}
```

若还有别的module引用此module，还需在引用此module的build.gradle配置

```java
android {
    repositories {
        flatDir {
            dirs 'libs' , '../module name/libs' // 也引用被引用的module的被引用的module
        }
    }
}
```

### 12. 远程依赖

```java
dependencies {
    compile 'com.example.android:app-magic:12.3' // 不建议版本号写死，每次build都会检查最新版本，且各个开发者的版本不一样，或者更新遇到api变更
    compile group: 'com.example.android', name: 'app-magic', version: '12.3'
    compile ('com.android.support:appcompat-v7:25.3.1'){
        // 剔除某个库
        exclude group: 'com.android.support' // 仅仅写组织名称
        exclude group: 'com.android.support', module: 'support-annotations' // 写全称
    }
}
```
不建议将远程依赖版本写死，可能导致不必要的问题：
> 1. 每次build时会向网络进行检查，国内访问仓库速度很慢
> 2. 库更新后可能会更改库的内部逻辑和带来bug，这样就无法通过git的diff来规避此问题
> 3. 每个开发者可能会得到不同的最新版本，带来潜在的隐患

### 13. gradle.properties配置

```java
# 官网说明：https://docs.gradle.org/current/userguide/build_environment.html
# 配置大内存
org.gradle.jvmargs=-Xmx2048m -XX:MaxPermSize=512m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
# 守护进程
org.gradle.daemon=true
# 并行编译
org.gradle.parallel=true
# 开启缓存
android.enableBuildCache=true
# 开启孵化模式
org.gradle.configureondemand=true
```

### 14.支持groovy

在根目录的build.gradle中：

```
apply plugin: 'groovy'

allprojects {
    // ...
｝

dependencies {
    compile localGroovy()
}
```

### 15. 在gradle.properies配置部分变量属性


```
# 配置部分变量
# 配置版本号
VersionName=1.4.1
# 配置版本码
VersionCode=1

// 在build.gradle引用方法
// 方法一：直接引用
println(VersionName)

方法二：读取gradle.properies文件
// 声明文件路径
def pFile = file("../[module name]/gradle.properties")
def Properties p = new Properties()
//读取文件内容
def loadProperties = {
    pFile.withInputStream { stream ->
        InputStreamReader read = new InputStreamReader(stream, "utf-8");
        BufferedReader bf = new BufferedReader(read);
        p.load(bf)
    }
}
// 引用
println(p.VersionName)
```

### 16. 设置第三方maven仓库


```
allprojects {
    repositories {
        maven {
            // 设置本地依赖
            // url "$rootDir/module_name/libs/android"
            url 'http://repo.xxxx.net/nexus/'
            name 'maven name'
            credentials {
                username = 'username'
                password = 'password'
            }
        }
    }
}

```

### 17. 打aar包


```
//打包SDK
task buildSDK(type: Copy) {
    // 设置拷贝的文件
    from('build/outputs/aar/')
    //打进aar包后的文件目录
    into('../libs/')
    include('geemessage-release.aar')
    // 重命名
    rename('geemessage-release.aar', SDK_NAME + "" + '.aar')
}
buildSDK.dependsOn(clearJar, build)

//  命令行 打arr包
//  ./gradlew buildSDK
```


## 分析


新配置 | 已弃用配置 | 概述
----|----|----
implementation | compile | 依赖项在编译时对模块可用，并且仅在运行时对模块的消费者可用。 对于大型多项目构建，使用 implementation 而不是 api/compile 可以显著缩短构建时间，因为它可以减少构建系统需要重新编译的项目量。 大多数应用和测试模块都应使用此配置。
