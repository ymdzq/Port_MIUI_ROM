# 小米平板5 PRO 移植小米平板6 MAX MIUI 14记录
资源来源于网络，仅供交流学习，不得用做任何商业用途，不提供任何技术支持，请在下载后24小时内删除  
基于ELISH_OS1.0.2.0，移植文件来源于miui_YUDI_V14.0.6.0  
本文仅记录一下修改内容，具体修改行以及内容以实际文件对比结果为准  

由于修改了系统文件，所以avb验证肯定是要关的。  
而想保证各种app兼容性，所以建议保持selinux enforce，要么保持5pro原版sepolicy放弃pc引擎，要么移植6的sepolicy。  
pc版wps这一套东西2.44GB，该版本小米是都放在vendor分区，  
如果要集成6max这个老版本的pc版wps，就需要极限精简系统分区内文件，或者扩容机器的super分区才能刷得进去，或者换一个支持erofs文件系统的第三方内核，使用erofs文件系统压缩打包系统img（https://www.coolapk.com/feed/66511232?s=N2IxM2UwMmQxYWRkNjJmZzY4YWVmNjM5ega1551 或者https://www.coolapk.com/feed/66205342?s=MTkxNDBhNDExYWRkNjJmZzY4YWVmODIxega1551 ）  。  
澎湃新版本pc版wps由于运行时有版本号验证，所以即使把放在odm分区的linux容器反向移植到miui上能正常启动也没法用，  
如果不集成，就不需要改vendor分区，随便在product分区里精简一点东西，就可以确保刷进机器那8.5G的super分区。  
## mi_ext分区修改，整体上照搬6max，但要注意以下部分
build.prop修改机型代号，这里这个代号是miui ota更新服务器用来识别推送更新用的，你都刷第三方rom了这个就不重要了，除非你能用到那个服务器推送更新  
mi_ext\etc\build.prop
```
ro.product.mod_device=elish
```
小米debug用的官方root相关文件（澎湃1.0.2.0有升级，所以使用5Pro的文件替换）  
mi_ext\root\bin\remount  
## odm分区
替换为小米平板6标准版的预编译sepolicy文件（主要用途是支持PC引擎）  
odm\etc\selinux\precompiled_sepolicy  

可选修改  
添加修改过的默认云控文件  
odm\etc\default_cloud.json  
## product分区修改，整体上照搬6max，但要注意以下部分
pc版wps相关文件  
访问linux容器的rdp后端MSLgRdp，PC 框架？和交互操作的前端WpsLauncher  
不集成pc版wps可以直接删除，集成则保留  
product\app\MSLgRdp   
product\data-app\WpsLauncher  

product\app  
提取7Pro小爱翻译 AiAsstVision  
（如果需要使用离线翻译模型，你就去装破解pipa的小爱翻译，直接装原版整个翻译app无法打开，但破解后支持elish和enuma使用中英文离线翻译。已测试lisa、ruan、dizi、muyu使用的小爱翻译是在线翻译模型，使用模块解锁实时字幕功能可以在线翻译多国语言，不解锁实时字幕elish和enuma也可以使用其他翻译功能）  
删除无法使用的6max人脸识别解锁 MiuiBiometric  
保留5pro人脸识别解锁 MiuiBiometric3373  
删除6max的手写笔和键盘设置 MiuiInputSettings_M80 据说会导致有线鼠标操作失灵  
保留5pro的手写笔和键盘设置 MiuiInputSettings  
overlay同上需要替换 product\overlay\MiuiInputSettingsOverlay.apk

按需精简  
快应用服务引擎  
product\app\HybridPlatform  
智能服务  
product\app\MSA  

data-app可卸载的预装app，其中不少app都是可以在应用商店里重新安装的，  
product\data-app\  
因为平板5pro默认的super分区只有8.5G，而且重新打包必须预留更多空间，所以可以精简这里，把super精简到7.4G以下，越小越好  
不过如果你要塞pc版wps，推荐使用erofs文件系统打包系统分区，但是如果空间占用太大可能遇到rec刷不进去  
我个人是觉得没必要搞极限精简，很多常用自带功能用户到时候又要想办法装回来，挺烦人的  
小米创作  
product\data-app\Creation  
小米商城  
product\data-app\MiShopPad  
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
product\data-app\MIUIVipAccountPad  
米家  
product\data-app\SmartHome  

设备功能配置文件，本来正常代号要用elish稳定使用的话，删除yudi.xml，照搬elish.xml就好了，  
但是如果你要全局改机器代号的话，这里配置文件也要改名成yudi.xml，  
所以我是建议干脆把elish.xml复制两份一个叫elish.xml一个叫yudi.xml，都放进去，这样用哪个代号也不要紧  
product\etc\device_features\elish.xml  
product\etc\device_features\yudi.xml  
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
product\etc\displayconfig\display_id_4630946932993367170.xml  

