# 小米平板5 PRO 5G 移植小米平板6 11英寸 HyperOS记录
资源来源于网络，仅供交流学习，不得用做任何商业用途，不提供任何技术支持，请在下载后24小时内删除  
基于enuma_OS1.0.3.0，移植文件来源于PIPA_OS2.0.11.0  
本文仅记录一下修改内容，具体修改行以及内容以实际文件对比结果为准  

由于修改了系统文件，所以avb验证肯定是要关的。  
而想保证各种app兼容性，所以建议保持selinux enforce，要么保持5pro原版sepolicy放弃pc引擎，要么移植6的sepolicy。  
集成pc版wps需要一个支持erofs文件系统的内核，因为linux容器使用了erofs文件系统打包的img  
如果不集成，就不需要改vendor分区，随便在product分区里精简一点东西，就可以确保刷进机器那8.5G的super分区。  
## mi_ext分区修改，在5Pro的基础上，覆盖6的所有文件
build.prop修改机型代号，这里这个代号是miui ota更新服务器用来识别推送更新用的，你都刷第三方rom了这个就不重要了，除非你能用到那个服务器推送更新  
把这个东西改掉的好处就是可以屏蔽更新，不会收到移植的那个机型的更新，导致用户误升级变砖  
修改版本号为UKZCNXM  
mi_ext\etc\build.prop
```
ro.product.mod_device=enuma
ro.mi.os.version.incremental=OS2.0.11.0.UKZCNXM
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
                "speaker_calibration_bin_str":"spkcal_enuma",
                "speaker_calibration_cmds":["spkcal_enuma -c","spkcal_enuma -m"]
```
替换为
```
                "speaker_calibration_cmds": [
                    "spkcal  -c ",
                    "spkcal  -m "
                ]
```
spkcal_enuma不支持安卓15，所以修复不了，可选spkcal_dagu代替，但是5pro有8个扬声器，dagu只能校准4个  
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
odm\etc\selinux\precompiled_sepolicy.plat_sepolicy_and_mapping.sha256  
odm\etc\selinux\precompiled_sepolicy.product_sepolicy_and_mapping.sha256  
odm\etc\selinux\precompiled_sepolicy.system_ext_sepolicy_and_mapping.sha256  

修改tar-rootfs.sh中的验证机型
```
#删除
if [[ $device == "sheng" || $device == "pipa" || $device == "yudi" || $device == "liuqin" ]]; then
#改成
if [[ $device == "nabu" || $device == "enuma" || $device == "enuma" || $device == "dagu" ]]; then
```
修改odm\etc\build.prop添加mslg
```
# Add xiaomi-wps-build-prop
ro.vendor.mslg.rootfs.version=rootfs-24.11.22.tgz
sys.mslg.available=1
```
可选补全PC版CAD，需要从小米平板7Pro或者小米平板7SPro提取（感觉这里会影响selinux，不推荐添加set_dns相关代码）  
odm\bin\clear-caddata.sh  
odm\bin\set_dns.sh  
odm\etc\assets\md5.txt  
odm\etc\assets\mslgusrimg  
odm\etc\assets\rootfs-25.07.03.tgz  
odm\etc\init\mslgservice.rc  

修改odm\etc\build.prop  
```
ro.vendor.mslg.rootfs.version=rootfs-25.07.03.tgz
```
## product分区修改，整体上照搬6，但要注意以下部分
pc版wps相关文件  
访问linux容器的rdp后端MSLgRdp，PC 框架？和交互操作的前端WpsLauncher  
不集成pc版wps可以直接删除，集成则保留  
product\app\MSLgRdp   
product\data-app\WpsLauncher  
product\data-app\CAJLauncher  
product\data-app\CADLauncher  

product\app  
提取7Pro小爱翻译 AiAsstVision  
（pipa本来的小爱翻译是离线翻译模型，需要使用模块解锁实时字幕功能，否则整个翻译app无法打开，但解锁后支持elish和enuma使用中英文离线翻译。已测试lisa、ruan、dizi、muyu使用的小爱翻译是在线翻译模型，使用模块解锁实时字幕功能可以在线翻译多国语言，不解锁实时字幕elish和enuma也可以使用其他翻译功能）  
删除6人脸识别解锁 Biometric  
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
5G版因为要保留网络注册和短信功能，需要更多空间  
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

