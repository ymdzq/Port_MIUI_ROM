# 小米平板5 PRO 移植小米平板6 Pro 11英寸 HyperOS记录
资源来源于网络，仅供交流学习，不得用做任何商业用途，不提供任何技术支持，请在下载后24小时内删除  
基于ELISH_OS1.0.2.0，移植文件来源于LIUQIN_OS2.0.203.0  
本文仅记录一下修改内容，具体修改行以及内容以实际文件对比结果为准  

由于修改了系统文件，所以avb验证肯定是要关的。  
而想保证各种app兼容性，所以建议保持selinux enforce，要么保持5pro原版sepolicy放弃pc引擎，要么移植6的sepolicy。  
集成pc版wps需要一个支持erofs文件系统的内核，因为linux容器使用了erofs文件系统打包的img  
如果不集成，就不需要改vendor分区，随便在product分区里精简一点东西，就可以确保刷进机器那8.5G的super分区。  
## mi_ext分区修改，在5Pro的基础上，覆盖6Pro的所有文件
build.prop修改机型代号，这里这个代号是miui ota更新服务器用来识别推送更新用的，你都刷第三方rom了这个就不重要了，除非你能用到那个服务器推送更新  
把这个东西改掉的好处就是可以屏蔽更新，不会收到移植的那个机型的更新，导致用户误升级变砖  
修改版本号为UKYCNXM  
mi_ext\etc\build.prop
```
ro.product.mod_device=elish
ro.mi.os.version.incremental=OS2.0.203.0.VKYCNXM
```

这里提一句，比较新的机型的剃刀计划版本也比较新，支持卸载平板/手机管家，而版本不兼容就导致了部分机型移植完桌面没有平板/手机管家的图标，这里把有相关影响的内容列出来，这个部分提到的文件需要从6Max(yudi)的rom中提取  
mi_ext\etc\build.prop里面有一行`ro.miui.support.system.app.uninstall.v2=true`  
mi_ext\product\etc\permissions\platform-miui-uninstall.xml  
mi_ext\product\framework\miui-uninstall-empty.jar  
mi_ext\product\overlay\signed_PLATFORM_cf766d1e91_app_sec_overlay-release-unsigned.apk  

product\data-app\MIUISecurityManager\MIUISecurityManager.apk  
## odm分区
处理cit扬声器校准  
odm\etc\cit_param_config.json  
```
                "speaker_calibration_bin_str":"spkcal_elish",
                "speaker_calibration_cmds":["spkcal_elish -c","spkcal_elish -m"]
```
替换为
```
                "speaker_calibration_cmds": [
                    "spkcal  -c ",
                    "spkcal  -m "
                ]
```
spkcal_elish不支持安卓15，所以修复不了，可选spkcal_dagu代替，但是5pro有8个扬声器，dagu只能校准4个  
## 可选odm分区修改，补全PC版WPS
odm\bin\hw\mslgservice  
odm\bin\clear-cajdata.sh  
odm\bin\clear-wpsdata.sh  
odm\bin\losetup.sh  
odm\bin\start-rootfs.sh  
odm\bin\tar-rootfs.sh  
odm\etc\assets\md5.txt  
odm\etc\assets\mslgusrimg  
odm\etc\assets\rootfs-24.11.22.tgz  
odm\etc\init\mslgservice.rc  
odm\etc\selinux\precompiled_sepolicy  

修改tar-rootfs.sh中的验证机型
```
#删除
if [[ $device == "sheng" || $device == "pipa" || $device == "yudi" || $device == "liuqin" ]]; then
#改成
if [[ $device == "nabu" || $device == "elish" || $device == "enuma" || $device == "dagu" ]]; then
```
修改odm\etc\build.prop添加mslg
```
# Add xiaomi-wps-build-prop
ro.vendor.mslg.rootfs.version=rootfs-24.11.22.tgz
sys.mslg.available=1
```
可选补全PC版CAD，需要从小米平板7sPro或者小米平板7ultra提取  
odm\bin\clear-caddata.sh  
odm\bin\set_dns.sh  
odm\etc\assets\md5.txt  
odm\etc\assets\mslgusrimg  
odm\etc\assets\rootfs-25.04.23.tgz  
odm\etc\init\mslgservice.rc  

修改odm\etc\build.prop  
```
ro.vendor.mslg.rootfs.version=rootfs-25.04.23.tgz
```
## product分区修改，整体上照搬6Pro，但要注意以下部分
pc版wps相关文件  
访问linux容器的rdp后端MSLgRdp，PC 框架？和交互操作的前端WpsLauncher  
不集成pc版wps可以直接删除，集成则保留  
product\app\MSLgRdp   
product\data-app\WpsLauncher  
product\data-app\CAJLauncher  
product\data-app\CADLauncher  

product\app  
保留5pro小爱翻译 AiAsstVision  
（a13澎湃内置的版本号是4.6.0，可能需要使用模块解锁实时字幕功能）  
删除6Pro人脸识别解锁 Biometric  
保留5Pro人脸识别解锁 MiuiBiometric3373  
替换AnalyticsCore（来自白羊唐黎明）  

