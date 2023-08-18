# 小米平板5 PRO 移植小米平板6 MAX MIUI 14记录
资源来源于网络，仅供交流学习，不得用做任何商业用途，不提供任何技术支持，请在下载后24小时内删除  
基于miui_ELISH_V14.0.4.0，移植文件来源于miui_YUDI_V14.0.3.0  
由于是同一个安卓版本同一个MIUI大版本移植，所以需要修改的内容不多  
本文准备仅记录一下修改内容，具体修改行以及内容以实际文件对比结果为准，先打一个草稿，慢慢更新  

这里有两个选择，  
第一个选择是很多有的没的（像是控制中心工作台开关、设置里的平板专区、平行世界拖拽点、会议工具箱等等）是6max专属新功能，有机型验证，  
如果需要解锁这些功能，要么用模块hook或者apktool反编译对应文件改判断代码，要么就全局修改机器代号，把所有代号全改成yudi，  
但是相对来说，因为硬件不匹配，改代号一般都会有bug，所以如果要追求稳定，代号最好就还是用elish，用户要改代号可以用模块改。  
第二个选择就是是否集成pc版wps这一套东西，  
这个玩意2.44GB，目前小米是都放在vendor分区，  
因为有一个vendor服务启动项的mslgservice.rc文件，每次开机挂载镜像，解压rootfs，把linux容器放到/data/vendor/mslg/rootfs  
服务启动优先级更高，所以不能普通的用面具模块代替，除非你重写一个shell脚本开机执行代替mslgservice.rc服务，而且据说不能直接用面具的losetup得用系统的，面具的权限太高了运行不了，就很麻烦。  
如果就这样直接集成，就需要扩容机器的super分区才能刷得进去，或者你就干脆做成dsu系统包，直接dsu侧载就不管包多大也不需要扩容。  
如果不集成，随便精简一点东西，就可以确保刷进机器那8.5G的super分区。  

## mi_ext分区修改，未完成

## odm分区修改，未完成

## product分区修改，整体上照搬6max，但要注意以下部分未完成

pc版wps相关文件  
访问linux容器的rdp后端MSLgRdp和交互操作的前端WpsLauncher  
如果不集成pc版wps就可以删除  
product\app\MSLgRdp   
product\data-app\WpsLauncher  

data-app可卸载的预装app，其中不少app都是可以在应用商店里重新安装的，  
因为平板5pro默认的super分区只有8.5G，而且重新打包必须预留更多空间，所以可以精简这里，把super精简到7G左右，越小越好  
不过如果你要塞pc版wps，就一定是扩容了机器的super分区，搞不好空间还有多就没必要精简了，未完成  
product\data-app\

设备功能配置文件，本来正常代号要用elish稳定使用的话，删除yudi.xml，照搬elish.xml就好了，  
但是如果你要全局改机器代号的话，这里配置文件也要改名成yudi.xml，  
所以我是建议干脆把elish.xml复制两份一个叫elish.xml一个叫yudi.xml，都放进去，这样用哪个代号也不要紧  
product\etc\device_features\elish.xml  
product\etc\device_features\yudi.xml  

build.prop修改，未完成  
product\etc\build.prop

overlay保留5pro本身设备的apk，未完成  
product\overlay\DevicesAndroidOverlay.apk  
product\overlay\DevicesOverlay.apk  
product\overlay\MiuiBiometricResOverlay.apk  
product\overlay\MiuiFrameworkResOverlay.apk  

## system分区修改，整体上照搬6max，但要注意以下部分未完成

系统app，未完成  
system\system\app  

data-app，未完成  
system\system\data-app\  

键位文件由于从内容上看6max的文件已经包含了5pro的所有内容，还有一点补充，所以直接照搬  
system\system\usr\keylayout  

build.prop修改，未完成  
system\system\build.prop  

## system_ext分区无需修改，直接照搬6max

## vendor分区修改，整体上保留5pro，但要注意以下部分未完成

6max新增pc版wps相关文件，只要对比6pro（liuqin）的整个vendor分区，看孤立文件，一眼就能看出这些文件跟pc版wps有关，  
其中mslgoptimg、mslgusrimg两个1G以上大文件，是导致super分区需要扩容的原因，  
所以如果能以某种方法比如这里留一个到userdata的链接，然后把实际文件丢进userdata，  
或者直接改sh脚本把位置就写到其他地方，就不需要占用vendor、super分区了  
另外说一句，由于有人测试了，这东西类原生也可以用，所以如果你能把这些相关文件还有上面product分区里面的两个app放进其他安卓系统的对应位置，其他安卓系统搞不好也能运行  
```
/vendor/bin/hw/mslgservice
/vendor/bin/losetup.sh
/vendor/bin/start-rootfs.sh
/vendor/bin/tar-rootfs.sh

/vendor/etc/assets/md5.txt
/vendor/etc/assets/mslgoptimg
/vendor/etc/assets/mslgusrimg
/vendor/etc/assets/rootfs-23.07.28.tgz

/vendor/etc/init/mslgservice.rc

/vendor/lib/libext2_uuid.so

/vendor/lib64/libext2_uuid.so
/vendor/lib64/vendor.xiaomi.mslg.keeper@1.0.so
```
修改权限设置文件，未完成  
vendor\etc\selinux\vendor_file_contexts  

修改se策略文件，未完成  
vendor\etc\selinux\vendor_sepolicy.cil  

build.prop修改，未完成  
vendor\odm\etc\build.prop

build.prop修改，未完成  
vendor\odm_dlkm\etc\build.prop

build.prop修改，未完成  
vendor\vendor_dlkm\etc\build.prop

build.prop修改，未完成  
vendor\build.prop

## 重新打包mi_ext、odm、system、system_ext、vendor、product分区，未完成
先用make_ext4fs或者e2fsdroid+mke2fs打包为raw image，  
这里的目标是打包成super.img，vab机器一般是线刷用fastboot刷进super分区，卡刷用zstd压缩后在recovery里解压到super分区
解包打包偷懒就找个安卓工具箱，米欧、dna、多幸运之类的，直接一键打包

## avb验证文件，直接替换成关闭avb的文件  
vbmeta.img