设备功能配置文件，本来正常代号要用enuma稳定使用的话，删除pipa.xml，照搬enuma.xml就好了，  
但是如果你要全局改机器代号的话，这里配置文件也要改名成pipa.xml，  
所以我是建议干脆把enuma.xml复制两份一个叫enuma.xml一个叫pipa.xml，都放进去，这样用哪个代号也不要紧  
product\etc\device_features\enuma.xml  
product\etc\device_features\pipa.xml  
修改预装app列表（剃刀计划）
```
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

    <!-- 支持熄屏听剧 -->
    <!-- whether remove screen off hold on feature -->
    <bool name="remove_screen_off_hold_on">false</bool>

    <!-- 支持语音通话工具箱 -->
    <!--whether the device supports conversation_tool_box voip record -->
    <bool name="support_conversation_toolbox_voiprecord">true</bool>

    <!-- 支持游戏HDR -->
    <!-- whether support displayfeature gamemode HDR -->
    <bool name="support_displayfeature_gamemode_HDR">true</bool>

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
修改屏幕亮度配置文件  
product\etc\displayconfig\display_id_4630946808805831297.xml  
product\etc\displayconfig\display_id_4630946922172900481.xml  

5pro屏幕的xml文件为：  
product\etc\displayconfig\display_id_19260527152667265.xml  
product\etc\displayconfig\display_id_4630946481717202305.xml  
product\etc\displayconfig\display_id_4630946545580055169.xml  
这三个文件的内容是完全一样的，所以我选择再复制两个替换display_id_4630946808805831297.xml和display_id_4630946922172900481.xml，保留这五个xml文件，屏幕亮度调节就正常了  
这里需要注意Overlay里面的AospFrameworkResOverlay.apk要换成5Pro的，否则会遇到自动亮度导致系统软重启的问题  
product\overlay\AospFrameworkResOverlay.apk  
需要apkeditor反编译修改，  
替换所有default_wallpaper.jpg，  
修改bools.xml  
```
可选修改
  <bool name="config_voice_capable">true</bool>
添加
  <bool name="config_dozeAlwaysOnEnabled">false</bool>
  <bool name="config_sms_capable">true</bool>
```
修改strings.xml  
```
添加
  <string name="config_defaultAttentionService">com.xiaomi.aon/com.xiaomi.aon.AonAttentionService</string>
```
修改public.xml，id我不确定，随便写的不重复新id  
```
添加
  <public id="0x7f020008" type="bool" name="config_sms_capable" />
  <public id="0x7f020007" type="bool" name="config_dozeAlwaysOnEnabled" />
  <public id="0x7f070003" type="string" name="config_defaultAttentionService" />
```
build.prop修改机型代号、版本指纹，设置默认屏幕密度，关闭内存扩展  
product\etc\build.prop
```
ro.product.product.name=enuma
ro.product.build.fingerprint=Xiaomi/enuma/miproduct:14/UKQ1.240624.001/OS2.0.11.0.UKZCNXM:user/release-keys
ro.product.build.version.incremental=OS2.0.11.0.UKZCNXM

ro.sf.lcd_density=360
persist.miui.density_v2=360

#默认关闭内存扩展
persist.miui.extm.enable=0

#开启高级材质选项
persist.sys.background_blur_supported=true
persist.sys.background_blur_version=2

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
persist.sys.adaptive_hdr_supported=true
persist.sys.hdr_dimmer_supported=true

#app调整游戏显示布局功能
ro.config.miui_compat_enable=true
ro.config.miui_appcompat_enable=true

#静置时维持刷新率时长
ro.surface_flinger.use_content_detection_for_refresh_rate=true
ro.surface_flinger.set_idle_timer_ms=2147483647
ro.surface_flinger.set_touch_timer_ms=2147483647
ro.surface_flinger.set_display_power_timer_ms=2147483647

#可升级系统app
persist.sys.allow_sys_app_update=true

#作用未知
ro.audio.3d_play=true
```
内置完美横屏计划  
product\etc\autoui_list.xml  
product\etc\embedded_rules_list.xml  
product\etc\fixed_orientation_list.xml  

内置完美横屏计划窗口控制器3.0附加文件（配合system_ext修改，可以屏蔽任意应用顶栏三个点）  
product\etc\dot_black_list.json

内置完美图标计划  
product\media\theme\default\dynamicicons  
product\media\theme\default\icons  
product\media\theme\default\miui_mod_icons\  

保留5pro本身开机动画（分辨率匹配屏幕）  
product\media\bootanimation.zip  

overlay保留5pro本身设备的apk  
DevicesAndroidOverlay主要影响圆角弧率、状态栏高度，aod服务（lcd没有）  
product\overlay\DevicesAndroidOverlay.apk  
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
修改public.xml，id我不确定，随便写的不重复新id  
```
删除
  <public id="0x7f040022" type="drawable" name="charge_animation_turbo_tail_icon" />
