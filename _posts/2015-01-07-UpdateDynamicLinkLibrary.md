---
layout: post
title: Update dynamic link library in android app.
categories: [Development, Blog]
---

{{ page.title }}
================
android 应用中偶尔会碰到这样的需求，仅仅更新SO修复一些bug，或者做一些必要的更新，在网络上可以找到很多动态更新SO库的文章。

试验过后发现各种问题都很多，于是重新写了一个测试可用的例子，代码如下：

	String soName = "libxxx.so";
        String soName2 = "libxxx2.so";
        String filepath = SDCARD_LOCAL_PATH + soName;
        File soFile = new File(filepath);
        File dir = getDir("libs", Context.MODE_PRIVATE);
        File soFile2 = new File(dir, soName2);

        Log.d(TAG, soFile2.getAbsolutePath());
        Log.d(TAG, filepath);
        try {
            FileOutputStream out = new FileOutputStream(soFile2);
            FileInputStream in = new FileInputStream(soFile);

            byte[] bytes = new byte[1024 * 1024];
            int length = 0;
            while ((length = in.read(bytes, 0, 1024 * 1024)) != -1) {
               out.write(bytes, 0, length);
            }
            in.close();
            out.close();

        System.load(soFile2.getAbsolutePath());

        } catch (Exception e) {
            e.printStackTrace();
        }
SDCARD_LOCAL_PATH 为 本地SD卡下的目录，soName 为SD卡下的更新的so。
soName2 为拷贝到app内的目标SO名。

对于动态更新SO的方法有如下注意点：

1.  android 安装APP后SO的目录是/data/app-lib/package-name-N/
这个目录我们无法直接获得，N可能在安装多次之后1 2 ...N，是属于系统级的安装目录，所以我们不能也不应该直接通过代码更新这个目录下SO。
最好是通过getDir("libs", Context.MODE_PRIVATE)这个方法获得app私有目录：/data/data/package-name/app\_libs。更新新的SO到这个目录下，然后加载是最合乎逻辑的方法。

2.  APP运行时，已经加载了老的SO，JNI和Android系统没有unload SO的API。如果load同名的so，load是不会成功的。就测试情况来看，如果两个不同名的SO，如果接口一样，代码不一样，系统会按照最后一次load成功的so来调用。所以需要动态更新一个so，那么必须要用一个新的名字，才能load成功。

3.  更新动态链接库的弊端是如果CPU架构不同，SO也会不能兼容，所以服务端可能要根据请求设备的CPU来判断发放不同的SO。