按需精简  
快应用服务引擎  
product\app\HybridPlatform  
智能服务  
product\app\MSA  

data-app可卸载的预装app，其中不少app都是可以在应用商店里重新安装的，  
product\data-app\  
因为平板5pro默认的super分区只有8.5G，而且重新打包必须预留更多空间，所以可以精简这里，把super精简到7.4G以下，越小越好  
我个人是觉得没必要搞极限精简，很多常用自带功能用户到时候又要想办法装回来，挺烦人的  
百度输入法小米版  
product\data-app\BaiduIME  
PC版CAJ阅读器  
product\data-app\CAJLauncher  
讯飞输入法小米版  
product\data-app\iFlytekIME  
小米创作  
product\data-app\Creation  
小米商城  
product\data-app\MiShop  
米兔儿童  
product\data-app\Mitukid  
多看阅读  
product\data-app\MIUIDuokanReaderPad  
电子邮件  
product\data-app\MIUIEmail  
游戏中心  
product\data-app\MIUIGameCenterPad  
小米云盘  
product\data-app\MIUIMiDrive  
小米社区  
product\data-app\OS2VipAccountPad  
小米有品  
product\data-app\MIUIYoupin  
小米汽车拓展屏  
product\data-app\Padapp  
米家  
product\data-app\SmartHome  

设备功能配置文件，本来正常代号要用elish稳定使用的话，删除liuqin.xml，照搬elish.xml就好了，  
但是如果你要全局改机器代号的话，这里配置文件也要改名成liuqin.xml，  
所以我是建议干脆把elish.xml复制两份一个叫elish.xml一个叫liuqin.xml，都放进去，这样用哪个代号也不要紧  
product\etc\device_features\elish.xml  
product\etc\device_features\liuqin.xml  
修改预装app列表（剃刀计划）
```
    <!--Add for the system data-app which could uninstall by user-->
    <string-array name="system_data_packagename_list">
        <item>com.xiaomi.pass</item>
        <item>com.xiaomi.scanner</item>
        <item>com.xiaomi.gamecenter</item>
        <item>com.miui.weather2</item>
        <item>com.miui.notes</item>
        <item>com.miui.compass</item>
        <item>com.miui.calculator</item>
        <item>com.android.email</item>
        <item>com.miui.cleanmaster</item>
        <item>com.mi.misupport</item>
        <item>com.duokan.reader</item>
        <item>com.mfashiongallery.emag</item>
        <item>com.miui.personalassistant</item>
        <item>com.miui.voip</item>
        <item>com.miui.yellowpage</item>
        <item>com.xiaomi.midrop</item>
        <item>com.android.midrive</item>
        <item>com.xiaomi.drivemode</item>
        <item>com.miui.smarttravel</item>
        <item>com.android.soundrecorder</item>
        <item>com.miui.screenrecorder</item>
    </string-array>
    <!--system data-app path list -->
    <string-array name="system_data_path_list">
        <item>/system/data-app/XMPass/XMPass.apk</item>
        <item>/system/data-app/MIUIScannerGlobal/MIUIScannerGlobal.apk</item>
        <item>/system/data-app/GameCenter/GameCenter.apk</item>
        <item>/system/data-app/MIUIWeatherGlobal/MIUIWeatherGlobal.apk</item>
        <item>/system/data-app/MIUINotes/MIUINotes.apk</item>
        <item>/system/data-app/MIUICompassGlobal/MIUICompassGlobal.apk</item>
        <item>/system/data-app/MIUICalculatorGlobal/MIUICalculatorGlobal.apk</item>
        <item>/system/data-app/Email/Email.apk</item>
        <item>/system/data-app/CleanMaster/CleanMaster.apk</item>
        <item>/system/data-app/MiSupport/MiSupport.apk</item>
        <item>/system/data-app/com.duokan.reader/com.duokan.reader.apk</item>
        <item>/system/data-app/MiGalleryLockscreen/MiGalleryLockscreen.apk</item>
        <item>/system/data-app/PersonalAssistant/PersonalAssistant.apk</item>
        <item>/system/data-app/MiuiVoip/MiuiVoip.apk</item>
        <item>/system/data-app/YellowPage/YellowPage.apk</item>
        <item>/system/data-app/MIDrop/MIDrop.apk</item>
        <item>/system/data-app/MiDrive/MiDrive.apk</item>
        <item>/system/data-app/MiuiDriveMode/MiuiDriveMode.apk</item>
        <item>/system/data-app/SmartTravel/SmartTravel.apk</item>
        <item>/system/data-app/MIUISoundRecorderTargetSdk30Global/MIUISoundRecorderTargetSdk30Global.apk</item>
        <item>/system/data-app/MIUIScreenRecorderLiteGlobal/MIUIScreenRecorderLiteGlobal.apk</item>
    </string-array>
    <!--global uninstallable system app package list-->
    <string-array name="global_uninstallable_system_packagename_list">
        <item>com.xiaomi.scanner</item>
        <item>com.miui.weather2</item>
        <item>com.miui.notes</item>
        <item>com.miui.compass</item>
        <item>com.miui.calculator</item>
        <item>com.xiaomi.midrop</item>
        <item>com.android.soundrecorder</item>
        <item>com.miui.screenrecorder</item>
    </string-array>

    <!-- 新版屏幕刷新率设置ui -->
    <!-- whether support fps change -->
    <bool name="support_smart_fps">true</bool>
    <!-- smart fps value-->
    <integer name="smart_fps_value">120</integer>
    <integer-array name="fpsList">
        <item>120</item>
        <item>60</item>
    </integer-array>

    <!-- 一些米板6功能，未测试是否生效，可能仅显示开关 -->
    <!-- whether support expert primary -->
    <bool name="need_remove_expert_primary">false</bool>
    <bool name="support_nature_mode">true</bool>
    <!-- whether support expert bright -->
    <bool name="need_remove_expert_bright">true</bool>
    <!-- whether support stylus quick note-->
    <bool name="stylus_quick_note">true</bool>
    <!-- device support screen enhance engine -->
    <bool name="support_screen_enhance_engine">true</bool>
    <!--Ignore installing the app in the current and below ram-->
    <string-array name="ignoredAppsForPackages">
        <item>16,com.xiaomi.drivemode</item>
    </string-array>
    <!-- Whether support Google rsa agreement -->
    <bool name="support_google_rsa_protocol">true</bool>
    <!-- whether support true color -->
    <bool name="support_true_color">true</bool>
    <!--Configuration for default color mode -->
    <integer name="default_display_color_mode">3</integer>

    <!-- gallery setting -->
    <bool name="gallery_support_media_feature">true</bool>
    <bool name="gallery_support_video_compress">true</bool>
    <bool name="gallery_support_analytic_face_and_scene">true</bool>
    <string name="gallery_cpu_series">8350</string>
    <bool name="gallery_support_time_burst_video">true</bool>
    <integer name="gallery_device_series">1</integer>
    <bool name="support_local_ocr">true</bool>
    <bool name="gallery_support_dolby">true</bool>

    <!-- Whether support dolby version brighten -->
    <bool name="support_dolby_version_brighten">true</bool>

    <!--  system firware related settings from bsp-adapt  -->
    <!--  these items are used by SecurityCenter  -->
    <!--  MIUI ADD: PKMS_ParentalControl  -->
    <bool name="support_parental_control">true</bool>
    <!-- END PKMS_ParentalControl -->
    <!-- Port ADD:  -->
    <bool name="support_hdr_enhance">true</bool>
    <!-- whether support AI Display-->
    <bool name="support_AI_display">true</bool>
    <!-- default rhythmic eyecare mode -->
    <integer name="default_eyecare_mode">2</integer>

```
关闭官方键盘支持（否则系统会以为你一直插着键盘，屏幕不显示安全输入法按键）
```
    <!-- whether support usb keyboard -->
    <bool name="support_usb_keyboard">true</bool>
改成
    <!-- whether support usb keyboard -->
    <bool name="support_usb_keyboard">false</bool>
```
修改屏幕亮度配置文件  
product\etc\displayconfig\display_id_4630947141052476290.xml  
product\etc\displayconfig\display_id_4630947200012256898.xml  

