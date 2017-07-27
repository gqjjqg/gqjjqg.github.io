---
layout: post
title: libcurl for android
categories: [Development , Blog]
---

{{ page.title }}
================
libcurl 7.53.1 最低支持 android-14，编译出来之后放低版本系统编译，netrc.c里的会link不到 getpwuid_r方法
注释掉 curl\lib\curl_config.h  文件中的HAVE_GETPWUID_R 宏定义即可通过，可以顺利放android-9编译
目前只用到网络传输的接口，而netrc是服务于ftp之类的需要用户名密码的东东，所以应该可以正常运行。
未实测，仅记录。

