---
layout:     post
title:      "把 Charls CA 证书安装到 Android 系统级别信任中"
date:       2019-10-14 01:56:33+800
author:     "LnsooXD"
header-img: "img/post-bg-qa-type.jpg"
tags:
    - Android
    - Charles
    - 测试
    - QA
---

## 前言

使用 Charles 抓包测试 https 请求的时候，需要安装 Charles 的 CA 证书。一般情况下，在`设置->安全->加密与凭据` 中安装即可。但是有的时候， 这样安装的证书， 系统就是不认(一般是国产厂商的系统), 此时我们就需要把证书导入到系统信任列表中。

## Requements

- 一台root过的 Android 手机
- openssl
- The CA cert: charles.pem

## Steps

得到证书的hash

```shell
openssl x509 -inform PEM -subject_hash_old -in bec0e24d.0 | head -1
```

重命名证书文件({hash}为第一步得到的hash值)

```shell
openssl x509 -inform PEM -text -in {hash}.0 -out /dev/null >> {hash}.0
```

把证书push到手机上

```shell
adb push {hash}.0 /sdcard/
```

adb 登陆手机, 并重新挂载 `/system` 为可读写

```shell
adb shell
su
mount -o rw,remount /system
```

把证书导入系统 CA 目录

```shell
cp /sdcard/{hash}.0  /system/etc/security/cacerts/
```

确保证书文件权限

```shell
chmod 644  /system/etc/security/cacerts/{hash}.0
```

重启， 打开 `设置->安全->加密与凭据[系统]` 查看