5pro屏幕的xml文件为：  
product\etc\displayconfig\display_id_19260527152667265.xml  
product\etc\displayconfig\display_id_4630946481717202305.xml  
product\etc\displayconfig\display_id_4630946545580055169.xml  
这三个文件的内容是完全一样的，所以我选择再复制两个替换display_id_4630947141052476290.xml和display_id_4630947200012256898.xml，保留这五个xml文件，屏幕亮度调节就正常了  
这里需要注意Overlay里面的AospFrameworkResOverlay.apk要换成5Pro的，否则会遇到自动亮度导致系统软重启的问题  
product\overlay\AospFrameworkResOverlay.apk  
需要apkeditor反编译修改，  
替换所有default_wallpaper.jpg，  
修改arrays.xml，添加  
```
  <string-array name="config_enabledCredentialProviderService">
    <item>com.miui.contentcatcher/com.miui.credential.provider.XiaomiCredentialProviderService</item>
    <item>com.google.android.gms/.auth.api.credentials.credman.service.PasswordAndPasskeyService</item>
  </string-array>
  <string-array name="config_primaryCredentialProviderService">
    <item>com.miui.contentcatcher/com.miui.credential.provider.XiaomiCredentialProviderService</item>
  </string-array>

  <string-array name="disallowed_apps_managed_profile">
    <item>com.miui.mishare.connectivity</item>
  </string-array>
```
修改bools.xml，添加  
```
  <bool name="config_dozeAlwaysOnEnabled">false</bool>
```
修改integers.xml  
```
  <integer name="config_screenBrightnessDim">13</integer>
  <integer name="config_screenBrightnessSettingDefault">307</integer>
  <integer name="config_screenBrightnessSettingMaximum">2047</integer>
  <integer name="config_screenBrightnessSettingMinimum">6</integer>
改成
  <integer name="config_screenBrightnessDim_hyper">13</integer>
  <integer name="config_screenBrightnessSettingDefault_hyper">307</integer>
  <integer name="config_screenBrightnessSettingMaximum_hyper">2047</integer>
  <integer name="config_screenBrightnessSettingMinimum_hyper">6</integer>
```
修改strings.xml，添加  
```
  <string name="config_defaultAttentionService">com.xiaomi.aon/com.xiaomi.aon.AonAttentionService</string>
  <string name="config_defaultCredentialManagerHybridService">com.miui.contentcatcher/com.miui.credential.provider.XiaomiCredentialProviderService</string>
```
修改public.xml，id我不确定，随便写的不重复新id  
```
添加
  <public id="0x7f01000c" type="array" name="config_enabledCredentialProviderService" />
  <public id="0x7f01000d" type="array" name="config_primaryCredentialProviderService" />
  <public id="0x7f01000b" type="array" name="disallowed_apps_managed_profile" />
  <public id="0x7f020007" type="bool" name="config_dozeAlwaysOnEnabled" />
  <public id="0x7f070003" type="string" name="config_defaultAttentionService" />
  <public id="0x7f070004" type="string" name="config_defaultCredentialManagerHybridService" />

  <public id="0x7f06000a" type="integer" name="config_screenBrightnessSettingMinimum" />
  <public id="0x7f060007" type="integer" name="config_screenBrightnessDim" />
  <public id="0x7f060008" type="integer" name="config_screenBrightnessSettingDefault" />
  <public id="0x7f060009" type="integer" name="config_screenBrightnessSettingMaximum" />
改成
  <public id="0x7f06000a" type="integer" name="config_screenBrightnessSettingMinimum_hyper" />
  <public id="0x7f060007" type="integer" name="config_screenBrightnessDim_hyper" />
  <public id="0x7f060008" type="integer" name="config_screenBrightnessSettingDefault_hyper" />
  <public id="0x7f060009" type="integer" name="config_screenBrightnessSettingMaximum_hyper" />
```