5pro屏幕的xml文件为：  
product\etc\displayconfig\display_id_19260527152667265.xml  
product\etc\displayconfig\display_id_4630946481717202305.xml  
product\etc\displayconfig\display_id_4630946545580055169.xml  
这三个文件的内容是完全一样的，所以我选择再复制一个替换display_id_4630946932993367170.xml，保留这四个xml文件，屏幕亮度调节就正常了  
这里需要注意Overlay里面的AospFrameworkResOverlay.apk要换成5Pro的（必须用miui14的版本），否则会遇到自动亮度导致系统软重启的问题  
product\overlay\AospFrameworkResOverlay.apk  
build.prop直接用5Pro澎湃1.0.2.0修改  
product\etc\build.prop
```
ro.product.build.date=Fri Sep 15 11:54:50 UTC 2023
ro.product.build.date.utc=1694778890
ro.product.build.fingerprint=Xiaomi/miproduct_elish_cn/missi:13/TKQ1.221114.001/V14.0.6.0.TKYCNXM:user/release-keys
ro.product.build.version.incremental=V14.0.6.0.TKYCNXM

ro.miui.ui.version.code=14
ro.miui.ui.version.name=V140

#删除
ro.miui.build.region=cn
persist.sys.disable_bganimate=false
ro.miui.product.home=com.miui.home

#默认关闭内存扩展
persist.miui.extm.enable=0

#开启高级材质选项
persist.sys.background_blur_supported=true
persist.sys.advanced_visual_release=2
persist.sys.mi_shadow_supported=true

#6max多了的两行玄学优化，平滑圆角
persist.sys.support_view_smoothcorner=true
persist.sys.support_window_smoothcorner=true

#开启布局优化
persist.miui.auto_ui_enable=true

#游戏加载加速？
debug.game.video.speed=true
debug.game.video.support=true

#HDR修复？
persist.sys.support_ultra_hdr=true

#app调整游戏显示布局功能
ro.config.miui_compat_enable=true
ro.config.miui_appcompat_enable=true

#静置时维持刷新率时长
ro.surface_flinger.use_content_detection_for_refresh_rate=true
ro.surface_flinger.set_idle_timer_ms=2147483647
ro.surface_flinger.set_touch_timer_ms=2147483647
ro.surface_flinger.set_display_power_timer_ms=2147483647

#作用未知
persist.sys.launch_response_optimization.enable=true
ro.surface_flinger.game_default_frame_rate_override=120
ro.miui.shell_anim_enable_fcb=true
```
内置完美横屏计划  
product\etc\autoui_list.xml  
product\etc\embedded_rules_list.xml  
product\etc\fixed_orientation_list.xml  

内置完美图标计划  
product\media\theme\default\dynamicicons  
product\media\theme\default\icons  
product\media\theme\default\miui_mod_icons\  

保留5pro本身开机动画（分辨率匹配屏幕）  
product\media\bootanimation.zip  

overlay保留5pro本身设备的apk（必须用miui14的版本）  
DevicesAndroidOverlay主要影响圆角弧率、状态栏高度，aod服务（lcd没有）  
product\overlay\DevicesAndroidOverlay.apk  
DevicesOverlay主要影响导航栏（小白条）布局以及圆角，充电动画  
product\overlay\DevicesOverlay.apk  
MiuiFrameworkResOverlay主要影响屏幕hbm背光、hbm亮度曲线、以及一些网络制式的属性  
product\overlay\MiuiFrameworkResOverlay.apk  
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
内置启用小米工具AI功能叠加层文件  
product\overlay\MiuiNotesOverlay.apk  
product\overlay\MiuiSoundrecorderOverlay.apk  
product\overlay\MiuiThememanagerOverlay.apk  

删除6max相机，否则会提示机型不匹配无法使用然后退出，  
直接小米11青春版（lisa）的5.1通用相机，其他选择也可以使用sevtinge修改相机4.7.230127.0版本，没有机型限制而且解锁更多功能  
product\priv-app\MiuiCamera  
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
      <permission name="android.permission.READ_CLIPBOARD_IN_BACKGROUND" />
   </privapp-permissions>
