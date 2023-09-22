# 小米平板5 PRO 移植小米平板6 MAX MIUI 14记录
资源来源于网络，仅供交流学习，不得用做任何商业用途，不提供任何技术支持，请在下载后24小时内删除  
基于miui_ELISH_V14.0.23.7.31，移植文件来源于miui_YUDI_V14.0.6.0  
由于是同一个安卓版本同一个MIUI大版本移植，所以需要修改的内容不多  
本文仅记录一下修改内容，具体修改行以及内容以实际文件对比结果为准  

这里有两个选择，  
第一个选择是很多有的没的（像是控制中心工作台开关、设置里的平板专区、平行世界拖拽点、会议工具箱等等）是6max专属新功能，有机型验证，  
如果需要解锁这些功能，要么用模块hook或者apktool反编译对应文件改判断代码，要么就全局修改机器代号，把所有代号全改成yudi，  
但是相对来说，因为硬件不匹配，改代号一般都会有bug，所以如果要追求稳定，代号最好就还是用elish，用户要改代号可以用模块改。  
第二个选择就是是否集成pc版wps这一套东西，  
这个玩意2.44GB，目前小米是都放在vendor分区，  
因为有一个vendor服务启动项的mslgservice.rc文件，每次开机挂载镜像，解压rootfs，把linux容器放到/data/vendor/mslg/rootfs  
服务启动优先级更高，所以不能普通的用面具模块代替，除非你重写一个shell脚本开机执行代替mslgservice.rc服务，而且据说不能直接用面具的losetup得用系统的，面具的权限太高了运行不了，就很麻烦。  
如果就这样直接集成，就需要扩容机器的super分区才能刷得进去，或者你就干脆做成dsu系统包，直接dsu侧载就不管包多大也不需要扩容。  
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
如果不集成pc版wps就可以删除  
product\app\MSLgRdp   
product\data-app\WpsLauncher  

product\app  
删除骁龙870算力不够导致离线字幕识别功能闪退，无法使用的6max小爱翻译 AiAsstVision  
保留5pro小爱翻译 AiAsstVision（开发板内置的版本号是3.2.7，目前可以用在线字幕识别的8月最新版是3.3.3）  
删除无法使用的6max人脸识别解锁 MiuiBiometric  
保留5pro人脸识别解锁 MiuiBiometric3373  
删除6max的手写笔和键盘设置 MiuiInputSettings_M80 据说会导致有线鼠标操作失灵  
保留5pro的手写笔和键盘设置 MiuiInputSettings  

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

修改屏幕亮度配置文件  
product\etc\displayconfig\display_id_4630946932993367170.xml  
目前6max只有一家屏幕供应商，后续更新可能会随着增加屏幕类型，而多出其他id的文件，  
而这个东西的文件名是在其他地方写死的，只能用display_id_4630946932993367170.xml，否则会出现`*** FATAL EXCEPTION IN SYSTEM PROCESS: android.display`报错无法开机  

5pro屏幕的xml文件为：  
product\etc\displayconfig\display_id_19260527152667265.xml  
product\etc\displayconfig\display_id_4630946481717202305.xml  
product\etc\displayconfig\display_id_4630946545580055169.xml  
这三个文件的内容是完全一样的，所以只要用任意一个里面的内容替换掉display_id_4630946932993367170.xml屏幕亮度调节就正常了  
屏幕亮度曲线，三个点，随意调一调，就加个400nits中间值好了，反正决定亮度的值是在MiuiFrameworkResOverlay.apk里，value是直接抄yudi，最大最小值也是有其他地方决定的，改的太离谱也会出现上面那个报错  
```
        <point>
            <value>0.001709819</value>
            <nits>2.0</nits>
        </point>
        <point>
            <value>0.49975574</value>
            <nits>400.0</nits>
        </point>
        <point>
            <value>1.0</value>
            <nits>500</nits>
        </point>
```
build.prop修改机型代号、版本指纹，设置默认屏幕密度，关闭内存扩展  
product\etc\build.prop
```
ro.product.product.name=elish
ro.product.build.fingerprint=Xiaomi/elish/missi:13/TKQ1.221114.001/V14.0.6.0.TMHCNXM:user/release-keys
ro.product.mod_device=elish

ro.sf.lcd_density=360
persist.miui.density_v2=360

persist.miui.extm.enable=0
```
默认主题谷歌支付无分层图标功能，平板6max好像直接就没这个文件夹，从6pro的包里提取一个，改好包名复制过来
product\media\theme\miui_mod_icons\dynamic\com.google.android.apps.nbu  

overlay保留5pro本身设备的apk  
product\overlay\DevicesAndroidOverlay.apk  
product\overlay\DevicesOverlay.apk  
product\overlay\MiuiBiometricResOverlay.apk  
product\overlay\MiuiFrameworkResOverlay.apk  

保留5pro相机，删除6max相机，否则会提示机型不匹配无法使用然后退出，  
我个人是建议使用sevtinge修改相机4.7.230127.0版本，没有机型限制而且解锁更多功能  
product\priv-app\MiuiCamera
## system分区修改，整体上照搬6max，但要注意以下部分
build.prop修改机型代号  
system\system\build.prop  
```
ro.product.mod_device=elish
```
## system_ext分区无需修改，直接照搬6max
## vendor分区修改，如果选择不集成pc版wps无需修改，直接用5pro的
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
接下来是补充selinux的上下文权限，  
补个锤子  
我看鲁迅的霸权、曾小理的移植包倒是根本就没改，直接改selinux宽容，然后mslgservice.rc把所有seclabel的mslgd改成shell，mslgservice直接不要了那行（修改方法感谢水龙指导）  

修改mslgservice.rc文件  
vendor\etc\init\mslgservice.rc  
```
service mslgservice /vendor/bin/hw/mslgservice
    class core
    user root
    disabled
    oneshot

service tar_rootfs /vendor/bin/tar-rootfs.sh ${vendor.mslgrootfs.version}
    class core
    user root
    seclabel u:r:shell:s0
    disabled
    oneshot

service losetup_rootfs /vendor/bin/losetup.sh
    class core
    user root
    seclabel u:r:shell:s0
    disabled
    oneshot

service mslgrootfs /vendor/bin/start-rootfs.sh
    class core
    user root
    seclabel u:r:shell:s0
    disabled
    oneshot
```
## 重新打包mi_ext、odm、system、system_ext、vendor、product分区
先用make_ext4fs或者e2fsdroid+mke2fs打包为raw image，  
然后用lpmake打包成super img  
这里的目标是打包成sparse格式的super.img（线刷必须要sparse格式，卡刷不一定取决于刷机脚本怎么写），vab机器一般是线刷用fastboot刷进super分区，卡刷是在recovery里用卡刷脚本写入到super分区，  
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
我看米欧是修改的十六进制0000007B这个地址00改成02  
另一个办法，用户刷入vbmeta、vbmeta_system时使用命令关闭avb验证或者在twrp中直接用选项关闭
```bash
fastboot --disable-verity --disable-verification flash vbmeta vbmeta.img
fastboot --disable-verity --disable-verification flash vbmeta_system vbmeta_system.img
```
