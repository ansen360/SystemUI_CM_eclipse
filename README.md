## 导入SystemUI到eclipse
> SystemUI路径: repo/frameworks/base/packages/SystemUI/
## 一·通过Android.mk文件查找依赖库
```
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)

LOCAL_MODULE_TAGS := optional

LOCAL_SRC_FILES := $(call all-java-files-under, src) \
    src/com/android/systemui/EventLogTags.logtags

LOCAL_STATIC_JAVA_LIBRARIES := Keyguard \
        android-support-v4
LOCAL_JAVA_LIBRARIES := telephony-common

LOCAL_PACKAGE_NAME := SystemUI
LOCAL_CERTIFICATE := platform
LOCAL_PRIVILEGED_MODULE := true

LOCAL_PROGUARD_FLAG_FILES := proguard.flags

LOCAL_RESOURCE_DIR := \
    frameworks/base/packages/Keyguard/res \
    $(LOCAL_PATH)/res
LOCAL_AAPT_FLAGS := --auto-add-overlay --extra-packages com.android.keyguard

ifneq ($(SYSTEM_UI_INCREMENTAL_BUILDS),)
    LOCAL_PROGUARD_ENABLED := disabled
    LOCAL_JACK_ENABLED := incremental
endif

include frameworks/base/packages/SettingsLib/common.mk

include $(BUILD_PACKAGE)

#ifeq ($(EXCLUDE_SYSTEMUI_TESTS),)
#    include $(call all-makefiles-under,$(LOCAL_PATH))
#endif

```
- 其中LOCAL_STATIC_JAVA_LIBRARIES为引用的静态库(静态库是需要编译进apk的)
> 静态库有: Keyguard android-support-v4
- LOCAL_JAVA_LIBRARIES为非静态库(非静态库是Android系统自带的库)
> 非静态库有: telephony-common
- LOCAL_AAPT_FLAGS := --auto-add-overlay --extra-packages com.android.keyguard(合并keyguard的资源文件)

**其中Keyguard和SettingsLib是包含res的工程库,因此不能导入jar包**

**从Android.mk文件得出结论是: systemUI依赖SettingsLib和keyguard两个工程(带有res文件),和telephony-common和android-support-v4两个jar包**

## 二.查找对应的jar和源码
- **导入SettingsLib源码**
> 路径: /msm8976/repo/frameworks/base/packages/SettingsLib/

(1). 将SettingsLib导入Eclipse,根据import报错,需要framework.jar,通过UserLibraries方式导入framework.jar(同时需要配置优先使用这个库(top一下))如下图
> framework.jar的路径:

>/msm8976/repo/out/target/common/obj/JAVA_LIBRARIES/framework_intermediates/classes.jar修改名字即可

![](http://oma689k8f.bkt.clouddn.com/note/1/1.png)
(2). 根据如下报错找到对应源码,然后在/msm8976/repo/out/target/common/obj/JAVA_LIBRARIES/core-libart_intermediates/下找到class.jar,修改名字即可
> 报错代码路径: /msm8976/repo/libcore/luni/src/main/java/libcore/icu

![](http://oma689k8f.bkt.clouddn.com/note/1/2.png)

SettingsLib报错解决,把SettingsLib设置成Is Library

- **导入Keyguard源码**

> 路径: /msm8976/repo/frameworks/base/packages/Keyguard

(1).同上导入framework.jar

(2).根据报错发现需要依赖SettingsLib工程

(3)minSdkVersion和targetSdkVersion修改成23

Keyguard报错解决,把Keyguard设置成Is Library


- **SystemUI引用SettingsLib和Keyguard源码**
- **SystemUI同上导入framework.jar**
- **根据以下报错,SystemUI同上导入libcore.jar**

![](http://oma689k8f.bkt.clouddn.com/note/1/3.png)

- **根据以下报错,导入telephony-common.jar**
> telephony-common路径:

> /msm8976/repo/out/target/common/obj/JAVA_LIBRARIES/telephony-common_intermediates/classes.jar修改名字即可
![](http://oma689k8f.bkt.clouddn.com/note/1/5.png)

- **根据以下报错,缺失EventLogTags.java,在源码从搜索找对该类复制到SystemUI对应包下(/com/android/systemui/EventLogTags.java)**

![](http://oma689k8f.bkt.clouddn.com/note/1/4.png)
![](http://oma689k8f.bkt.clouddn.com/note/1/6.png)

**clean一下,SystemUI报错全部解决完成,能够在windows下编译出apk,由于SystemUI的特殊性,没有系统签名push到系统依然用不了(DeskClock等没有系统签名也是能push使用的)**
![](http://oma689k8f.bkt.clouddn.com/note/1/7.png)
## 三.配置ANT编译脚本和系统签名
- ANT脚本在github已上传,我们一般需要配置的以下信息:
``` 
SDK路径:
<property name="sdk-folder" value="C:\ADT\sdk" />
第三方引用的工程目录:
<property name="library-dir1" value="../Keyguard" />
<property name="library-dir2" value="../SettingsLib" />
外部类库所在目录:
<property name="external-lib" value="libs" />  
<property name="ext-lib" value="ext_libs" />  
证书文件及所在目录:
<property name="sign-folder" value="ext_libs/security" /> 
<property name="keystore-file" value="${sign-folder}/platform.keystore" />
<property name="keystore-debug-file" value="${sign-folder}/platform.keystore" />
<property name="keystore-jar" value="${sign-folder}/signapk.jar" /> 
<property name="keystore-pk8" value="${sign-folder}/platform.pk8" /> 
<property name="keystore-pem" value="${sign-folder}/platform.x509.pem" /> 
引用的外部库:
<path id="lib_classpath">
         <!-- <fileset dir="${jar-path}" includes="*.jar" /> -->
		<fileset dir="${ext-lib}">
  	        <include name="android-support-v4.jar" />
         	<include name="core-libart.jar" />
			<include name="frameworks.jar" />
         	<include name="javalib.jar" />
			<include name="service.jar" />
         	<include name="telephony-common.jar" />
        </fileset>
        <path refid="libs_classpath"/>
        <pathelement location="${android-jar}" />
 
</path>


```


**下面是基于CM13 SystemUI的代码地址,已在eclipse上编译以及签名apk:**

https://github.com/ansen360/SystemUI_CM_eclipse
> CM代码相比高通代码多引用几个jar包,都是根据报错查找对应的jar


[Android系统源码framework SystemUI导入eclipse编译](http://blog.csdn.net/qq_25804863/article/details/48669667)

[Android系统源码Settings导入eclipse](http://blog.csdn.net/qq_25804863/article/details/48669477)