```
保留5Pro原版音质音效  
product\app\MiSound  
替换5Pro澎湃1.0.2.0小爱同学，当前版本超级小爱不支持miui，等待应用商店后续推送新版低端机通道更新  
product\app\VoiceAssistAndroidT\VoiceAssistAndroidT.apk  
更新预置webview浏览器内核  
product\app\TrichromeLibrary64\TrichromeLibrary64.apk  
product\app\WebViewGoogle64\WebViewGoogle64.apk  
OS3新版PC布局浏览器  
product\priv-app\MIUIBrowserPad\MIUIBrowserPad.apk  
删除6max桌面  
product\priv-app\MiuiHomePadT  
替换5Pro澎湃1.0.2.0桌面，支持工作台模式，更流畅  
product\priv-app\MiuiHome\MiuiHome.apk  
替换修改版平板管家（不要用太高版本，OS3的会导致小窗贴边闪退）  
product\priv-app\MIUISecurityCenterPad  
## system分区不修改，直接照搬6max也行
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
ro.system_dlkm.build.fingerprint=qti/missi_pad_cn/missi:13/TKQ1.221114.001/V14.0.6.0.TKYCNXM:user/release-keys
ro.system_dlkm.build.version.incremental=V14.0.6.0.TKYCNXM
```
system\system\build.prop
```
ro.system.build.fingerprint=qti/missi_pad_cn/missi:13/TKQ1.221114.001/V14.0.6.0.TKYCNXM:user/release-keys
ro.system.build.version.incremental=V14.0.6.0.TKYCNXM
ro.build.version.incremental=V14.0.6.0.TKYCNXM
ro.product.mod_device=elish

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
zram配置文件，提取自7Pro OS3.0.0.16的system_ext分区，添加平板5系列、6系列ram/zram容量1：1（实际是否有效果未测试）  
system\system\etc\perfinit.conf
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
system\system\etc\perfinit_bdsize_zram.conf
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
## system_ext分区不修改，直接照搬6max
可选修改  
build.prop修改机型代号、版本指纹  
system_ext\etc\build.prop
```
ro.system_ext.build.fingerprint=qti/missi_pad_cn/missi:13/TKQ1.221114.001/V14.0.6.0.TKYCNXM:user/release-keys
ro.system_ext.build.version.incremental=V14.0.6.0.TKYCNXM
#完美横屏附加功能主动适配
ro.config.sothx_project_treble_support_magic_window_fix=true
```
完美横屏附加功能修改(开启平行世界左右比例调节)  
system_ext\framework\miui-embedding-window.jar  
系统设置解锁平板专区、工作台模式开关  
system_ext\priv-app\Settings\Settings.apk  
## vendor分区修改，整体上用5pro的，但要注意以下部分
从小米平板6标准版的OS2.0.13.0提取以下文件替换，修复mslg服务的selinux权限  
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

vendor/build.prop加入代码  
```
#玄学优化代码
# fix the drop frame issus
ro.surface_flinger.enable_frame_rate_override=false
debug.sf.auto_latch_unsignaled=0
vendor.display.enable_display_extensions=1
```
## vendor分区可选修改，集成6max老版本linux容器
如果要集成pc版wps则注意以下部分  
6max新增pc版wps相关文件，只要对比6pro（liuqin）的整个vendor分区，看孤立文件，一眼就能看出这些文件跟pc版wps有关，  
其中mslgoptimg、mslgusrimg两个1G以上大文件，是导致super分区需要扩容的原因，  
所以如果能以某种方法比如这里留一个到userdata的链接，然后把实际文件丢进userdata，  
或者直接改sh脚本把位置就写到其他地方，就不需要占用vendor、super分区了  
另外说一句，由于有人测试了，这东西类原生也可以用，所以理论上用这个东西不一定非要移植6max的rom，如果你能把这些相关文件还有上面product分区里面的两个app放进其他安卓系统的对应位置，其他安卓系统搞不好也能运行  
```
/vendor/bin/hw/mslgservice
/vendor/bin/losetup.sh
/vendor/bin/start-rootfs.sh
/vendor/bin/tar-rootfs.sh

/vendor/etc/assets/md5.txt
/vendor/etc/assets/mslgoptimg
/vendor/etc/assets/mslgusrimg
/vendor/etc/assets/rootfs-23.09.08.tgz

/vendor/etc/init/mslgservice.rc

/vendor/lib/libext2_uuid.so

/vendor/lib64/libext2_uuid.so
/vendor/lib64/vendor.xiaomi.mslg.keeper@1.0.so
```
vendor/build.prop加入代码  
```
ro.vendor.mslg.rootfs.version=rootfs-23.09.08.tgz
sys.mslg.available=1
```
由于sepoolicy文件我们是移植的澎湃版本mslg服务在odm分区，但是这套msgl服务是老版本在vendor分区，路径上可能对不上，未测试，需要自行参悟修改mslg服务的selinux权限
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