添加
  <public id="0x7f03006e" type="dimen" name="notification_panel_width_lockscreen" />
```
MiuiFrameworkResOverlay主要影响屏幕hbm背光、hbm亮度曲线、以及一些网络制式的属性  
MiuiBiometricResOverlay人脸识别资源文件空包  
product\overlay\MiuiBiometricResOverlay.apk  
SettingsRroDeviceTypeOverlay修复我的设备里的认证信息  
product\overlay\SettingsRroDeviceTypeOverlay.apk  
需要apkeditor反编译修改，  
添加resources\package_1\res\drawable-440dpi\credentials_image_m2105k81c.png  
添加resources\package_1\res\drawable-xhdpi\credentials_image_m2105k81c.png  
添加resources\package_1\res\drawable-xxhdpi\credentials_image_m2105k81c.png  
添加resources\package_1\res\drawable-xxxhdpi\credentials_image_m2105k81c.png  
修改public.xml，id我不确定，随便写的不重复新id  
```
添加
  <public id="0x7f030011" type="drawable" name="credentials_image_m2105k81c" />
```
删除6相机，否则会提示机型不匹配无法使用然后退出，  
目前澎湃只能用5.0以上版本的相机，老apk无法使用，同样会提示机型不匹配无法使用然后退出，  
直接小米11青春版（lisa）的5.1通用相机，其他选择只能用谷歌相机、骁龙相机这种第三方相机  
product\priv-app\MiuiCamera  
并且删除两个oat文件  
替换修改版应用包安装组件  
product\priv-app\MIUIPackageInstallerVariants  
并且删除两个oat文件  

5G版为了开启网络和短信功能，需要从手机rom提取所需文件，这里以从lisa提取举例  
product\app\remoteSimLockAuthentication\remoteSimLockAuthentication.apk  
product\app\XiaomiSimActivateService\XiaomiSimActivateService.apk  
product\pangu\system\etc\permissions\privapp-permissions-systemhelper.xml  
product\pangu\system\priv-app\SystemHelper\SystemHelper.apk  
product\priv-app\AutoRegistration\AutoRegistration.apk  
product\priv-app\MIUIContactsT\MIUIContactsT.apk  
product\priv-app\MiuiMms\MiuiMms.apk  
product\priv-app\RegService\RegService.apk  
product\overlay\AospFrameworkTelephonyResOverlay.apk
product\overlay\MiuiCarrierConfigOverlay.apk
product\overlay\MiuiTelephonyResOverlay.apk

删除product\priv-app\MIUIContactsPad  
修改product\etc\permissions\privapp-permissions-product.xml，添加  
```
   <privapp-permissions package="com.android.mms">
      <permission name="android.permission.READ_PRIVILEGED_PHONE_STATE" />
      <permission name="android.permission.READ_PHONE_STATE" />
      <permission name="android.permission.CALL_PRIVILEGED" />
      <permission name="android.permission.GET_ACCOUNTS_PRIVILEGED" />
      <permission name="android.permission.WRITE_SECURE_SETTINGS" />
      <permission name="android.permission.SEND_SMS_NO_CONFIRMATION" />
      <permission name="android.permission.SEND_RESPOND_VIA_MESSAGE" />
      <permission name="android.permission.UPDATE_APP_OPS_STATS" />
      <permission name="android.permission.MODIFY_PHONE_STATE" />
      <permission name="android.permission.WRITE_MEDIA_STORAGE" />
      <permission name="android.permission.MANAGE_USERS" />
      <permission name="android.permission.WRITE_APN_SETTINGS" />
      <permission name="android.permission.INTERACT_ACROSS_USERS" />
      <permission name="android.permission.SCHEDULE_EXACT_ALARM" />
   </privapp-permissions>
   <privapp-permissions package="com.miui.dmregservice">
      <permission name="android.permission.READ_PHONE_STATE" />
      <permission name="android.permission.READ_PRIVILEGED_PHONE_STATE" />
      <permission name="android.permission.MODIFY_PHONE_STATE" />
      <permission name="android.permission.WRITE_SECURE_SETTINGS" />
      <permission name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS" />
      <permission name="android.permission.PACKAGE_USAGE_STATS" />
      <permission name="android.permission.WRITE_APN_SETTINGS" />
      <permission name="android.permission.READ_NETWORK_USAGE_HISTORY" />
      <permission name="android.permission.LOCAL_MAC_ADDRESS" />
      <permission name="android.permission.SCHEDULE_EXACT_ALARM" />
   </privapp-permissions>
   <privapp-permissions package="com.xiaomi.registration">
      <permission name="android.permission.LOCAL_MAC_ADDRESS" />
      <permission name="android.permission.READ_PHONE_STATE" />
      <permission name="android.permission.READ_PRIVILEGED_PHONE_STATE" />
      <permission name="android.permission.READ_PRECISE_PHONE_STATE" />
   </privapp-permissions>