有几个东西我需要从pipa的rom里复制过来（因为都是骁龙870，比较接近）  
product\etc\android_dm_table_a  
product\etc\android_dm_table_b  

build.prop修改机型代号、版本指纹，设置默认屏幕密度，关闭内存扩展  
product\etc\build.prop
```
ro.product.product.name=elish
ro.product.build.fingerprint=Xiaomi/elish/miproduct:15/AQ3A.241006.001/OS2.0.203.0.VKYCNXM:user/release-keys
ro.product.build.version.incremental=OS2.0.203.0.VKYCNXM

persist.miui.density_v2=360
ro.sf.lcd_density=360

#默认关闭内存扩展
persist.miui.extm.enable=0

#修改ro.miui.cust_erofs改成
ro.miui.cust_erofs=0

#添加ro.millet.netlink
ro.millet.netlink=29

#开启高级材质选项
persist.sys.background_blur_supported=true
persist.sys.advanced_visual_release=2

#6max多了的两行玄学优化，平滑圆角
persist.sys.support_view_smoothcorner=true
persist.sys.support_window_smoothcorner=true

#修复heic
vendor.mm.enable.qcom_parser=16776951

#开启布局优化
persist.miui.auto_ui_enable=true

#游戏加载加速？
debug.game.video.speed=true
debug.game.video.support=true

#HDR修复？
persist.sys.support_ultra_hdr=true
persist.sys.hdr_dimmer_supported=true

#加回5Pro本身的玄学优化，性能调度
persist.sys.perf.cgroup8250.stune=true
persist.sys.miui.sf_cores=4-7
persist.vendor.display.miui.composer_boost=4-7
persist.sys.minfree_def=73728,92160,110592,154832,482560,579072
persist.sys.minfree_6g=73728,92160,110592,258048,663552,903168
persist.sys.minfree_8g=73728,92160,110592,387072,1105920,1451520

#内存扩展DM映射器回写块设备优化
persist.miui.extm.dm_opt.enable=true

#作用未知
persist.sys.launch_response_optimization.enable=true
ro.config.miui_compat_enable=true
ro.config.miui_appcompat_enable=true
ro.surface_flinger.game_default_frame_rate_override=120
ro.miui.shell_anim_enable_fcb=true
ro.audio.3d_play=true
```
内置完美横屏计划  
product\etc\autoui_list.xml  
product\etc\embedded_rules_list.xml  
product\etc\fixed_orientation_list.xml  

内置完美图标计划  
product\media\theme\default\dynamicicons  
product\media\theme\default\icons  
product\media\theme\default\miui_mod_icons\  

默认开启通信共享（来自Amktiao）  
product\media\theme\default\framework-miui-res  

