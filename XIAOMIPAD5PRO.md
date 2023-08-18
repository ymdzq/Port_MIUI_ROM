# 小米平板5 PRO 移植小米平板6 MAX MIUI 14记录
资源来源于网络，仅供交流学习，不得用做任何商业用途，不提供任何技术支持，请在下载后24小时内删除  
基于miui_ELISH_V14.0.4.0，移植文件来源于miui_YUDI_V14.0.3.0  
由于是同一个安卓版本同一个MIUI大版本移植，而且已经有两个大佬发了现成的免费rom包，所以需要修改的内容不多  
本文准备仅记录一下修改内容，具体修改行以及内容以实际文件对比结果为准，先打一个草稿，慢慢更新

## mi_ext分区修改，未完成

## odm分区修改，未完成

## product分区修改，整体上照搬6max，但要注意以下部分未完成

本来是在剃刀计划里的可卸载预装app，而且很多app都是可以在应用商店里重新安装的，  
因为平板5pro的super分区只有8.5G，而且重新打包必须预留更多空间，所以需要精简这里，未完成  
product\data-app\

设备功能配置文件，本来正常要稳定使用的话，删除yudi.xml，照搬elish.xml就好了，  
但是因为很多有的没的新功能（像是控制中心工作台开关、设置里的平板专区、平行世界拖拽点、会议工具箱等等）是6max专属，有机型验证，  
所以如果需要解锁这些功能，改了全局机器型号后，这里配置文件也要改名成yudi.xml，  
所以我是建议干脆把elish.xml复制两份一个elish.xml一个yudi.xml，都放进去也不要紧  
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

6max新增pc wps相关文件，整个vendor分区对比6pro（liuqin），看孤立文件，一眼就能看出哪些文件跟pc wps有关，未完成  

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
这里的目标是打包成super.img
解包打包偷懒就找个安卓工具箱，米欧、dna、多幸运之类的，直接一键打包


## avb验证文件，直接替换成关闭avb的文件  
vbmeta.img
