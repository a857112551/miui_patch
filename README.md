# miui_patch

个人兴趣项目存档，使用 apktool 魔改 MIUI ROM
​
## 环境依赖
- [Java](http://www.oracle.com/technetwork/pt/java/javase/downloads/index.html)
- [Apktool](https://ibotpeaches.github.io/Apktool/install/)
​
## app
- [小米云服务](https://github.com/a857112551/miui_patch/tree/v9.5/patch/app/CloudService.md)
- [文件管理](https://github.com/a857112551/miui_patch/tree/v9.5/patch/app/FileExplorer.md)
- [个性主题](https://github.com/a857112551/miui_patch/tree/v9.5/patch/app/ThemeManager.md)
- [小米帐号](https://github.com/a857112551/miui_patch/tree/v9.5/patch/app/XiaomiAccount.md)
​
## priv-app
- [日历](https://github.com/a857112551/miui_patch/tree/v9.5/patch/priv-app/Calendar.md)
- [垃圾清理](https://github.com/a857112551/miui_patch/tree/v9.5/patch/priv-app/CleanMaster.md)
- [联系人](https://github.com/a857112551/miui_patch/tree/v9.5/patch/priv-app/Contacts.md)
- [传送门](https://github.com/a857112551/miui_patch/tree/v9.5/patch/priv-app/ContentExtension.md)
- [下载管理](https://github.com/a857112551/miui_patch/tree/v9.5/patch/priv-app/DownloadProviderUi.md)
- [MIUI软件包安装程序](https://github.com/a857112551/miui_patch/tree/v9.5/patch/priv-app/MiuiPackageInstaller.md)
- [状态栏（系统用户界面）](https://github.com/a857112551/miui_patch/tree/v9.5/patch/priv-app/MiuiSystemUI.md)
- [短信](https://github.com/a857112551/miui_patch/tree/v9.5/patch/priv-app/Mms.md)
- [负一屏（信息助手）](https://github.com/a857112551/miui_patch/tree/v9.5/patch/priv-app/PersonalAssistant.md)
- [开机引导](https://github.com/a857112551/miui_patch/tree/v9.5/patch/priv-app/Provision.md)
- [安全中心](https://github.com/a857112551/miui_patch/tree/v9.5/patch/priv-app/SecurityCenter.md)
- [设置](https://github.com/a857112551/miui_patch/tree/v9.5/patch/priv-app/Settings.md)
- [天气](https://github.com/a857112551/miui_patch/tree/v9.5/patch/priv-app/Weather.md)
​
## framework
- [services.jar](https://github.com/a857112551/miui_patch/tree/v9.5/patch/app/framework/services.md)

> 如果你想在手机上操作，可使用 [MT管理器](https://www.coolapk.com/apk/bin.mt.plus) 。

## 准备工作
访问 [MIUI官网](http://www.miui.com/download.html) 下载 MIUI ROM 卡刷包，使用 [SuperR's Kitchen](https://forum.xda-developers.com/apps/superr-kitchen/kitchen-superr-s-kitchen-v1-1-50-v2-1-6-t3597434) 解包以及全局 deodex 处理，然后将以下目录（若存在）拷贝出来。
```
.
└── system
    ├── app
    ├── framework
    ├── priv-app
    └── vendor
        ├── app
        └── framework
```

## 应用 `miui-purify`
根据你使用的 MIUI 版本切换至相应 `miui-purify` 分支，使用 apktool 反编译需要修改的 APK，执行相应方案的修改并回编（注意保留源签名），然后替换到原来的APK。

> `miui-purify` 修改策略中的 `return false`、`return true` 针对布尔方法（函数名以字母 `Z` 结尾），`return null` 针对无返回值的方法（函数名以字母 `V` 结尾）。

`return true` 指将方法的函数体修改为以下代码：

```
.locals 1

const/4 v0, 0x1

return v0
```

`return false` 指将方法的函数体修改为以下代码：
```
.locals 1

const/4 v0, 0x0

return v0
```

`return null` 指将方法的函数体修改为以下代码：
```
.locals 0

return-void
```

## 删除 MIUI 冗余应用
请在 `/system/app` 目录下删除以下软件包（若存在）。

核心广告组件：
- AnalyticsCore
- mab
- MSA 
- SystemAdSolution

`miui-purify` 建议删除的软件：
- GameCenter
- MiGameCenterSDKService
- MiuiSuperMarket
- MiuiVideo
- Music
- QuickSearchBox
- Updater
- VipAccount
- XMPass

> 对于 `miui-purify` 没有给出修改方案却含有广告的APP，如小米视频、小米音乐、浏览器等，你可以将它们删除（不会影响系统的正常使用），选择更优秀的替代品。

## 制作 `miui-purify` 卡刷包
下载模板文件 `sample.zip` 解压，目录结构如下：
```
.
├── META-INF
│   └── com
│       └── google
│           └── android
│               ├── update-binary
│               └── updater-script  # 刷机脚本
└── system
    ├── app
    ├── framework
    ├── priv-app
    └── vendor
        ├── app
        └── framework
```
> `miui-purify` 通过全局替换 deodex 后的 APK/JAR 到系统达到修改的目的。

将你修改好的 `system` 目录下的相应内容覆盖到 `smaple/system` 目录下。

生成刷机脚本 `updater-script`，参考示例：
```
ifelse(is_mounted("/cust"), unmount("/cust"));
ifelse(is_mounted("/system"), unmount("/system"));

mount("ext4", "EMMC", "/dev/block/bootdevice/by-name/cust", "/cust", "max_batch_time=0,commit=1,data=ordered,barrier=1,errors=panic,nodelalloc");
delete_recursive("/cust/app");
delete_recursive("/cust/cust");
unmount("/cust");

ui_print("Extracting system...");
mount("ext4", "EMMC", "/dev/block/bootdevice/by-name/system", "/system", "max_batch_time=0,commit=1,data=ordered,barrier=1,errors=panic,nodelalloc");

# 清理旧内容
delete_recursive("/system/app");
delete_recursive("/system/data-app");
delete_recursive("/system/framework");
delete_recursive("/system/priv-app");
delete_recursive("/system/vendor/app");
delete_recursive("/system/vendor/framework");

# 如果你精简的APP包含动态so库，一并删除
# 可选，你不删也没事
delete("/system/lib64/libmtk_serialnum.so");
delete("/system/lib64/libxmpass_sdk_patcher.so");
delete("/system/lib64/xmpass_libweibosdkcore.so");

# 这是为了防止恢复官方recovery
delete("/system/bin/install-recovery.sh");
delete("/system/etc/recovery-resource.dat");
delete("/system/recovery-from-boot.p");

package_extract_dir("system", "/system");

# 链接动态so库，很重要
# 如果你替换的组件使用了动态so库，必须重新链接
# 可以参考 SuperR's Kitchen 生成的 symlinks
ui_print("Creating symlinks...");
symlink("/data/miui/miuisdk.apk", "/system/framework/miuisdk.jar");
symlink("/data/miui/miuisystem.apk", "/system/framework/miuisystem.jar");
symlink("/system/lib64/libbluetooth_jni.so", "/system/app/Bluetooth/lib/arm64/libbluetooth_jni.so");
symlink("/system/lib64/libcups.so", "/system/app/BuiltInPrintService/lib/arm64/libcups.so");
symlink("/system/lib64/libdefcontainer_jni.so", "/system/priv-app/DefaultContainerService/lib/arm64/libdefcontainer_jni.so");
symlink("/system/lib64/libdiag_jni.so", "/system/app/Cit/lib/arm64/libdiag_jni.so");
symlink("/system/lib64/libdict-parser.so", "/system/app/YouDaoEngine/lib/arm64/libdict-parser.so");
symlink("/system/lib64/libfaceunlock.so", "/system/priv-app/MiuiSystemUI/lib/arm64/libfaceunlock.so");
symlink("/system/lib64/libgamemaster.so", "/system/app/MiuiVpnSdkManager/lib/arm64/libgamemaster.so");
symlink("/system/lib64/libgnustl_shared.so", "/system/priv-app/MiuiSystemUI/lib/arm64/libgnustl_shared.so");
symlink("/system/lib64/libimscamera_jni.so", "/system/app/ims/lib/arm64/libimscamera_jni.so");
symlink("/system/lib64/libimsmedia_jni.so", "/system/app/ims/lib/arm64/libimsmedia_jni.so");
symlink("/system/lib64/libjni_pacprocessor.so", "/system/app/PacProcessor/lib/arm64/libjni_pacprocessor.so");
symlink("/system/lib64/libmiuiclassproxy.so", "/system/app/miui/lib/arm64/libmiuiclassproxy.so");
symlink("/system/lib64/libmiuidiffpatcher.so", "/system/app/miui/lib/arm64/libmiuidiffpatcher.so");
symlink("/system/lib64/libmiuiimageutilities.so", "/system/app/miui/lib/arm64/libmiuiimageutilities.so");
symlink("/system/lib64/libmiuinative.so", "/system/app/miui/lib/arm64/libmiuinative.so");
symlink("/system/lib64/libmivrnative.so", "/system/priv-app/MiVRFramework/lib/arm64/libmivrnative.so");
symlink("/system/lib64/libnfc_nci_jni.so", "/system/app/NfcNci/lib/arm64/libnfc_nci_jni.so");
symlink("/system/lib64/libopencv_java3.so", "/system/priv-app/MiuiSystemUI/lib/arm64/libopencv_java3.so");
symlink("/system/lib64/libpowerkeeper_jni.so", "/system/app/PowerKeeper/lib/arm64/libpowerkeeper_jni.so");
symlink("/system/lib64/libprintspooler_jni.so", "/system/app/PrintSpooler/lib/arm64/libprintspooler_jni.so");
symlink("/system/lib64/libqspower-1.0.0.so", "/system/priv-app/MiuiSystemUI/lib/arm64/libqspower-1.0.0.so");
symlink("/system/lib64/libSampleAuthJNI.so", "/system/app/SampleAuthenticatorService/lib/arm64/libSampleAuthJNI.so");
symlink("/system/lib64/libSampleExtAuthJNI.so", "/system/app/SampleExtAuthService/lib/arm64/libSampleExtAuthJNI.so");
symlink("/system/lib64/libSecureExtAuthJNI.so", "/system/app/SecureExtAuthService/lib/arm64/libSecureExtAuthJNI.so");
symlink("/system/lib64/libSecureSampleAuthJNI.so", "/system/app/SecureSampleAuthService/lib/arm64/libSecureSampleAuthJNI.so");
symlink("/system/lib64/libsim-activate-root-checker.so", "/system/app/XiaomiSimActivateService/lib/arm64/libsim-activate-root-checker.so");
symlink("/system/lib64/libSNPE.so", "/system/priv-app/MiuiSystemUI/lib/arm64/libSNPE.so");
symlink("/system/lib64/libtasft_jni.so", "/system/app/Cit/lib/arm64/libtasft_jni.so");
symlink("/system/lib64/libuptsmaddonmi.so", "/system/app/TSMClient/lib/arm64/libuptsmaddonmi.so");
symlink("/system/lib64/libwfds.so", "/system/app/BuiltInPrintService/lib/arm64/libwfds.so");
symlink("/system/lib64/libXMFD_FaceDetect.so", "/system/priv-app/MiuiSystemUI/lib/arm64/libXMFD_FaceDetect.so");
symlink("/system/lib/libfb.so", "/system/app/HybridPlatform/lib/arm/libfb.so");
symlink("/system/lib/libgifimage.so", "/system/app/HybridPlatform/lib/arm/libgifimage.so");
symlink("/system/lib/libimagepipeline.so", "/system/app/HybridPlatform/lib/arm/libimagepipeline.so");
symlink("/system/lib/libj2v8.so", "/system/app/HybridPlatform/lib/arm/libj2v8.so");
symlink("/system/lib/libsdk_patcher.so", "/system/app/HybridPlatform/lib/arm/libsdk_patcher.so");
symlink("/system/lib/libstatic-webp.so", "/system/app/HybridPlatform/lib/arm/libstatic-webp.so");
symlink("/system/lib/libttscompat.so", "/system/priv-app/PicoTts/lib/arm/libttscompat.so");
symlink("/system/lib/libttspico.so", "/system/priv-app/PicoTts/lib/arm/libttspico.so");
symlink("/system/lib/libyoga.so", "/system/app/HybridPlatform/lib/arm/libyoga.so");
symlink("/system/vendor/lib64/libmmi_jni.so", "/system/vendor/app/Qmmi/lib/arm64/libmmi_jni.so");

# 设置权限
ui_print("Setting Permissions...");
set_metadata_recursive("/system/app", "uid", 0, "gid", 0, "dmode", 0755, "fmode", 0644, "capabilities", 0x0, "selabel", "u:object_r:system_file:s0");
set_metadata_recursive("/system/framework", "uid", 0, "gid", 0, "dmode", 0755, "fmode", 0644, "capabilities", 0x0, "selabel", "u:object_r:system_file:s0");
set_metadata_recursive("/system/priv-app", "uid", 0, "gid", 0, "dmode", 0755, "fmode", 0644, "capabilities", 0x0, "selabel", "u:object_r:system_file:s0");
set_metadata_recursive("/system/vendor/app", "uid", 0, "gid", 2000, "dmode", 0755, "fmode", 0644, "capabilities", 0x0, "selabel", "u:object_r:system_file:s0");
set_metadata_recursive("/system/vendor/framework", "uid", 0, "gid", 2000, "dmode", 0755, "fmode", 0644, "capabilities", 0x0, "selabel", "u:object_r:system_file:s0");

unmount("/system");

ui_print("Done!");
set_progress(1.000000);
```

大功告成！现在你可以打包成 `zip` 卡刷包，并通过TWRP刷进你的手机~