保留5pro本身开机动画（分辨率匹配屏幕）  
product\media\bootanimation.zip  

overlay保留5pro本身设备的apk  
DevicesAndroidOverlay主要影响圆角弧率、状态栏高度，aod服务（lcd没有）  
product\overlay\DevicesAndroidOverlay.apk  
修改bools.xml，添加  
```
  <bool name="config_showBuiltinWirelessChargingAnim">false</bool>
  <bool name="config_volume_down_to_enter_silent">true</bool>
```
修改strings.xml，添加  
```
  <string name="config_mainDisplayShape">M20 0h1040s20 0 20 20v2360s0 20 -20 20h-1040s-20 0 -20 -20v-2360s0 -20 20 -20</string>
```
添加values-land\dimens.xml  
修改public.xml，id我不确定，随便写的不重复新id，添加  
```
  <public id="0x7f020004" type="bool" name="config_volume_down_to_enter_silent" />
  <public id="0x7f020003" type="bool" name="config_showBuiltinWirelessChargingAnim" />
  <public id="0x7f040009" type="dimen" name="toast_y_offset" />
  <public id="0x7f070002" type="string" name="config_mainDisplayShape" />
```
DevicesOverlay主要影响导航栏（小白条）布局以及圆角，充电动画  
product\overlay\DevicesOverlay.apk  
需要apkeditor反编译修改，  
替换resources\package_1\res\drawable-nodpi\charge_animation_charge_icon.webp  
替换resources\package_1\res\drawable-nodpi\charge_animation_turbo_icon.webp  
替换resources\package_1\res\drawable-nodpi\wired_charge_video_bg_img.webp  
替换resources\package_1\res\drawable-nodpi\wired_strong_super_charge_video_bg_img.webp  
替换resources\package_1\res\drawable-nodpi\wired_super_charge_video_bg_img.webp  
删除resources\package_1\res\drawable-nodpi\charge_animation_turbo_tail_icon.webp  
替换resources\package_1\res\raw\wired_charge_video.mp4  
替换resources\package_1\res\raw\wired_quick_charge_video.mp4  
替换resources\package_1\res\raw\wired_strong.mp4  
添加resources\package_1\res\values-sw600dp-port\dimens.xml  
添加colors.xml  
修改dimens.xml  
```
  <dimen name="notification_min_height">132.0dp</dimen>
改成
  <dimen name="notification_min_height">114.0dp</dimen>
```
修改values-sw600dp\dimens.xml，添加  
```
  <dimen name="miui_keyguard_pin_view_password_entry_bg_corners">20.0dp</dimen>
```
修改public.xml，id我不确定，随便写的不重复新id  
```
删除
  <public id="0x7f040022" type="drawable" name="charge_animation_turbo_tail_icon" />
添加
  <public id="0x7f020009" type="color" name="dark_mode_icon_color_single_tone" />
  <public id="0x7f020010" type="color" name="light_mode_icon_color_single_tone" />
  <public id="0x7f03006f" type="dimen" name="miui_keyguard_pin_view_password_entry_bg_corners" />
```
MiuiFrameworkResOverlay主要影响屏幕hbm背光、hbm亮度曲线、以及一些网络制式的属性  
product\overlay\MiuiFrameworkResOverlay.apk  
可选修改，通信共享，跟上面那个framework-miui-res效果是一样的，二个方案选一即可  
修改bools.xml，添加  
```
  <bool name="config_celluar_shared_support">true</bool>
```
修改public.xml，id我不确定，随便写的不重复新id，添加  
```
  <public id="0x7f02000e" type="bool" name="config_celluar_shared_support" />
```
MiuiBiometricResOverlay人脸识别资源文件空包  
product\overlay\MiuiBiometricResOverlay.apk  
SettingsRroDeviceTypeOverlay修复我的设备里的认证信息  
product\overlay\SettingsRroDeviceTypeOverlay.apk  
需要apkeditor反编译修改，  
添加resources\package_1\res\drawable-440dpi\credentials_image_m2105k81ac.png  
添加resources\package_1\res\drawable-xhdpi\credentials_image_m2105k81ac.png  
添加resources\package_1\res\drawable-xxhdpi\credentials_image_m2105k81ac.png  
添加resources\package_1\res\drawable-xxxhdpi\credentials_image_m2105k81ac.png  
修改public.xml，id我不确定，随便写的不重复新id  
```
添加
  <public id="0x7f030011" type="drawable" name="credentials_image_m2105k81ac" />
```
指南针授权？  
product\pangu\system\etc\permissions\signature-permission-pangu.xml添加  
```
   <signature-permissions package="com.miui.compass">
      <permission name="android.permission.WRITE_SETTINGS" />
      <permission name="android.permission.WRITE_SECURE_SETTINGS" />
   </signature-permissions>
```
删除6Pro相机，否则会提示机型不匹配无法使用然后退出，  
目前澎湃2只能用5.3以上版本的相机，老apk无法使用，同样会提示机型不匹配无法使用然后退出，  
目前没有可用的官方签名通用相机，Sunshine0078修改5.3专版相机，其他选择只能用谷歌相机、骁龙相机这种第三方相机  
product\priv-app\MiuiCamera  
并且删除两个oat文件  
替换修改版桌面  
product\priv-app\MiuiHome  
并且删除两个oat文件  
替换修改版应用包安装组件  
product\priv-app\MIUIPackageInstallerVariants  
并且删除两个oat文件  
## 可选product分区修改，补全小米平板缺失的工具app
CarWith  
product\app\CarWith  
悬浮球  
product\app\MIUITouchAssistant  
小米锁屏画报  
product\data-app\MIGalleryLockscreen\MIGalleryLockscreen.apk  
指南针  
product\data-app\MIUICompass\MIUICompass.apk  
传送门  
product\priv-app\MIUIContentExtension\MIUIContentExtension.apk  
添加传送门所需权限  
product\etc\permissions\privapp-permissions-product.xml  
```
   <privapp-permissions package="com.miui.contentextension">
      <permission name="android.permission.WRITE_SECURE_SETTINGS" />
   </privapp-permissions>
```
## system分区不修改，直接照搬6Pro
可选修改  
从pipa的rom里复制过来一些调度文件  
system\system\etc\cgroups_8250_u_stune.json  
system\system\etc\cgroups_v1.json  
system\system\etc\task_profiles_8250_u_stune.json  
system\system\etc\task_profiles_v1.json  

