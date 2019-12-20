---
layout: post
title: "检查 IMEI 格式是否合规"
subtitle: ' IMEI 生成器 '
author: "Song"
header-style: text
tags:
  - imei
  - check
---

## 简介

国际移动设备识别码（International Mobile Equipment Identity，IMEI），即通常所说的手机“串号”，用于在移动电话网络中识别每一部独立的手机等移动通信设备，相当于移动电话的身份证。序列号共有15位数字，前6位（TAC）是型号核准号码，代表手机类型。接着2位（FAC）是最后装配号，代表产地。后6位（SNR）是串号，代表生产顺序号。最后1位（SP）一般为0，是检验码，备用

## IMEI 组成

1. 前 6 位数 (TAC) 是“型号核准号码”，一般代表机型。
2. 接着的 2 位数 (FAC) 是 “最后装配号”，一般代表产地。
3. 之后的 6 位数 (SNR) 是 “串号”，一般代表生产顺序号。
4. 最后 1 位数 (SP) 通常是 “0”，为检验码，目前暂备用。

> IMEI 最后一位是验证码，通过官方提供的验证方式就可以知道每一个 IMEI 是否为符合规则的 IMEI 号。

## 验证规则

1. 区分 IMEI 的奇数位和偶数位。
2. 奇数位相加。
3. 偶数为乘以 2，若小于 10 则直接相加，大于 10 则对十位数和个位数进行相加。
4. 奇数位相加之和与第 3 步逻辑只和相加，获取到一个数字。
5. 得到的数字与 10 进行取余，余数若为 0，则验证位数字为 0，若余数不为 0，则验证位为(10-余数)。

举例: IEMI 为 `866696022549032` 的验证逻辑如下。

![check imei](https://song-dev.github.io/img/in-post/post-c/imei-check.jpg)

获得最后的结果为 58，58/10 余 8，则验证位为 10-8=2。

## 生成代码

> 见 GitHub 仓库：https://github.com/song-dev/c-study

## 验证代码

```java
public static boolean isIMEI(String imei) {
    char[] imeiChar = imei.toCharArray();
    int resultInt = 0;
    for (int i = 0; i < imeiChar.length-1; i++) {
        int a = Integer.parseInt(String.valueOf(imeiChar[i]));
        i++;
        final int temp = Integer.parseInt(String.valueOf(imeiChar[i])) * 2;
        final int b = temp < 10 ? temp : temp - 9;
        resultInt += a + b;
    }
    resultInt %= 10;
    resultInt = resultInt == 0 ? 0 : 10 - resultInt;
    int crc= Integer.parseInt(String.valueOf(imeiChar[14]));
    return (resultInt == crc);
}
```

## 官网验证

https://www.imei.info/