```

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
      <permission name="android.permission.READ_CLIPBOARD_IN_BACKGROUND" />
   </privapp-permissions>
```
第三方解锁ai功能补丁  
product\overlay\MiuiNotesOverlay.apk  
product\overlay\MiuiSoundrecorderOverlay.apk  
product\overlay\MiuiThememanagerOverlay.apk  
## system分区不修改，直接照搬6
可选修改  
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
system\system\system_dlkm\etc\build.prop
```
ro.system_dlkm.build.fingerprint=Android/missi_pad_cn/missi:14/UKQ1.240624.001/OS2.0.11.0.UKZCNXM:user/release-keys
ro.system_dlkm.build.version.incremental=OS2.0.11.0.UKZCNXM
```
system\system\build.prop
```
ro.system.build.fingerprint=Android/missi_pad_cn/missi:14/UKQ1.240624.001/OS2.0.11.0.UKZCNXM:user/release-keys
ro.system.build.version.incremental=OS2.0.11.0.UKZCNXM
ro.build.version.incremental=OS2.0.11.0.UKZCNXM

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
## system_ext分区不修改，直接照搬6
可选修改  
build.prop修改机型代号、版本指纹  
system_ext\etc\build.prop
```
ro.system_ext.build.fingerprint=Android/missi_pad_cn/missi:14/UKQ1.240624.001/OS2.0.11.0.UKZCNXM:user/release-keys
ro.system_ext.build.version.incremental=OS2.0.11.0.UKZCNXM