根据用户反馈蓝牙LHDC无需修复，直接用6Pro的不用动  
system\system\apex\com.android.btservices.apex  
签名破解，要修改系统app，就需要修改services.jar文件，我这里使用的SYT_ROM工具提供的插件自动修改  
system\system\framework\services.jar  

可选修改，内置完美横屏计划（防止云控修改，感觉这里会影响selinux，不推荐修改）  
system\system\bin\project_treble_magic_window_service.sh  
system\system\etc\init\project_treble_magic_window_service.rc  
system\system\etc\ProjectTrebleMagicWindowService\autoui_list.xml  
system\system\etc\ProjectTrebleMagicWindowService\embedded_rules_list.xml  
system\system\etc\ProjectTrebleMagicWindowService\fixed_orientation_list.xml  
修改system\system\etc\selinux\plat_sepolicy.cil，添加  
```
(allow init system_file (file (execute execute_no_trans open read getattr)))
(allow init shell_exec (file (execute execute_no_trans open read getattr)))
(allow toolbox system_data_file (file (read open write getattr setattr create unlink relabelfrom relabelto)))
(allow toolbox self (capability (dac_read_search dac_override chown fowner fsetid linux_immutable)))
```

build.prop修改机型代号、版本指纹  
system\system\build.prop
```
ro.system.build.fingerprint=qti/missi/missi:15/AQ3A.241006.001/OS2.0.203.0.VKYCNXM:user/release-keys
ro.system.build.version.incremental=OS2.0.203.0.VKYCNXM
ro.build.version.incremental=OS2.0.203.0.VKYCNXM
ro.build.description=missi-user 15 AQ3A.241006.001 OS2.0.203.0.VKYCNXM release-keys

#玄学优化
#加密状态-已加密
ro.crypto.state=encrypted
#WiFi扫描时间修改
wifi.supplicant_scan_interval=250
#关闭蓝牙日志
vendor.bluetooth.startbtlogger=false
#关闭内核日志
persist.sys.offlinelog.kernel=false
#关闭MIUI内核日志
sys.miui.ndcd=off
#禁止错误检测
ro.kernel.android.checkjni=0
ro.kernel.checkjni=0
```
## system_ext分区不修改，直接照搬6Pro
可选修改  
build.prop修改机型代号、版本指纹  
system_ext\etc\build.prop
```
ro.system_ext.build.fingerprint=qti/missi/missi:15/AQ3A.241006.001/OS2.0.203.0.VKYCNXM:user/release-keys
ro.system_ext.build.version.incremental=OS2.0.203.0.VKYCNXM

#完美横屏主动适配（需要对应修改system_ext\framework\miui-embedding-window.jar）
ro.config.sothx_project_treble_support_magic_window_fix=true

#完美横屏解锁工作台模式无极小窗限制（需要对应修改system_ext\framework\miui-framework.jar和system_ext\priv-app\MiuiSystemUI\MiuiSystemUI.apk）
ro.config.sothx_project_treble_support_cvw_full=true
ro.config.sothx_project_treble_cvw_full_version=1
```
zram配置文件，改末尾  
system_ext\etc\perfinit_bdsize_zram.conf
```
            "product_name": ["manet", "vermeer"],
            "zram_size": {
                "16":16384
```
改成
```
            "product_name": ["manet", "vermeer"],
            "zram_size": {
                "16":16384
            }
        },
        {
            "product_name": ["nabu", "elish", "enuma", "dagu", "pipa", "liuqin", "yudi"],
            "zram_size": {
                "6":6144, "8":8192, "12":12288, "16":16384
```
完美横屏主动适配，修改system_ext\framework\miui-embedding-window.jar  
通过MT管理器反编译classes.edx，搜索：  
```
invoke-static {}, Landroid/sizecompat/MiuiSizeCompatManager;->getMiuiSizeCompatEnabledApps()Ljava/util/Map;

move-result-object v2
```
替换为
```
invoke-static {}, Landroid/sizecompat/MiuiSizeCompatManager;->getMiuiSizeCompatEnabledApps()Ljava/util/Map;

move-result-object v2

const/4 v2, 0x0
```
完美横屏解锁工作台模式无极小窗限制，感谢绵狗 做梦书  
miui-framework.jar\smali\classes\android\util\MiuiMultiWindowAdapter.smali  
搜索并从.method public static blacklist getFreeformBlackList()Ljava/util/List;一直到下面第一个.end method的内容全删除，替换为
```
.method public static blacklist getFreeformBlackList()Ljava/util/List;
    .registers 1

    .line 1
    sget-object v0, Landroid/util/MiuiMultiWindowAdapter;->sEmptyList:Ljava/util/List;
    return-object v0
.end method
```
MiuiSystemUI.apk\smali\classes3\com\android\wm\shell\common\split\SplitScreenUtils.smali  
搜索.method public static isLeftRightSplit然后在这一行上面的空白处添加代码（只要在.method和上一个.end method之间插入就行，不一定要认准isLeftRightSplit，只要不破坏插.method开始.end method结束的规则，插在哪一个方式前面都没区别）
```
.method public static getVerticalSplitValue()I
    .registers 5

    .line 1
    invoke-static {}, Landroid/app/ActivityThread;->currentApplication()Landroid/app/Application;
    move-result-object v0

    if-eqz v0, :not_enabled

    const-string v1, "sothx_project_treble_vertical_screen_split_enable"

    invoke-virtual {v0}, Landroid/content/Context;->getContentResolver()Landroid/content/ContentResolver;
    move-result-object v2

    const/4 v3, 0x0

    invoke-static {v2, v1, v3}, Landroid/provider/Settings$System;->getInt(Landroid/content/ContentResolver;Ljava/lang/String;I)I
    move-result v1

    if-eqz v1, :not_enabled

    const/16 v0, 0x400
    goto :end

    :not_enabled
    const/16 v0, 0x258

    :end
    return v0
.end method
```
搜索const/16 v1, 0x258替换为
```
    invoke-static {}, Lcom/android/wm/shell/common/split/SplitScreenUtils;->getVerticalSplitValue()I
    move-result v1
```
MiuiSystemUI.apk\smali\classes3\com\android\wm\shell\multitasking\miuiinfinitymode\MiuiInfinityModeSizeLevelConfig.smali  
搜索.method public getLevelType然后在这一行上面的空白处添加代码（只要在.method和上一个.end method之间插入就行，不一定要认准getLevelType，只要不破坏插.method开始.end method结束的规则，插在哪一个方式前面都没区别）
```
.method public static getCvwFullEnabled()Z
    .registers 5

    .line 1
    invoke-static {}, Landroid/app/ActivityThread;->currentApplication()Landroid/app/Application;
    move-result-object v0

    if-eqz v0, :return_false

    const-string v1, "sothx_project_treble_cvw_full_enable"

    invoke-virtual {v0}, Landroid/content/Context;->getContentResolver()Landroid/content/ContentResolver;
    move-result-object v2

    const/4 v3, 0x0

    invoke-static {v2, v1, v3}, Landroid/provider/Settings$System;->getInt(Landroid/content/ContentResolver;Ljava/lang/String;I)I
    move-result v1

    if-eqz v1, :return_false

    const/4 v0, 0x1  # true
    return v0

:return_false
    const/4 v0, 0x0  # false
    return v0
.end method
```
搜索"getLevelType unsupport cvw2.0 leve full packageName="在接下来的第一个move-result p0的下一行插入新代码  
```
    invoke-static {}, Lcom/android/wm/shell/multitasking/miuiinfinitymode/MiuiInfinityModeSizeLevelConfig;->getCvwFullEnabled()Z

    move-result p0
```
修改resources\package_1\res\values\styles.xml  
resources\package_1\res\values-night\styles.xml  
兼容小米错误的资源数据-@style/null  
```
  <style name="ShortcutHelperTheme" parent="@style/null">
替换为
  <style name="ShortcutHelperTheme" parent="">
```
## vendor分区修改，整体上用5pro的，但要注意以下部分
从米板6标准版的OS2.0.7.0提取以下文件替换，修复selinux权限  
vendor\etc\selinux\plat_pub_versioned.cil  
vendor\etc\selinux\vendor_file_contexts  
vendor\etc\selinux\vendor_hwservice_contexts  
vendor\etc\selinux\vendor_property_contexts  
vendor\etc\selinux\vendor_sepolicy.cil  
vendor\etc\selinux\vendor_service_contexts  

