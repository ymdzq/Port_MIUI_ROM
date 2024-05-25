# 小米平板5 PRO 移植小米平板6S Pro 12.4英寸 HyperOS记录
资源来源于网络，仅供交流学习，不得用做任何商业用途，不提供任何技术支持，请在下载后24小时内删除  
基于ELISH_OS1.0.2.0，移植文件来源于SHENG_OS1.0.7.0  
这里推荐一下隔壁大佬的[HyperOS 移植项目](https://github.com/toraidl/hyperos_port)，有很多移植澎湃的经验、修改启发  
本文仅记录一下修改内容，具体修改行以及内容以实际文件对比结果为准  

由于修改了系统文件，所以avb验证肯定是要关的，而想保证各种app兼容性，所以我选择保持selinux enforce，即不集成pc版wps  
如果不集成，就不需要改vendor分区，随便在product分区里精简一点东西，就可以确保刷进机器那8.5G的super分区。  
## mi_ext分区修改，整体上照搬6s Pro，但要注意以下部分
build.prop修改机型代号，这里这个代号是miui ota更新服务器用来识别推送更新用的，你都刷第三方rom了这个就不重要了，除非你能用到那个服务器推送更新  
把这个东西改掉的好处就是可以屏蔽更新，不会收到移植的那个机型的更新，导致用户误升级变砖  
修改版本号为UKYCNXM  
mi_ext\etc\build.prop
```
ro.product.mod_device=elish
ro.mi.os.version.incremental=OS1.0.7.0.UKYCNXM
```
按需精简  
预装画世界Pro，用来绘画的app?  
mi_ext\product\data-app\HsjPro\HsjPro.apk  

这里提一句，比较新的机型的剃刀计划版本也比较新，支持卸载平板/手机管家，而版本不兼容就导致了部分机型移植完桌面没有平板/手机管家的图标，这里把有相关影响的内容列出来  
mi_ext\etc\build.prop里面有一行`ro.miui.support.system.app.uninstall.v2=true`  
mi_ext\product\etc\permissions\platform-miui-uninstall.xml  
mi_ext\product\framework\miui-uninstall-empty.jar  
mi_ext\product\overlay\signed_PLATFORM_cf766d1e91_app_sec_overlay-release-unsigned.apk  

product\data-app\MIUISecurityManager\MIUISecurityManager.apk  
## odm分区，用5pro的，不用改
这个分区是跟vendor分区配套的，目前无需修改  
## product分区修改，整体上照搬6s Pro，但要注意以下部分
pc版wps相关文件  
访问linux容器的rdp后端MSLgRdp，PC 框架？和交互操作的前端WpsLauncher  
不集成pc版wps可以直接删除  
product\app\MSLgRdp   
product\data-app\WpsLauncher  

product\app  
保留5pro小爱翻译 AiAsstVision  
（a13澎湃内置的版本号是4.6.0，可能需要使用模块解锁实时字幕功能）  
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
product\data-app\com.iflytek.inputmethod.miui  
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
product\data-app\MIUIVipAccountPad  
小米有品  
product\data-app\MIUIYoupin  
小米汽车拓展屏  
product\data-app\Padapp  
米家  
product\data-app\SmartHome  

设备功能配置文件，本来正常代号要用elish稳定使用的话，删除sheng.xml，照搬elish.xml就好了，  
但是如果你要全局改机器代号的话，这里配置文件也要改名成sheng.xml，  
所以我是建议干脆把elish.xml复制两份一个叫elish.xml一个叫sheng.xml，都放进去，这样用哪个代号也不要紧  
product\etc\device_features\elish.xml  
product\etc\device_features\sheng.xml  
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
```
修改屏幕亮度配置文件  
product\etc\displayconfig\display_id_4630947038039379843.xml  
目前6s Pro只有一家屏幕供应商，由于没有测试是否还有之前版本出现的`*** FATAL EXCEPTION IN SYSTEM PROCESS: android.display`报错无法开机的问题  

5pro屏幕的xml文件为：  
product\etc\displayconfig\display_id_19260527152667265.xml  
product\etc\displayconfig\display_id_4630946481717202305.xml  
product\etc\displayconfig\display_id_4630946545580055169.xml  
这三个文件的内容是完全一样的，所以我选择删掉display_id_4630947038039379843.xml，并且保留这三个xml文件，屏幕亮度调节就正常了  
这里需要注意Overlay里面的AospFrameworkResOverlay.apk要换成5Pro的，否则会遇到自动亮度导致系统软重启的问题  
product\overlay\AospFrameworkResOverlay.apk  
照搬官方澎湃，不用修改，好耶  

build.prop修改机型代号、版本指纹，设置默认屏幕密度，关闭内存扩展  
product\etc\build.prop
```
ro.product.product.name=elish
ro.product.build.fingerprint=Xiaomi/elish/miproduct:14/UKQ1.231003.002/V816.0.7.0.UKYCNXM:user/release-keys
ro.product.build.version.incremental=V816.0.7.0.UKYCNXM

persist.miui.density_v2=360
ro.sf.lcd_density=360

#默认关闭内存扩展
persist.miui.extm.enable=0

#修改ro.miui.cust_erofs改成
ro.miui.cust_erofs=0

#修改ro.millet.netlink改成
ro.millet.netlink=29
 
#删除ro.config.miui_screen_ratio这一行
#ro.config.miui_screen_ratio=0.5

#开启高级材质选项
persist.sys.background_blur_supported=true
persist.sys.background_blur_version=2
persist.sys.mi_shadow_supported=true

#6max多了的两行玄学优化，平滑圆角
persist.sys.support_view_smoothcorner=true
persist.sys.support_window_smoothcorner=true

#游戏加载加速？
debug.game.video.speed=true
debug.game.video.support=true

#加回5Pro本身的玄学优化，性能调度
persist.sys.miui.sf_cores=4-7
persist.vendor.display.miui.composer_boost=4-7
persist.sys.minfree_def=73728,92160,110592,154832,482560,579072
persist.sys.minfree_6g=73728,92160,110592,258048,663552,903168
persist.sys.minfree_8g=73728,92160,110592,387072,1105920,1451520

#作用未知
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
MiuiFrameworkResOverlay主要影响屏幕hbm背光、hbm亮度曲线、以及一些网络制式的属性  
product\overlay\MiuiFrameworkResOverlay.apk  
MiuiBiometricResOverlay人脸识别资源文件空包  
product\overlay\MiuiBiometricResOverlay.apk  

删除6s Pro相机，否则会提示机型不匹配无法使用然后退出，  
目前澎湃只能用5.0以上版本的相机，老apk无法使用，同样会提示机型不匹配无法使用然后退出，  
直接抄闪电flasshh的5.1通用相机，其他选择只能用谷歌相机、骁龙相机这种第三方相机  
product\priv-app\MiuiCamera  
并且删除两个oat文件  
## 可选product分区修改，补全小米平板缺失的工具app
悬浮球  
product\app\MIUITouchAssistant  
小米锁屏画报  
product\data-app\MIGalleryLockscreen-MIUI15\MIGalleryLockscreen-MIUI15.apk  
指南针  
product\data-app\MIUICompass\MIUICompass.apk  
传送门（更新为网络来源3.2.5版本，支持横屏触发）  
product\priv-app\MIUIContentExtension\MIUIContentExtension.apk  
添加传送门所需权限  
product\etc\permissions\privapp-permissions-product.xml  
```
   <privapp-permissions package="com.miui.contentextension">
      <permission name="android.permission.WRITE_SECURE_SETTINGS" />
   </privapp-permissions>
```
## system分区不修改，直接照搬6s Pro
可选修改  
签名破解，要修改系统app，就需要修改services.jar文件，我这里使用的SYT_ROM工具提供的插件自动修改  
system\system\framework\services.jar  
build.prop修改机型代号、版本指纹  
system\system\system_dlkm\etc\build.prop
```
ro.system_dlkm.build.fingerprint=Android/missi_pad_cn/missi:14/UKQ1.231003.002/V816.0.7.0.UKYCNXM:user/release-keys
ro.system_dlkm.build.version.incremental=V816.0.7.0.UKYCNXM
```
system\system\build.prop
```
ro.system.build.fingerprint=Android/missi_pad_cn/missi:14/UKQ1.231003.002/V816.0.7.0.UKYCNXM:user/release-keys
ro.system.build.version.incremental=V816.0.7.0.UKYCNXM
ro.build.version.incremental=V816.0.7.0.UKYCNXM
```
## system_ext分区不修改，直接照搬6s Pro
可选修改  
状态栏歌词（作者来自酷安 白羊唐黎明）  
system_ext\priv-app\MiuiSystemUI  
反编译MiuiSystemUI.apk，修改完成后替换，并删除两个oat文件  
system_ext\priv-app\Settings  
反编译Settings.apk，修改完成后替换，并删除两个oat文件  
我这里使用的方法是先把两个apk复制到新建文件夹，然后使用[APKEditor](https://github.com/REAndroid/APKEditor) 反编译，得到MiuiSystemUI_decompile_xml和Settings_decompile_xml文件夹，按照酷安教程修改，他原教程是使用手机修改的，用电脑修改会稍微有点区别，具体修改内容可以参考我上传的[pad6sp_statusbar_lyric](https://github.com/ymdzq/pad6sp_statusbar_lyric/)仓库，他附件提供的有一个xml是加密xml，电脑上处理不了，所以那个xml我是直接从他的米13官改包里反编译拿的  

build.prop修改机型代号、版本指纹  
system_ext\etc\build.prop
```
ro.system_ext.build.fingerprint=Android/missi_pad_cn/missi:14/UKQ1.231003.002/V816.0.7.0.UKYCNXM:user/release-keys
ro.system_ext.build.version.incremental=V816.0.7.0.UKYCNXM
```
## vendor分区修改，整体上用5pro的，但要注意以下部分
vendor/build.prop加入代码  
```
#玄学优化代码
# fix the drop frame issus
ro.surface_flinger.enable_frame_rate_override=false
persist.vendor.mi_sf.optimize_for_refresh_rate.enable=1
ro.vendor.mi_sf.ultimate.perf.support=true
ro.surface_flinger.use_content_detection_for_refresh_rate=false
ro.surface_flinger.set_touch_timer_ms=0
ro.surface_flinger.set_idle_timer_ms=0
ro.build.recovery.version.release=14
debug.sf.auto_latch_unsignaled=0
```
`ro.millet.netlink`上面已经加到product分区的build.prop里了，所以vendor里不需要重复添加  

骁龙GPU驱动更新（来自酷安 诺蓝）  
vendor\lib  
vendor\lib64
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