#完美横屏附加功能主动适配
ro.config.sothx_project_treble_support_magic_window_fix=true
ro.config.sothx_project_treble_support_cvw_full=true
ro.config.sothx_project_treble_cvw_full_version=2
ro.config.sothx_project_treble_support_disable_resize_black_list=true
ro.config.sothx_project_treble_disable_resize_black_list_version=1
ro.config.sothx_project_treble_support_default_desktop_mode_max_freeform_count=true
ro.config.sothx_project_treble_default_desktop_mode_max_freeform_count_version=1
ro.config.sothx_project_treble_support_miui_desktop_mode_max_freeform_count=true
ro.config.sothx_project_treble_miui_desktop_mode_max_freeform_count_version=1
ro.config.sothx_project_treble_support_disable_freeform_bottom_caption=true
ro.config.sothx_project_treble_disable_freeform_bottom_caption_version=1
ro.config.sothx_project_treble_support_immerse_freeform_bottom_caption=true
ro.config.sothx_project_treble_immerse_freeform_bottom_caption_version=1
ro.config.sothx_project_treble_support_custom_dot_black_list=true
ro.config.sothx_project_treble_custom_dot_black_list_version=1
```
zram配置文件，提取自7Pro OS3.0.0.16，添加平板5系列、6系列ram/zram容量1：1  
system_ext\etc\perfinit.conf
```
{
    "common": {
        "swap_on": 1,
        "global_swappiness": 100,
        "page_cluster": -1,
        "zram_size": {
            "def":512,"2":1024,"3":1536,"4":2252,"6":4096,"8":6144,"10":6144,"12":8192,"16":14336,"18":14336,"20":15360,"24":15360,"32":16384
        },
        "extm_on": 1,
        "extm_size": {
            "def":1024, "high_device":3072, "3+64":1024, "4+64":1024, "6+64":2048, "4+128":2048, "6+128":2048
        },
        "extm_file": "/data/extm/extm_file",
        "dex2oat_threads": {
            "def": 1,
            "4"  : 3,
            "8"  : 6,
            "10" : 8
        },
        "boot_dex2oat_threads": {
            "def": 1,
            "4"  : 3,
            "8"  : 6,
            "10" : 8
        },
        "bg_dex2oat_threads": {
            "def":1,"1":1,"2":2,"3":3,"4":4,"5":5,"6":6,"7":7,"8":8,"9":9,"10":10
        },
        "reclaim_on_start": {
            "def":0
        },
        "swappiness_on_start": {
            "def":100
        },
        "reclaim_on_launcher": {
            "def":0,"1":120,"2":120
        },
        "swappiness_on_launcher": {
            "def":100,"2":200
        }
    },
    "dex2oat": [
        {
            "product_name": ["dandelion", "angelica", "cattail", "angelican", "willow", "ginkgo", "cannon", "cannong",
                             "mojito", "sunny", "rainbow", "rosemary", "secret", "maltose", "biloba", "XIG02", "chopin",
                             "camellia", "camellian", "selene", "atom", "bomb", "spes", "spesn", "lime", "citrus"],
            "dex2oat_threads": {
                "def": 4
            },
            "boot_dex2oat_threads": {
                "def": 6
            },
            "bg_dex2oat_threads": {
                "def": 4
            }
        },
        {
            "product_name": ["merlin", "merlinin", "merlinnfc", "lancelot", "shiva", "pine", "olive", "olivelite", "olivewood", "onc", "lavender", "violet", "laurus", "camellia", "camellian", "sapphire", "sapphiren"],
            "dex2oat_threads": {
                "def": 4
            },
            "boot_dex2oat_threads": {
                "def": 6
            },
            "bg_dex2oat_threads": {
                "def": 4
            }
        },
        {
            "product_name": ["bixi"],
            "dex2oat_threads": {
                "def": 6
            },
            "boot_dex2oat_threads": {
                "def": 6
            },
            "bg_dex2oat_threads": {
                "def": 4
            }
        }
    ]
}
```
system_ext\etc\perfinit_bdsize_zram.conf
```
{
    "auto_zram": [
        {
            "auto_zram_flash": [16],
            "auto_zram_ram_2G": {
                "def_bdsize":0.5, "0.5":1024
            }
        },
        {
            "auto_zram_flash": [32],
            "auto_zram_ram_2G":{
                "def_bdsize":0.5, "0.5":1024, "1.0":1024, "2.0":2048
            },
            "auto_zram_ram_3G":{
                "def_bdsize":0.5, "0.5":1536, "1.0":1536, "2.0":2048
            }
        },
        {
            "auto_zram_flash": [64],
            "auto_zram_ram_2G":{
                "def_bdsize":1.0, "0.5":1024, "1.0":1024, "2.0":2048
            },
            "auto_zram_ram_3G":{
                "def_bdsize":1.0, "1.0":2048, "2.0":2048, "3.0":3072
            },
            "auto_zram_ram_4G":{
                "def_bdsize":1.0, "1.0":3072, "2.0":3072, "4.0":4096
            },
            "auto_zram_ram_6G":{
                "def_bdsize":2.0, "1.0":4096, "2.0":4096, "4.0":4096
            }
        },
        {
            "auto_zram_flash": [128],
            "auto_zram_ram_4G":{
                "def_bdsize":2.0, "1.0":3072, "2.0":3072, "4.0":4096
            },
            "auto_zram_ram_6G":{
                "def_bdsize":2.0, "2.0":4096, "4.0":4096, "6.0":6144
            },
            "auto_zram_ram_8G":{
                "def_bdsize":4.0, "4.0":6144, "6.0":6144, "8.0":8192
            },
            "auto_zram_ram_12G":{
                "def_bdsize":4.0, "4.0":8192, "6.0":8192, "8.0":8192, "12.0":12288
            },
            "auto_zram_ram_16G":{
                "def_bdsize":6.0, "6.0":12288, "8.0":12288, "12.0":12288, "16.0":16384
            },
            "auto_zram_ram_24G":{
                "def_bdsize":6.0, "6.0":16384, "8.0":16384, "12.0":16384, "16.0":16384
            }
        },
        {
            "auto_zram_flash": [256],
            "auto_zram_ram_4G":{
                "def_bdsize":2.0, "1.0":3072, "2.0":3072, "4.0":4096
            },
            "auto_zram_ram_6G":{
                "def_bdsize":4.0, "2.0":4096, "4.0":4096, "6.0":6144
            },
            "auto_zram_ram_8G":{
                "def_bdsize":4.0, "4.0":6144, "6.0":6144, "8.0":8192
            },
            "auto_zram_ram_12G":{
                "def_bdsize":4.0, "4.0":8192, "6.0":8192, "8.0":8192, "12.0":12288
            },
            "auto_zram_ram_16G":{
                "def_bdsize":6.0, "6.0":12288, "8.0":12288, "12.0":12288, "16.0":16384
            },
            "auto_zram_ram_24G":{
                "def_bdsize":6.0, "6.0":16384, "8.0":16384, "12.0":16384, "16.0":16384
            }
        },
        {
            "auto_zram_flash": [512, 1024],
            "auto_zram_ram_4G":{
                "def_bdsize":2.0, "1.0":3072, "2.0":3072, "4.0":4096
            },
            "auto_zram_ram_6G":{
                "def_bdsize":4.0, "2.0":4096, "4.0":4096, "6.0":6144
            },
            "auto_zram_ram_8G":{
                "def_bdsize":6.0, "4.0":6144, "6.0":6144, "8.0":8192
            },
            "auto_zram_ram_12G":{
                "def_bdsize":6.0, "4.0":8192, "6.0":8192, "8.0":8192, "12.0":12288
            },
            "auto_zram_ram_16G":{
                "def_bdsize":6.0, "6.0":12288, "8.0":12288, "12.0":12288, "16.0":16384
            },
            "auto_zram_ram_24G":{
                "def_bdsize":6.0, "6.0":16384, "8.0":16384, "12.0":16384, "16.0":16384
            }
        }
    ],
    "zram":[
        {
            "product_name": ["evergo", "evergreen", "opal", "selene", "spes", "fog", "wind", "rain", "spesn", "earth", "aether", "eos", "rock", "stone", "camellian", "camellia", "tapas", "tapaz", "sea", "ocean","light","sunstone"],
            "zram_size": {
                "def":512,"2":1024,"3":3072,"4":4096,"6":4096,"8":6144,"10":6144,"12":8192,"16":14336, "18":14336, "20":15360, "24":15360, "32":16384
            }
        },
        {
            "product_name": ["yunluo"],
            "zram_size": {
                "3":3072, "4":4096
            }
        },
        {
            "product_name": ["air", "atmos", "gust", "gale", "xun", "lake", "pond"],
            "zram_size": {
                "4":4096
            }
        },
        {
            "product_name": ["miro", "shennong", "haotian","xuanyuan", "popsicle", "pandora", "dali", "jinghu", "nezha", "myron", "annibale","klimt","bixi","ruyi"],
            "zram_size": {
                "12":12288, "16":16384
            }
        },
        {
            "product_name": ["rothko","houji", "dada", "pudding", "violin"],
            "zram_size": {
                "8":8192,
                "12":12288,
                "16":16384
            }
        },
        {
            "product_name": ["manet", "vermeer", "dijun"],
            "zram_size": {
                "16":16384
            }
         },
        {
            "product_name": ["muyu"],
            "zram_size": {
                "8":8192, "12":12288, "16":16384
            }
        },
        {
            "product_name": ["piano", "yupei"],
            "zram_size": {
                "8":8192, "12":12288
            }
        },
        {
            "product_name": ["nabu", "elish", "enuma", "dagu", "pipa", "liuqin", "yudi"],
            "zram_size": {
                "6":6144, "8":8192, "12":12288, "16":16384
            }
        }
    ]
}
```
完美横屏附加功能修改  
system_ext\framework\miui-embedding-window.jar  
system_ext\framework\miui-services.jar  
system_ext\priv-app\MiuiSystemUI\MiuiSystemUI.apk  
system_ext\priv-app\Settings\Settings.apk  
已经完成通过github action实现自动化修改https://github.com/ymdzq/mipad_module  
## vendor分区修改，整体上用5pro的，但要注意以下部分
从6的OS2.0.11.0提取以下文件替换，修复selinux权限  
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
从6的OS2.0.10.0提取以下文件替换，修复蓝牙耳机播放视频音画不同步bug，感谢云彩之枫  
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
