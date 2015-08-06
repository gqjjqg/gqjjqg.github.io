---
layout: post
title: Start create your own robot(3)
categories: [Development, Blog]
---

{{ page.title }}
================
CubieBoard1 Android 固件制作(支持RTL8187驱动+OV5640UVC摄像头)

1.编译环境搭建

    参考官网文档

2.解压固件

    解压：tar -zxvf /xxx/xxx/A10-android4.0.tar.gz

3.配置内核

    如果没有需要，可以略过。
    进入根目录： cd kernel/allwinner/common/。
    配置内核：make ARCH=arm menuconfig。
    进入了UI界面：Linux/arm 3.0.52 Kernel Configuration。
    移动到最后一行并选择：Load an Alternate Configuration File 。
    加载默认的配置并确定：arch/arm/configs/cubieboard_defconfig。
    现在正式开始配置：
    
	a.增加UVC Camera支持
			Device Drivers  --->   
					Multimedia support  --->   
							Video capture adapters  --->  
									V4L USB devices (NEW)  ---> 
											<M>   USB Video Class (UVC)  
	b.增加USB WIFI 8187芯片支持 
			kconfig中有提示：
			tristate "Realtek 8187 and 8187B USB support"
			depends on MAC80211 && USB
			select EEPROM_93CX6
			LED配置：                
			depends on RTL8187 && MAC80211_LEDS && (LEDS_CLASS = y || LEDS_CLASS = RTL8187)
			default y
			
			配置一下：
			Networking support  --->
					Wireless  --->  
							<M>   Generic IEEE 802.11 Networking Stack (mac80211) 
							[*]   Enable LED triggers  (这个就是MAC80211_LEDS)
			Device Drivers  --->
					Misc devices  ---> 
							EEPROM support  ---> 
									EEPROM 93CX6 support  
					Network device support  --->    
							Wireless LAN  --->  
											<M>   Realtek 8187 and 8187B USB support 
											<M>   Realtek 8192C USB WiFi for SW 
											<M>   Realtek 8188E USB WiFi
	c.增加串口驱动支持（默认配置即可）
			Device Drivers  --->  
					Character devices  --->
							Serial drivers  --->  
									
	退出并保存。
    
