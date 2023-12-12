# 小米平板5 PRO 移植小米平板6 MAX HyperOS记录
资源来源于网络，仅供交流学习，不得用做任何商业用途，不提供任何技术支持，请在下载后24小时内删除  
基于miui_ELISH_V14.0.5.0，移植文件来源于YUDI_OS1.0.2.0  
其中由于我自己开原包，系统设置会无限闪退，目测是平板专区等6max专属功能检测到5Pro的型号不支持，澎湃早期版本有bug小米没写完对老设备的兼容所导致的，可能需要修改apk，没时间，没脑子，搞不定，一旦要改系统apk我就搞不定miui的签名，就必须同时改核心破解  
所以system、system_ext部分直接抄的[柚坛移植包](https://www.uotan.cn/resources/hyperos_1-0-2_for_-5pro.507/)感谢暮间雾大佬  
这里推荐一下隔壁大佬的[HyperOS 移植项目](https://github.com/toraidl/hyperos_port)，有很多移植澎湃的经验、修改启发  
本文仅记录一下修改内容，具体修改行以及内容以实际文件对比结果为准  

由于修改了系统文件，所以avb验证肯定是要关的，而想保证各种app兼容性，所以我选择保持selinux enforce，即不集成pc版wps  
如果不集成，就不需要改vendor分区，随便在product分区里精简一点东西，就可以确保刷进机器那8.5G的super分区。  
## mi_ext分区修改，整体上照搬6max，但要注意以下部分
build.prop修改机型代号，这里这个代号是miui ota更新服务器用来识别推送更新用的，你都刷第三方rom了这个就不重要了，除非你能用到那个服务器推送更新  
mi_ext\etc\build.prop
```
ro.product.mod_device=elish
```
## odm分区无修改
这个分区是跟vendor分区配套的，目前无需修改  
## product分区修改，整体上照搬6max，但要注意以下部分
pc版wps相关文件  
访问linux容器的rdp后端MSLgRdp和交互操作的前端WpsLauncher  
不集成pc版wps可以直接删除  
product\app\MSLgRdp   
product\data-app\WpsLauncher  

product\app  
删除骁龙870算力不够导致离线字幕识别功能闪退，无法使用的6max小爱翻译 AiAsstVision  
保留5pro小爱翻译 AiAsstVision（MIUI 14内置的版本号是3.2.7，目前可以用在线字幕识别的8月最新版是3.3.3）  
（目前12月小米在应用商店全平台推送了4.7.0更新，这个版本据说在开发版上可以启动字幕识别在线模型兼容算力不足老设备，但是我测试的在这个版本澎湃上还是离线模型闪退，所以不更新）  
删除无法使用的6max人脸识别解锁 Biometric  
保留5pro人脸识别解锁 MiuiBiometric3373  
我选择删除两个oat文件  
product\app\Biometric\oat\arm64\Biometric.odex  
product\app\Biometric\oat\arm64\Biometric.vdex  
然后直接把MiuiBiometric3373.apk改名成Biometric.apk替换  
以及保留配套的5pro的overlay资源文件  
product\overlay\MiuiBiometricResOverlay.apk  

按需精简  
快应用服务引擎  
product\app\HybridPlatform  
智能服务  
product\app\MSA  

data-app可卸载的预装app，其中不少app都是可以在应用商店里重新安装的，  
product\data-app\  
因为平板5pro默认的super分区只有8.5G，而且重新打包必须预留更多空间，所以可以精简这里，把super精简到7.4G以下，越小越好  
不过如果你要塞pc版wps，就一定是扩容了机器的super分区，搞不好空间还有多就没必要精简了  
我个人是觉得没必要只为了一个难用的特别版wps，就去搞极限精简，很多常用自带功能用户到时候又要想办法装回来，挺烦人的  
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
米家  
product\data-app\SmartHome  

设备功能配置文件，本来正常代号要用elish稳定使用的话，删除yudi.xml，照搬elish.xml就好了，  
但是如果你要全局改机器代号的话，这里配置文件也要改名成yudi.xml，  
所以我是建议干脆把elish.xml复制两份一个叫elish.xml一个叫yudi.xml，都放进去，这样用哪个代号也不要紧  
product\etc\device_features\elish.xml  
product\etc\device_features\yudi.xml  

修改屏幕亮度配置文件  
product\etc\displayconfig\display_id_4630946932993367170.xml  
目前6max只有一家屏幕供应商，由于没有再之前版本出现的`*** FATAL EXCEPTION IN SYSTEM PROCESS: android.display`报错无法开机的问题  

5pro屏幕的xml文件为：  
product\etc\displayconfig\display_id_19260527152667265.xml  
product\etc\displayconfig\display_id_4630946481717202305.xml  
product\etc\displayconfig\display_id_4630946545580055169.xml  
这三个文件的内容是完全一样的，所以我选择覆盖掉display_id_4630946932993367170.xml，并且保留这四个xml文件，屏幕亮度调节就正常了  
这里需要注意Overlay里面的AospFrameworkResOverlay.apk要换成5Pro的，否则会遇到自动亮度导致系统软重启的问题  
product\overlay\AospFrameworkResOverlay.apk  
本来按理说是要反编译这个apk，对比6max的澎湃os与miui14改动，修改到5Pro的文件上的，能力有限，不想努力了  

build.prop修改机型代号、版本指纹，设置默认屏幕密度，关闭内存扩展  
product\etc\build.prop
```
ro.product.product.name=elish
ro.product.build.fingerprint=Xiaomi/elish/miproduct:14/UKQ1.230804.001/V816.0.2.0.UMHCNXM:user/release-keys

ro.sf.lcd_density=360
persist.miui.density_v2=360

persist.miui.extm.enable=0

注释掉这行，因为cust分区格式不是erofs
#ro.miui.cust_erofs=1
```
overlay保留5pro本身设备的apk  
DevicesAndroidOverlay主要影响圆角弧率、状态栏高度，aod服务（lcd没有）  
product\overlay\DevicesAndroidOverlay.apk  
DevicesOverlay主要影响导航栏（小白条）布局以及圆角，充电动画  
product\overlay\DevicesOverlay.apk  
MiuiFrameworkResOverlay主要影响屏幕hbm背光、hbm亮度曲线、以及一些网络制式的属性  
product\overlay\MiuiFrameworkResOverlay.apk  

删除6max相机，否则会提示机型不匹配无法使用然后退出，  
目前澎湃只能用5.0以上版本的相机，老apk无法使用，同样会提示机型不匹配无法使用然后退出，  
我所以只能用sevtinge修改相机5.0.230706.0版本，没有机型限制而且解锁更多功能  
缺点就是ui不匹配，横屏影像会错位，但由于没的其他选择，只能用谷歌相机、骁龙相机这种第三方相机  
product\priv-app\MiuiCamera  
并且删除两个oat文件
## system分区不修改，直接照搬暮间雾大佬的raw image，经对比以下部分为修改文件
system\system\framework\services.jar  
反编译，大致修改为核心破解，但是具体修改内容详询暮间雾大佬  
## system_ext分区不修改，直接照搬暮间雾大佬的raw image，经对比以下部分为修改文件
system_ext\priv-app\Settings\Settings.apk  
反编译，大致修改为找到平板相关新功能的代码，修改平板6系列机型代号为平板5系列机型代号，但是具体修改内容详询暮间雾大佬  

为什么system、system_ext要照搬暮间雾大佬的raw image呢？因为系统设置闪退，用他的这两个镜像就没有问题，而我自己拆原包替换里面两个文件之后仍然卡第二屏开机动画，目测为拆包打包软件不兼容，二次拆包容易损坏文件系统
## vendor分区修改，整体上用5pro的，但要注意以下部分
vendor/build.prop加入代码  
修复millet  
```
#add millet netlink property
ro.millet.netlink=29
```
咦，这段代码我是不是可以加到其他分区的build.prop里，vendor分区就彻底不用改了  
可选修改fstab.qcom：是否更新mi_ext分区相关内容，另外不推荐动userdata，一个搞不好就用户数据火葬场
## 可选product分区修改，补全小米平板缺失的工具app
我用的k60手机官方澎湃，所以我这里的app是用的手机版补，按理说是要用mix fold3折叠屏的app更好  
悬浮球  
product\app\MIUITouchAssistant  
小爱建议  
product\app\XiaoaiRecommendation  
小米运动健康  
product\data-app\Health\Health.apk  
小米锁屏画报  
product\data-app\MIGalleryLockscreen-MIUI15\MIGalleryLockscreen-MIUI15.apk  
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
## 重新打包mi_ext、odm、system、system_ext、vendor、product分区
先用make_ext4fs或者e2fsdroid+mke2fs打包为raw image，  
然后用lpmake打包成super img  
vab机器一般是线刷用fastboot刷进super分区，卡刷是在recovery里用卡刷脚本写入到super分区，  
常见的情况也有使用zstd工具把super压缩成zst格式（打包zst需要raw格式的super.img），在线刷、卡刷的时候再解压，这种用压缩解压的时间来节省刷机包占用空间大小的做法，  
这种的情况就需要专门的脚本和工具了  
由于无wps版由于不需要修改odm、vendor分区，所以理论上其实你可以直接用fastbootd模式刷入mi_ext、system、system_ext、product分区  
dsu包的做法就是直接把mi_ext、system、system_ext、product分区的raw image文件打包成一个zip或者gz文件即可  
解包打包偷懒就找个安卓工具箱，米欧、dna、多幸运之类的，直接一键打包  
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
