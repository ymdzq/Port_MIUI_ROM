# 小米平板5 PRO 移植小米平板6 11英寸 HyperOS记录
资源来源于网络，仅供交流学习，不得用做任何商业用途，不提供任何技术支持，请在下载后24小时内删除  
基于ELISH_OS1.0.2.0，移植文件来源于PIPA_OS2.0.4.0  
这里推荐一下隔壁大佬的[HyperOS 移植项目](https://github.com/toraidl/hyperos_port)，有很多移植澎湃的经验、修改启发  
本文仅记录一下修改内容，具体修改行以及内容以实际文件对比结果为准  

由于修改了系统文件，所以avb验证肯定是要关的，而想保证各种app兼容性，所以我选择保持selinux enforce，即不集成pc版wps  
如果不集成，就不需要改vendor分区，随便在product分区里精简一点东西，就可以确保刷进机器那8.5G的super分区。  
## mi_ext分区修改，在5Pro的基础上，覆盖6的所有文件
build.prop修改机型代号，这里这个代号是miui ota更新服务器用来识别推送更新用的，你都刷第三方rom了这个就不重要了，除非你能用到那个服务器推送更新  
把这个东西改掉的好处就是可以屏蔽更新，不会收到移植的那个机型的更新，导致用户误升级变砖  
修改版本号为UKYCNXM  
mi_ext\etc\build.prop
```
ro.product.mod_device=elish
ro.mi.os.version.incremental=OS2.0.4.0.UKYCNXM
```

这里提一句，比较新的机型的剃刀计划版本也比较新，支持卸载平板/手机管家，而版本不兼容就导致了部分机型移植完桌面没有平板/手机管家的图标，这里把有相关影响的内容列出来，这个部分提到的文件需要从6Max(yudi)的rom中提取  
mi_ext\etc\build.prop里面有一行`ro.miui.support.system.app.uninstall.v2=true`  
mi_ext\product\etc\permissions\platform-miui-uninstall.xml  
mi_ext\product\framework\miui-uninstall-empty.jar  
mi_ext\product\overlay\signed_PLATFORM_cf766d1e91_app_sec_overlay-release-unsigned.apk  

product\data-app\MIUISecurityManager\MIUISecurityManager.apk  
## odm分区，用5pro的，不用改
这个分区是跟vendor分区配套的，目前无需修改  
## product分区修改，整体上照搬6，但要注意以下部分
pc版wps相关文件  
访问linux容器的rdp后端MSLgRdp，PC 框架？和交互操作的前端WpsLauncher  
不集成pc版wps可以直接删除  
product\app\MSLgRdp   
product\data-app\WpsLauncher  

product\app  
保留5pro小爱翻译 AiAsstVision  
（a13澎湃内置的版本号是4.6.0，可能需要使用模块解锁实时字幕功能）  
删除6人脸识别解锁 Biometric  
保留5pro人脸识别解锁 MiuiBiometric3373  
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

设备功能配置文件，本来正常代号要用elish稳定使用的话，删除pipa.xml，照搬elish.xml就好了，  
但是如果你要全局改机器代号的话，这里配置文件也要改名成pipa.xml，  
所以我是建议干脆把elish.xml复制两份一个叫elish.xml一个叫pipa.xml，都放进去，这样用哪个代号也不要紧  
product\etc\device_features\elish.xml  
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
    <integer-array name="fpsList">
        <item>120</item>
        <item>60</item>
    </integer-array>
```
修改屏幕亮度配置文件  
product\etc\displayconfig\display_id_4630947141052476290.xml  
product\etc\displayconfig\display_id_4630947200012256898.xml  

5pro屏幕的xml文件为：  
product\etc\displayconfig\display_id_19260527152667265.xml  
product\etc\displayconfig\display_id_4630946481717202305.xml  
product\etc\displayconfig\display_id_4630946545580055169.xml  
这三个文件的内容是完全一样的，所以我选择删掉display_id_4630947141052476290.xml和display_id_4630947200012256898.xml，并且保留这三个xml文件，屏幕亮度调节就正常了  
这里需要注意Overlay里面的AospFrameworkResOverlay.apk要换成5Pro的，否则会遇到自动亮度导致系统软重启的问题  
product\overlay\AospFrameworkResOverlay.apk  
需要apkeditor-1.4.2反编译修改，  
替换所有default_wallpaper.jpg，  
修改bools.xml  
```
删除
  <bool name="config_supportsMicToggle">true</bool>

添加
  <bool name="config_dozeAlwaysOnEnabled">false</bool>
```
修改strings.xml  
```
添加
  <string name="config_defaultAttentionService">com.xiaomi.aon/com.xiaomi.aon.AonAttentionService</string>
```
修改public.xml，id我不确定，随便写的不重复新id  
```
删除
  <public id="0x7f020005" type="bool" name="config_supportsMicToggle" />
添加
  <public id="0x7f020007" type="bool" name="config_dozeAlwaysOnEnabled" />
  <public id="0x7f070003" type="string" name="config_defaultAttentionService" />
```
build.prop修改机型代号、版本指纹，设置默认屏幕密度，关闭内存扩展  
product\etc\build.prop
```
ro.product.product.name=elish
ro.product.build.fingerprint=Xiaomi/elish/miproduct:14/UKQ1.240624.001/OS2.0.4.0.UKYCNXM:user/release-keys
ro.product.build.version.incremental=OS2.0.4.0.UKYCNXM

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

#作用未知
ro.config.miui_compat_enable=true
ro.config.miui_appcompat_enable=true
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

默认开启通信共享  
product\media\theme\default\framework-miui-res  

保留5pro本身开机动画（分辨率匹配屏幕）  
product\media\bootanimation.zip  

overlay保留5pro本身设备的apk  
DevicesAndroidOverlay主要影响圆角弧率、状态栏高度，aod服务（lcd没有）  
product\overlay\DevicesAndroidOverlay.apk  
DevicesOverlay主要影响导航栏（小白条）布局以及圆角，充电动画  
product\overlay\DevicesOverlay.apk  
需要apkeditor-1.4.2反编译修改，  
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
product\overlay\MiuiFrameworkResOverlay.apk  
MiuiBiometricResOverlay人脸识别资源文件空包  
product\overlay\MiuiBiometricResOverlay.apk  

删除6相机，否则会提示机型不匹配无法使用然后退出，  
目前澎湃只能用5.0以上版本的相机，老apk无法使用，同样会提示机型不匹配无法使用然后退出，  
直接抄闪电flasshh的5.1通用相机，其他选择只能用谷歌相机、骁龙相机这种第三方相机  
product\priv-app\MiuiCamera  
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
## system分区不修改，直接照搬6
可选修改  
签名破解，要修改系统app，就需要修改services.jar文件，我这里使用的SYT_ROM工具提供的插件自动修改  
system\system\framework\services.jar  
build.prop修改机型代号、版本指纹  
system\system\system_dlkm\etc\build.prop
```
ro.system_dlkm.build.fingerprint=Android/missi_pad_cn/missi:14/UKQ1.240624.001/OS2.0.4.0.UKYCNXM:user/release-keys
ro.system_dlkm.build.version.incremental=OS2.0.4.0.UKYCNXM
```
system\system\build.prop
```
ro.system.build.fingerprint=Android/missi_pad_cn/missi:14/UKQ1.240624.001/OS2.0.4.0.UKYCNXM:user/release-keys
ro.system.build.version.incremental=OS2.0.4.0.UKYCNXM
ro.build.version.incremental=OS2.0.4.0.UKYCNXM

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
ro.system_ext.build.fingerprint=Android/missi_pad_cn/missi:14/UKQ1.240624.001/OS2.0.4.0.UKYCNXM:user/release-keys
ro.system_ext.build.version.incremental=OS2.0.4.0.UKYCNXM
```
zram配置文件，提取自6Pro OS2.0.4.0，添加平板5系列、6系列ram/zram容量1：1  
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
            "product_name": ["evergo", "evergreen", "opal", "selene", "spes", "fog", "wind", "rain", "spesn", "earth", "aether", "eos", "rock", "stone", "camellian", "camellia", "tapas", "topaz", "sea", "ocean","light","sunstone"],
            "zram_size": {
                "def":512,"2":1024,"3":3072,"4":4096,"6":4096,"8":6144,"10":6144,"12":8192,"16":14336, "18":14336, "20":15360, "24":15360, "32":16384
            }
        },
        {
            "product_name": ["yunluo", "sky", "river"],
            "zram_size": {
                "3":3072, "4":4096
            }
        },
        {
            "product_name": ["air", "atmos", "gust", "gale", "xun"],
            "zram_size": {
                "4":4096
            }
        },
        {
            "product_name": ["fire", "heat"],
            "zram_size": {
                "4":4096
            }
        },
        {
            "product_name": ["houji", "dada"],
            "zram_size": {
                "8":8192, "12":12288, "16":16384
            }
        },
        {
            "product_name": ["miro", "shennong", "haotian","zorn", "xuanyuan"],
            "zram_size": {
                "12":12288, "16":16384
            }
        },
        {
            "product_name": ["garnet"],
            "zram_size": {
                "8":8192
            }
        },
        {
            "product_name": ["manet", "vermeer"],
            "zram_size": {
                "16":16384
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
## vendor分区修改，整体上用5pro的，但要注意以下部分
从6的OS2.0.4.0提取以下文件替换，修复蓝牙耳机播放视频音画不同步bug，感谢云彩之枫  
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
## 重新打包mi_ext、odm、system、system_ext、vendor、product分区
先用make_ext4fs或者e2fsdroid+mke2fs打包为raw image，  
然后用lpmake打包成super img  
vab机器一般是线刷用fastboot刷进super分区，卡刷是在recovery里用卡刷脚本写入到super分区，  
常见的情况也有使用zstd工具把super压缩成zst格式（打包zst需要raw格式的super.img），在线刷、卡刷的时候再解压，这种用压缩解压的时间来节省刷机包占用空间大小的做法，  
这种的情况就需要专门的脚本和工具了  
由于无wps版由于不需要修改odm、vendor分区，所以理论上其实你可以直接用fastbootd模式刷入mi_ext、system、system_ext、product分区  
dsu包的做法就是直接把mi_ext、system、system_ext、product分区的raw image文件打包成一个zip或者gz文件即可  
解包打包偷懒就找个安卓工具箱，SYT、米欧、dna、多幸运之类的，直接一键打包  
## 关闭avb验证  
可选，修改fstab.qcom去除avb代码  
vendor\etc\fstab.qcom  
把system那一行的flags从`,avb_keys=`开始把后面的内容全删除，所有`,avb=vbmeta_system`删除，所有`,avb=vbmeta`删除，  

可选，vendor_boot修改header在最后增加设置宽容的代码，如果要打包pc版wps就设置一下宽容，如果不需要改vendor就算了  
`androidboot.selinux=permissive`

修改vbmeta.img、vbmeta_system.img，关闭avb验证，这玩意得用十六进制编辑器或者打包工具修改，  
我看米欧是修改的十六进制0000007B这个地址00改成02，这个改法跟下面两条命令是同样的效果  
另一个办法，用户刷入vbmeta、vbmeta_system时使用命令关闭avb验证或者在twrp中直接用选项关闭
```bash
fastboot --disable-verity --disable-verification flash vbmeta vbmeta.img
fastboot --disable-verity --disable-verification flash vbmeta_system vbmeta_system.img
```