4. 修改framework配置

	a. device/allwinner/cubieboard/BoardConfig.mk
        因为手头的USB WIFI 芯片是8187的，有一大堆需要修改的：
        #WIFI_DRIVER_MODULE_PATH          := "/system/lib/modules/8192cu.ko"
        #WIFI_DRIVER_MODULE_NAME          := 8192cu
        WIFI_DRIVER_MODULE_PATH          := "/system/lib/modules/rtl8187.ko"         #新增
        WIFI_DRIVER_MODULE_NAME          := rtl8187                                                                #新增

        SW_BOARD_USR_WIFI := rtl8187                                                                                        #新增
        BOARD_WLAN_DEVICE := rtl8187                                                                                        #新增

        #SW_BOARD_USR_WIFI := rtl8192cu
        #BOARD_WLAN_DEVICE := rtl8192cu

        最后检查一下，要确保这个生效： BOARD_WIFI_VENDOR := realtek。

	b. device/allwinner/cubieboard/camera.cfg
        修改配置，把默认CSI的CAMERA设备文件（/dev/video1）改成UVC的CAMERA 设备文件：/dev/video0
        camera_device = /dev/video0

	c. hardware/libhardware_legacy/wifi/wifi.c
        在8188前面增加配置：
        #elif defined RTL_8187_WIFI_USED
        /* rtl8192cu usb wifi */
        #ifndef WIFI_DRIVER_MODULE_PATH
        #define WIFI_DRIVER_MODULE_PATH         "/system/lib/modules/rtl8187.ko"
        #endif
        #ifndef WIFI_DRIVER_MODULE_NAME
        #define WIFI_DRIVER_MODULE_NAME         "rtl8187"
        #endif

	d. hardware/libhardware_legacy/wifi/Android.mk
        增加：
        ifeq ($(SW_BOARD_USR_WIFI), rtl8187)
        LOCAL_CFLAGS += -DRTL_8187_WIFI_USED
        LOCAL_CFLAGS += -DRTL_WIFI_VENDOR
        endif

	e.device/allwinner/cubieboard/init.sun4i.rc
        在boot下追加：
        insmod /system/lib/modules/uvcvideo.ko
        insmod /system/lib/modules/mac80211.ko
        insmod /system/lib/modules/eeprom_93cx6.ko
        mac80211.ko 和 eeprom_93cx6.ko 是8187依赖的模块，网络和led灯，确保启动后能挂载。

	f.tools/pack/chips/sun4i/configs/crane/cubieboard/sys_config1.fex
        启用串口4，找到 uart_para4 ，uart_used = 1即可打开。
        查询硬件图纸，应该可以找到对应U15模块的 17和18号的引脚为TX 和 RX。
        port 默认是PH04 和PH05，但是实际上并非如此，对应的应该是PG10和PG11.
        检查一下CSI1 是used 状态，并占用了PG00 -PG12，CSI1因为没有用到，就可以关闭了。

	g.device/allwinner/commom/hardware/realtek/wlan/driver_cmd_nl80211.c
        从网络上参考到的修改，这个是无奈之举，会导致AP热点不能使用，但是STA模式可用。
        add "return 0;" line 214:
        vi device/allwinner/commom/hardware/realtek/wlan/driver_cmd_nl80211.c
        int wpa_driver_nl80211_driver_cmd(void *priv, char *cmd, char *buf,
							  size_t buf_len )
        {
			struct i802_bss *bss = priv;
			struct wpa_driver_nl80211_data *drv = bss->drv;
			struct ifreq ifr;
			android_wifi_priv_cmd priv_cmd;
			int ret = 0;
			//test for STA MODE
			return 0;
			if (os_strcasecmp(cmd, "STOP") == 0) {

5. 开始编译固件

    source build/envsetup.sh
	lunch
	4
	make
	可能出来的错误：
	1. frameworks/base/include/utils/KeyedVector.h:193:31: note: use ‘this->indexOfKey’ instead
	make: *** [out/host/linux-x86/obj/EXECUTABLES/aapt_intermediates/AaptAssets.o] Error 1
	fix:
	vi frameworks/base/tools/aapt/Android.mk
	Add '-fpermissive' to line 31:
	LOCAL_CFLAGS += -Wno-format-y2k -fpermissive
	
	2.frameworks/base/include/utils/KeyedVector.h:193:31: note: use ‘this->indexOfKey’ instead
	make: *** [out/host/linux-x86/obj/STATIC_LIBRARIES/libutils_intermediates/AssetManager.o] Error 1
	fix:
	vi frameworks/base/libs/utils/Android.mk
	Add '-fpermissive' to line 64:
	LOCAL_CFLAGS += -DLIBUTILS_NATIVE=1 $(TOOL_CFLAGS) -fpermissive
	
	3.external/srec/tools/thirdparty/OpenFst/fst/lib/connect.h:102:36: warning: unused parameter ‘arc’ [-Wunused-parameter]
	bool TreeArc(StateId s, const A &arc) { return true; }
								^
	make: *** [out/host/linux-x86/obj/EXECUTABLES/grxmlcompile_intermediates/grxmlcompile.o] Error 1
	fix:
	cd external/srec
	wget "https://github.com/CyanogenMod/android_external_srec/commit/4d7ae7b79eda47e489669fbbe1f91ec501d42fb2.diff"
	patch -p1 < 4d7ae7b79eda47e489669fbbe1f91ec501d42fb2.diff
	rm -f 4d7ae7b79eda47e489669fbbe1f91ec501d42fb2.diff
	cd ../..
	
	4.make[2]: *** [prepare3] Error 1
	make[1]: *** [sub-make] Error 2
	make[1]: Leaving directory `/home/sys3/work/A10/A10-android4.0/kernel/allwinner/common'
	fix:
	cd kernel/allwinner/common
	make mrproper
	cd ../../..
	
	5.scripts/kconfig/zconf.tab.c fatal error: zconf.tab.c: No such file or directory
	scripts/kconfig/zconf.hash.c fatal error: zconf.hash.c: No such file or directory
	scripts/kconfig/lex.zconf.c fatal error: lex.zconf.c: No such file or directory
	fix:
	cd kernel/allwinner/common
	cp scripts/kconfig/zconf.tab.c_shipped scripts/kconfig/zconf.tab.c
	cp scripts/kconfig/zconf.hash.c_shipped scripts/kconfig/zconf.hash.c
	cp scripts/kconfig/lex.zconf.c_shipped scripts/kconfig/lex.zconf.c
	cd ../../..
	
	6.dalvik/vm/native/dalvik_system_Zygote.cpp:217:43: error: ‘setrlimit’ was not declared in this scope
	 err = setrlimit(contents[0], &rlim);
									   ^
	make: *** [out/host/linux-x86/obj/SHARED_LIBRARIES/libdvm_intermediates/native/dalvik_system_Zygote.o] Error 1
	fix:
	vi dalvik/vm/native/dalvik_system_Zygote.cpp
	Add '#include <sys/resource.h>' to line 22:
	#include "Dalvik.h"
	#include "native/InternalNativePriv.h"
	#include <sys/resource.h>
	#include <signal.h>

	7./cubieboard/obj/STATIC_LIBRARIES/libstagefright_rtsp_intermediates/libstagefright_rtsp.a: No such file or directory
	make: *** [out/target/product/cubieboard/obj/SHARED_LIBRARIES/libCedarX_intermediates/LINKED/libCedarX.so] Error 1
	fix:
	vi frameworks/base/media/libstagefright/Android.mk 
	Add 'libstagefright_rtsp \' to line 91:
	libFLAC \
	libstagefright_rtsp \

	ifeq ($(CEDARX_DEBUG_FRAMEWORK),Y)

	8.BEGIN failed--compilation aborted at external/webkit/Source/WebCore/make-hash-tools.pl line 23.
	make: *** [out/target/product/cubieboard/obj/STATIC_LIBRARIES/libwebcore_intermediates/Source/WebCore/html/DocTypeStrings.cpp] Error 2
	fix:
	sudo apt-get install libswitch-perl 
	
	9.frameworks/base/include/utils/KeyedVector.h:193:31: note: use ‘this->indexOfKey’ instead
	make: *** [out/host/linux-x86/obj/STATIC_LIBRARIES/libRS_intermediates/rsFont.o] Error 1
	fix:
	vi frameworks/base/libs/rs/Android.mk
	Add '-fpermissive' to line 183
	LOCAL_CFLAGS += -Werror -Wall -Wno-unused-parameter -Wno-unused-variable -fpermissive
	
	10. internal:aramGenerator<typename Container::value_type> ValuesIn(
													  ^
	make: *** [out/host/linux-x86/obj/STATIC_LIBRARIES/libgtest_host_intermediates/gtest-all.o] Error 1
	fix:
	vi external/gtest/src/Android.mk
	Add '-fpermissive' to lines 52 and 70 (both lines contain same info)
	LOCAL_CFLAGS += -O0 -fpermissive