修改fstab.qcom，加入erofs文件系统挂载（需配合第三方内核）
vendor\etc\fstab.qcom
```
system                                                  /system                erofs   ro                                                   wait,slotselect,avb=vbmeta_system,logical,first_stage_mount,avb_keys=/avb/q-gsi.avbpubkey:/avb/r-gsi.avbpubkey:/avb/s-gsi.avbpubkey
system_ext                                              /system_ext            erofs   ro                                                   wait,slotselect,avb=vbmeta_system,logical,first_stage_mount
product                                                 /product               erofs   ro                                                   wait,slotselect,avb=vbmeta_system,logical,first_stage_mount
vendor                                                  /vendor                erofs   ro                                                   wait,slotselect,avb,logical,first_stage_mount
odm                                                     /odm                   erofs   ro                                                   wait,slotselect,avb,logical,first_stage_mount
mi_ext                                                  /mnt/vendor/mi_ext     erofs   ro                                                   wait,slotselect,avb=vbmeta,logical,first_stage_mount,nofail

```
从6标准版的OS2.0.7.0提取以下文件替换，修复蓝牙耳机播放视频音画不同步bug，感谢云彩之枫  
vendor\lib\libbluetooth_audio_session_qti.so  
vendor\lib\libbluetooth_audio_session_qti_2_1.so  
vendor\lib64\libbluetooth_audio_session_qti.so  
vendor\lib64\libbluetooth_audio_session_qti_2_1.so  

