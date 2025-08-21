## PermissionController from android-13.0.0_r31
### PermissionController 脱离源码在Android Studio的编译

### 支持说明
* 不试图改变项目本身的目录结构，而是通过添加额外的配置和依赖构建Gradle环境支持

## 使用命令编译
### 环境依赖
*  Gradle 7.5
*  JDK version 11

```
# 构建环境
gradle wrapper

# 打包编译
./gradlew assemble
```


## 在Android Studio上编译
### 推荐使用
*  Android Studio Koala & JDK version 11

#### 第一步：执行Android Studio上Build APK的操作, 编译出apk

#### 第二步：创建系统目录，并将apk push到该目录下。
### PS: 鉴于高版本PermissionController已经换成APEX进行迭代更新，所以无法对该目录进行push替换，需要自己创建系统路径，再对它进行覆盖。通过如下命令可以查看PermissionController是否为apex目录。
```
adb shell pm path com.android.permissioncontroller  
```
![avatar](images/apex_path.png)

####  创建/system/priv-app/PermissionController目录，并对apex实现覆盖更新。
```
adb root

adb remount

adb shell mkdir /system/priv-app/PermissionController

adb push PermissionController.apk /system/priv-app/PermissionController/

```

#### 第三步：需要push权限文件到etc目录下
```
adb push com.android.permissioncontroller.xml /system/etc/permissions/

adb reboot
```

## 构建步骤

### Step1：引入静态依赖
##### @framework.jar:
```
// android-13/out/target/common/obj/JAVA_LIBRARIES/framework_intermediates/classes-header.jar
compileOnly files('libs/framework.jar')
```
![avatar](images/framework.png)


##### @android.car-stubs.jar:
```
// android-13/out/soong/.intermediates/packages/services/Car/car-lib/android.car-stubs/android_common/turbine/android.car-stubs.jar
compileOnly files('libs/android.car-stubs.jar')
```
![avatar](images/android.car-stubs.png)


##### @libprotobuf-java-lite.jar:
```
// android-13/out/soong/.intermediates/external/protobuf/libprotobuf-java-lite/android_common/javac/libprotobuf-java-lite.jar
compileOnly files('libs/libprotobuf-java-lite.jar')
```
![avatar](images/libprotobuf-java-lite.png)


##### @libprotobuf-java-nano.jar:
```
// android-13/out/soong/.intermediates/external/protobuf/libprotobuf-java-nano/android_common/javac/libprotobuf-java-nano.jar
implementation files('libs/libprotobuf-java-nano.jar')
```
![avatar](images/libprotobuf-java-nano.png)


##### @permissioncontroller-statsd.jar:
```
// android-13/out/soong/.intermediates/packages/modules/Permission/PermissionController/permissioncontroller-statsd/android_common_apex30/javac/permissioncontroller-statsd.jar
implementation files('libs/permissioncontroller-statsd.jar')
```
![avatar](images/permissioncontroller-statsd.png)


##### @modules-utils-build_system.jar:
```
// android-13/out/soong/.intermediates/frameworks/libs/modules-utils/java/com/android/modules/utils/build/modules-utils-build_system/android_common/javac/modules-utils-build_system.jar
implementation files('libs/modules-utils-build_system.jar')
```
![avatar](images/modules-utils-build_system.png)



### Step2：引入Module
###### 需要将具体路径下的代码直接导入到项目中作为Module依赖, 构建的时候可以直接通过implementation project引用，或者也可以gradle build生成aar,再放置到libs文件夹中，作为静态包使用。

##### @iconloaderlib: 
```
// android-13/frameworks/libs/systemui/iconloaderlib
implementation project(':iconloaderlib')
```
![avatar](images/iconloaderlib.png)



##### @SettingsLib: 
```
// android-13/frameworks/base/packages/SettingsLib
include 'SettingsLib:HelpUtils'
include 'SettingsLib:RestrictedLockUtils'
include 'SettingsLib:AppPreference'
include 'SettingsLib:SearchWidget'
include 'SettingsLib:LayoutPreference'
include 'SettingsLib:BarChartPreference'
include 'SettingsLib:ActionBarShadow'
include 'SettingsLib:ProgressBar'
include 'SettingsLib:CollapsingToolbarBaseActivity'
include 'SettingsLib:SettingsTheme'
include 'SettingsLib:FooterPreference'
include 'SettingsLib:RadioButtonPreference'
include 'SettingsLib:TwoTargetPreference'
include 'SettingsLib:SettingsTransition'
include 'SettingsLib:Utils'
include 'SettingsLib:SelectorWithWidgetPreference'
```
![avatar](images/SettingsLib.png)


### Step3：将proto文件单独抽取出来，作为独立的目录进行引用，因为原路径下的proto指定了基于AOSP下的路径，会导致编译时引用失败。
```
sourceSets {
    main {
	......
	proto.srcDirs = ['protos']
	......
    }
}
```

## 生成platform.keystore默认签名

在android-13/build/target/product/security路径下找到签名证书，并使用 [keytool-importkeypair](https://github.com/getfatday/keytool-importkeypair) 生成keystore,
执行如下命令：  

```
./keytool-importkeypair -k platform.keystore -p 123456 -pk8 platform.pk8 -cert platform.x509.pem -alias platform
```

并将以下代码添加到gradle配置中：

```
    signingConfigs {
        platform {
            storeFile file("platform.keystore")
            storePassword '123456'
            keyAlias 'platform'
            keyPassword '123456'
        }
    }

    buildTypes {
        release {
            debuggable false
            minifyEnabled false
            signingConfig signingConfigs.platform
        }

        debug {
            debuggable true
            minifyEnabled false
            signingConfig signingConfigs.platform
        }
    }
```

---

### 关联项目
* [Settings](https://github.com/siren-ocean/Settings)
* [Launcher3](https://github.com/siren-ocean/Launcher3)
* [DocumentsUI](https://github.com/siren-ocean/DocumentsUI)
* [Camera2](https://github.com/siren-ocean/Camera2)
* [SystemUI](https://github.com/siren-ocean/SystemUI)