vendor/build.prop加入代码  
```
vendor.audio.offload.track.enable=true

#玄学优化代码
# fix the drop frame issus
ro.surface_flinger.enable_frame_rate_override=false
debug.sf.auto_latch_unsignaled=0
vendor.display.enable_display_extensions=1
```
## boot分区，用5pro的，不用改
## vendor_boot分区，用5pro的，不用改
## 可选boot分区修改，kernel image替换为第三方内核
## 可选dtbo分区，直接用第三方内核dtbo.img替换
## 可选vendor_boot分区修改
修改fstab.qcom，加入erofs文件系统挂载（需配合第三方内核）  
vendor_boot\ramdisk\first_stage_ramdisk\fstab.qcom  
直接用上面改好的vendor\etc\fstab.qcom替换  

第三方内核配套的dtb文件  
dtb  

第三方内核配套的cmdline参数  
header
```
cmdline=console=ttyMSM0,115200n8 androidboot.hardware=qcom androidboot.console=ttyMSM0 androidboot.memcg=1 lpm_levels.sleep_disabled=1 video=vfb:640x400,bpp=32,memsize=3072000 msm_rtb.filter=0x237 service_locator.enable=1 androidboot.usbcontroller=a600000.dwc3 swiotlb=2048 loop.max_part=7 cgroup.memory=nokmem,nosocket cgroup_disable=pressure reboot=panic_warm cnss2.disable_nv_mac=1 quiet audit=0 mitigations=off kpti=off ssbd=force-off noirqdebug nodebugmon buildvariant=user
```
## 重新打包mi_ext、odm、system、system_ext、vendor、product分区
先用make_ext4fs或者e2fsdroid+mke2fs打包为raw image，  
然后用lpmake打包成super img  
vab机器一般是线刷用fastboot刷进super分区，卡刷是在recovery里用卡刷脚本写入到super分区，  
常见的情况也有使用zstd工具把super压缩成zst格式（打包zst需要raw格式的super.img），在线刷、卡刷的时候再解压，这种用压缩解压的时间来节省刷机包占用空间大小的做法，  
这种的情况就需要专门的脚本和工具了  
由于无wps版由于不需要修改odm、vendor分区，所以理论上其实你可以直接用fastbootd模式刷入mi_ext、system、system_ext、product分区  
集成wps会因为空间不够打包失败，所以需要精简更多文件，或者使用erofs文件系统压缩打包系统分区  
dsu包的做法就是直接把mi_ext、system、system_ext、product分区的raw image文件打包成一个zip或者gz文件即可  
解包打包偷懒就找个安卓工具箱，SYT、米欧、dna、多幸运之类的，直接一键打包  
## 关闭avb验证  
可选，修改fstab.qcom去除avb代码  
vendor\etc\fstab.qcom  
把system那一行的flags从`,avb_keys=`开始把后面的内容全删除，所有`,avb=vbmeta_system`删除，所有`,avb=vbmeta`删除，  

修改vbmeta.img、vbmeta_system.img，关闭avb验证，这玩意得用十六进制编辑器或者打包工具修改，  
我看米欧是修改的十六进制0000007B这个地址00改成02，这个改法跟下面两条命令是同样的效果  
另一个办法，用户刷入vbmeta、vbmeta_system时使用命令关闭avb验证或者在twrp中直接用选项关闭
```bash
fastboot --disable-verity --disable-verification flash vbmeta vbmeta.img
fastboot --disable-verity --disable-verification flash vbmeta_system vbmeta_system.img
```
