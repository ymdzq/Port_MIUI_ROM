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

按需精简  
product\app  
快应用服务引擎  
product\app\HybridPlatform  
智能服务  
product\app\MSA  

data-app可卸载的预装app，其中不少app都是可以在应用商店里重新安装的，  
product\data-app\  
因为平板5pro默认的super分区只有8.5G，而且重新打包必须预留更多空间，所以可以精简这里，把super精简到7G左右，越小越好  
不过如果你要塞pc版wps，就一定是扩容了机器的super分区，搞不好空间还有多就没必要精简了  
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
米家  
product\data-app\SmartHome  

设备功能配置文件，本来正常代号要用elish稳定使用的话，删除yudi.xml，照搬elish.xml就好了，  
但是如果你要全局改机器代号的话，这里配置文件也要改名成yudi.xml，  
所以我是建议干脆把elish.xml复制两份一个叫elish.xml一个叫yudi.xml，都放进去，这样用哪个代号也不要紧  
product\etc\device_features\elish.xml  
product\etc\device_features\yudi.xml  

屏幕亮度配置文件  
product\etc\displayconfig  
保留5pro本身屏幕的xml文件  
product\etc\displayconfig\display_id_19260527152667265.xml  
product\etc\displayconfig\display_id_4630946481717202305.xml  
product\etc\displayconfig\display_id_4630946545580055169.xml  
删除6max屏幕的xml文件  
product\etc\displayconfig\display_id_4630946932993367170.xml  

build.prop修改机型代号、版本指纹，设置默认屏幕密度，关闭内存扩展  
product\etc\build.prop
```
ro.product.product.name=elish
ro.product.build.fingerprint=Xiaomi/elish/missi:13/TKQ1.221114.001/V14.0.3.0.TMHCNXM:user/release-keys
ro.product.mod_device=elish

ro.sf.lcd_density=360
persist.miui.density_v2=360

persist.miui.extm.enable=0

```
overlay保留5pro本身设备的apk，未完成  
product\overlay\DevicesAndroidOverlay.apk  
product\overlay\DevicesOverlay.apk  
product\overlay\MiuiBiometricResOverlay.apk  
product\overlay\MiuiFrameworkResOverlay.apk  

## system分区修改，整体上照搬6max，但要注意以下部分

build.prop修改机型代号  
system\system\build.prop  
```
ro.product.mod_device=elish
```

## system_ext分区无需修改，直接照搬6max

## vendor分区修改，整体上保留5pro，但要注意以下部分

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
vendor/build.prop加入代码  
```
ro.vendor.mslg.rootfs.version=rootfs-23.07.28.tgz
sys.mslg.available=1
```
接下来是补充selinux的上下文权限，别人的移植包也没补，我也不知道补了有什么用，也不知道写的对不对，反正就是照yudi的文件抄，补了  

修改vendor文件上下文文件  
vendor\etc\selinux\vendor_file_contexts  
```
/(vendor|system/vendor)/bin/hw/vendor.xiaomi.mslg.keeper@1.0-service     u:object_r:hal_mslgkeeper_default_exec:s0
/(vendor|system/vendor)/bin/start-rootfs.sh     u:object_r:mslgd_exec:s0
/(vendor|system/vendor)/bin/tar-rootfs.sh     u:object_r:mslgd_exec:s0
/(vendor|system/vendor)/bin/losetup.sh     u:object_r:mslgd_exec:s0
/(vendor|system/vendor)/bin/hw/mslgservice     u:object_r:mslgd_exec:s0
/(vendor|system/vendor)/etc/assets(/.*)? u:object_r:vendor_file:s0
/(vendor|system/vendor)/etc/assets/mslgoptimg u:object_r:vendor_file:s0
/(vendor|system/vendor)/etc/assets/mslgusrimg u:object_r:vendor_file:s0

/dev/msl(/.*)?     u:object_r:mslg_rootfs_file:s0

/data/vendor/mslg(/.*)?     u:object_r:mslg_rootfs_file:s0
```
修改vendor服务上下文文件  
vendor\etc\selinux\vendor_hwservice_contexts  
```
vendor.xiaomi.mslg.keeper::IMSLgKeeper       u:object_r:hal_mslgkeeper_hwservice:s0
```
修改vendor属性上下文文件  
vendor\etc\selinux\vendor_property_contexts  
```
#line 1 "out/soong/.intermediates/system/sepolicy/vendor_property_contexts/android_common/gen/namespace_checked/vendor/xiaomi/proprietary/mslg/keeper/1.0/default/sepolicy/property_contexts"
persist.vendor.unzip.mslgrootfs    u:object_r:vendor_mslg_prop:s0
vendor.mslgrootfs.isready    u:object_r:vendor_mslg_prop:s0
vendor.mslgrootfs.version    u:object_r:vendor_mslg_prop:s0
vendor.setup.mslgrootfs    u:object_r:vendor_mslg_prop:s0
vendor.mslg.mslgusrimg    u:object_r:vendor_mslg_prop:s0
vendor.mslg.mslgoptimg    u:object_r:vendor_mslg_prop:s0
ro.vendor.mslg.rootfs.version    u:object_r:vendor_mslg_prop:s0
```
修改vendor seapp上下文文件  
vendor\etc\selinux\vendor_seapp_contexts  
```
user=_app seinfo=platform name=com.xiaomi.mslgrdp domain=mslg_app type=app_data_file levelFrom=all
```
修改se策略文件，这段需要补的权限超级多，痛苦  
vendor\etc\selinux\vendor_sepolicy.cil  
```
(typeattributeset domain (adbd_30_0开头的这行到下一行，最后面的))之前，按照前面的格式加上
hal_mslgkeeper_default mslg_app mslg_init mslgd

(typeattributeset fs_type (device_30_0开头的这行到下一行，最后面的))之前，按照前面的格式加上
mslg_app_devpts

(typeattributeset file_type (adbd_exec_30_0开头的这行到下三行，最后面的))之前，按照前面的格式加上
mslg_rootfs_file hal_mslgkeeper_default_exec mslg_init_exec mslgd_exec

(typeattributeset exec_type (adbd_exec_30_0开头的这行到下一行，最后面的))之前，按照前面的格式加上
hal_mslgkeeper_default_exec mslg_init_exec mslgd_exec

(typeattributeset data_file_type (incremental_control_file_30_0开头的这行到下一行，最后面的))之前，按照前面的格式加上
mslg_rootfs_file

(typeattributeset vendor_file_type (vendor_cgroup_desc_file_30_0开头的这行到下一行，最后面的))之前，按照前面的格式加上
hal_mslgkeeper_default_exec mslg_init_exec mslgd_exec

(typeattributeset property_type (apexd_prop_30_0开头的这行到下一行，最后面的))之前，按照前面的格式加上
vendor_mslg_prop

(typeattributeset vendor_property_type (rebootescrow_hal_prop_30_0开头的这行到最后面的))之前，按照前面的格式加上
vendor_mslg_prop

(typeattributeset vendor_public_property_type (开头的这行到最后面的))之前，按照前面的格式加上
vendor_mslg_prop

(typeattributeset hwservice_manager_type (default_android_hwservice_30_0开头的这行到下一行，最后面的))之前，按照前面的格式加上
hal_mslgkeeper_hwservice

(typeattributeset mlstrustedsubject (bufferhubd_30_0开头的这行到最后面的))之前，按照前面的格式加上
mslg_init mslgd

(typeattributeset mlstrustedobject (ashmem_device_30_0开头的这行到最后面的))之前，按照前面的格式加上
mslg_rootfs_file

(typeattributeset appdomain (开头的这行到最后面的))之前，按照前面的格式加上
mslg_app

(typeattributeset netdomain (dhcp_30_0(开头的这行到最后面的))之前，按照前面的格式加上
mslg_app

新增行
(typeattributeset bluetoothdomain (mslg_app))
(typeattributeset coredomain (mslg_app))
(typeattributeset socket_between_core_and_vendor_violators (mslg_app mslgd))

(typeattributeset halserverdomain (mediaswcodec_30_0开头的这行到最后面的))之前，按照前面的格式加上
hal_mslgkeeper_default

新增行
(typeattribute hal_mslgkeeper)
(typeattributeset hal_mslgkeeper (hal_mslgkeeper_default))
(typeattribute hal_mslgkeeper_server)
(typeattributeset hal_mslgkeeper_server (hal_mslgkeeper_default))
(typeattribute hal_mslgkeeper_client)

新增行
(type mslg_rootfs_file)
(roletype object_r mslg_rootfs_file)
(type hal_mslgkeeper_default)
(roletype object_r hal_mslgkeeper_default)
(type hal_mslgkeeper_default_exec)
(roletype object_r hal_mslgkeeper_default_exec)
(type hal_mslgkeeper_hwservice)
(roletype object_r hal_mslgkeeper_hwservice)
(type mslg_app)
(roletype object_r mslg_app)
(type mslg_app_userfaultfd)
(roletype object_r mslg_app_userfaultfd)
(type mslg_app_devpts)
(roletype object_r mslg_app_devpts)
(type mslg_init)
(roletype object_r mslg_init)
(type mslg_init_exec)
(roletype object_r mslg_init_exec)
(type mslgd)
(roletype object_r mslgd)
(type mslgd_exec)
(roletype object_r mslgd_exec)
(type vendor_mslg_prop)
(roletype object_r vendor_mslg_prop)

新增行
(allow hal_mslgkeeper_client hal_mslgkeeper_server (binder (call transfer)))
(allow hal_mslgkeeper_server hal_mslgkeeper_client (binder (transfer)))
(allow hal_mslgkeeper_client hal_mslgkeeper_server (fd (use)))
(allow hal_mslgkeeper_server hal_mslgkeeper_client (binder (call transfer)))
(allow hal_mslgkeeper_client hal_mslgkeeper_server (binder (transfer)))
(allow hal_mslgkeeper_server hal_mslgkeeper_client (fd (use)))
(allow hal_mslgkeeper_server hal_mslgkeeper_hwservice (hwservice_manager (add find)))
(allow hal_mslgkeeper_server hidl_base_hwservice_30_0 (hwservice_manager (add)))
(neverallow base_typeattr_724_30_0 hal_mslgkeeper_hwservice (hwservice_manager (add)))
(allow init_30_0 hal_mslgkeeper_default_exec (file (read getattr map execute open)))
(allow init_30_0 hal_mslgkeeper_default (process (transition)))
(allow hal_mslgkeeper_default hal_mslgkeeper_default_exec (file (read getattr map execute open entrypoint)))
(dontaudit init_30_0 hal_mslgkeeper_default (process (noatsecure)))
(allow init_30_0 hal_mslgkeeper_default (process (siginh rlimitinh)))
(typetransition init_30_0 hal_mslgkeeper_default_exec process hal_mslgkeeper_default)
(allow hal_mslgkeeper_default hwservicemanager_30_0 (binder (call transfer)))
(allow hwservicemanager_30_0 hal_mslgkeeper_default (binder (call transfer)))
(allow hwservicemanager_30_0 hal_mslgkeeper_default (dir (search)))
(allow hwservicemanager_30_0 hal_mslgkeeper_default (file (read map open)))
(allow hwservicemanager_30_0 hal_mslgkeeper_default (process (getattr)))
(allow hal_mslgkeeper_default vendor_mslg_prop (file (read getattr map open)))
(allow hal_mslgkeeper_default property_socket_30_0 (sock_file (write)))
(allow hal_mslgkeeper_default init_30_0 (unix_stream_socket (connectto)))
(allow hal_mslgkeeper_default vendor_mslg_prop (property_service (set)))
(allow hal_mslgkeeper_default vendor_mslg_prop (file (read getattr map open)))
(allow hal_mslgkeeper_default mslg_rootfs_file (file (read getattr open)))
(allow hal_mslgkeeper_default mslg_rootfs_file (dir (read getattr open search)))
(allow init_30_0 mslg_rootfs_file (dir (mounton)))
(allow init_30_0 mslg_rootfs_file (file (mounton)))
(allow init_30_0 fuse_30_0 (dir (mounton)))
(allow init_30_0 fuse_30_0 (file (mounton)))
(allow init_30_0 mnt_user_file_30_0 (lnk_file (read write create open)))
(allow hal_misys_default mslg_rootfs_file (file (read getattr open)))
(allow hal_misys_default mslg_rootfs_file (dir (read getattr open search)))
(typetransition mslg_app tmpfs_30_0 file appdomain_tmpfs)
(allow mslg_app mslg_app_userfaultfd (anon_inode (ioctl read create)))
(neverallow base_typeattr_725_30_0 mslg_app_userfaultfd (anon_inode (ioctl read write create getattr setattr lock relabelfrom relabelto append map unlink link rename execute quotaon mounton audit_access open execmod watch watch_mount watch_sb watch_with_perm watch_reads)))
(neverallow mslg_app base_typeattr_726_30_0 (anon_inode (ioctl read write create getattr setattr lock relabelfrom relabelto append map unlink link rename execute quotaon mounton audit_access open execmod watch watch_mount watch_sb watch_with_perm watch_reads)))
(allow mslg_app appdomain_tmpfs_30_0 (file (read write getattr map execute)))
(neverallow base_typeattr_727_30_0 base_typeattr_725_30_0 (file (ioctl read write create setattr lock relabelfrom append unlink link rename open watch watch_mount watch_sb watch_with_perm watch_reads)))
(neverallow base_typeattr_728_30_0 mslg_app (file (ioctl read write create setattr lock relabelfrom append unlink link rename open watch watch_mount watch_sb watch_with_perm watch_reads)))
(neverallow base_typeattr_729_30_0 mslg_app (process (ptrace)))
(allow mslg_app app_data_file_30_0 (dir (write add_name search)))
(allow mslg_app app_data_file_30_0 (file (create open)))
(auditallow mslg_app app_data_file_30_0 (file (execute)))
(allow mslg_app system_linker_exec_30_0 (file (execute_no_trans)))
(allow mslg_app privapp_data_file_30_0 (lnk_file (ioctl read getattr lock map open watch watch_reads)))
(allow mslg_app app_data_file_30_0 (lnk_file (ioctl read write create getattr setattr lock append map unlink rename open watch watch_reads)))
(allow mslg_app app_data_file_30_0 (sock_file (ioctl read write create getattr setattr lock append map unlink rename open watch watch_reads)))
(allow mslg_app app_data_file_30_0 (fifo_file (ioctl read write create getattr setattr lock append map unlink rename open watch watch_reads)))
(allow mslg_app asec_apk_file_30_0 (file (ioctl read getattr lock map open watch watch_reads)))
(allow mslg_app asec_apk_file_30_0 (dir (ioctl read getattr lock open watch watch_reads search)))
(allow mslg_app asec_public_file_30_0 (file (execute)))
(allow mslg_app shell_data_file_30_0 (file (ioctl read getattr lock map open watch watch_reads)))
(allow mslg_app shell_data_file_30_0 (dir (ioctl read getattr lock open watch watch_reads search)))
(allow mslg_app trace_data_file_30_0 (file (read getattr)))
(allow mslg_app system_app_data_file_30_0 (file (read write getattr)))
(allow mslg_app media_rw_data_file_30_0 (dir (ioctl read write create getattr setattr lock rename open watch watch_reads add_name remove_name reparent search rmdir)))
(allow mslg_app media_rw_data_file_30_0 (file (ioctl read write create getattr setattr lock append map unlink rename open watch watch_reads)))
(allow mslg_app mnt_media_rw_file_30_0 (dir (search)))
(allow mslg_app servicemanager_30_0 (service_manager (list)))
(allow mslg_app audioserver_service_30_0 (service_manager (find)))
(allow mslg_app cameraserver_service_30_0 (service_manager (find)))
(allow mslg_app drmserver_service_30_0 (service_manager (find)))
(allow mslg_app mediaserver_service_30_0 (service_manager (find)))
(allow mslg_app mediaextractor_service_30_0 (service_manager (find)))
(allow mslg_app mediametrics_service_30_0 (service_manager (find)))
(allow mslg_app mediadrmserver_service_30_0 (service_manager (find)))
(allow mslg_app nfc_service_30_0 (service_manager (find)))
(allow mslg_app radio_service_30_0 (service_manager (find)))
(allow mslg_app app_api_service (service_manager (find)))
(allow mslg_app vr_manager_service_30_0 (service_manager (find)))
(allow mslg_app gpu_service_30_0 (service_manager (find)))
(allow mslg_app gpuservice_30_0 (binder (call transfer)))
(allow gpuservice_30_0 mslg_app (binder (transfer)))
(allow mslg_app gpuservice_30_0 (fd (use)))
(allow mslg_app self (process (ptrace)))
(allow mslg_app runas_app_30_0 (unix_stream_socket (connectto)))
(allow mslg_app mslgd (unix_stream_socket (connectto)))
(allow mslg_app runas_app_30_0 (process (sigchld)))
(allow mslg_app sysfs_hwrandom_30_0 (dir (search)))
(allow mslg_app sysfs_hwrandom_30_0 (file (ioctl read getattr lock map open watch watch_reads)))
(allow mslg_app preloads_media_file_30_0 (dir (ioctl read getattr lock open watch watch_reads search)))
(allow mslg_app preloads_media_file_30_0 (file (ioctl read getattr lock map open watch watch_reads)))
(allow mslg_app preloads_data_file_30_0 (dir (search)))
(allow mslg_app vendor_app_file_30_0 (dir (read getattr open search)))
(allow mslg_app vendor_app_file_30_0 (file (ioctl read getattr lock map execute open watch watch_reads)))
(allow mslg_app vendor_app_file_30_0 (lnk_file (read getattr open)))
(allow mslg_app traced_30_0 (fd (use)))
(allow mslg_app traced_tmpfs_30_0 (file (read write getattr map)))
(allow mslg_app traced_producer_socket_30_0 (sock_file (write)))
(allow mslg_app traced_30_0 (unix_stream_socket (connectto)))
(allow traced_30_0 mslg_app (fd (use)))
(allow traced_perf_30_0 mslg_app (file (ioctl read getattr lock map open watch watch_reads)))
(allow traced_perf_30_0 mslg_app (dir (ioctl read getattr lock open watch watch_reads search)))
(allow traced_perf_30_0 mslg_app (process (signal)))
(allow mslg_app traced_perf_socket_30_0 (sock_file (write)))
(allow mslg_app traced_perf_30_0 (unix_stream_socket (connectto)))
(allow traced_perf_30_0 mslg_app (fd (use)))
(allow mslg_app system_server_30_0 (udp_socket (read write getattr connect getopt setopt recvfrom sendto)))
(allow mslg_app rs_exec_30_0 (file (read getattr map execute open)))
(allow mslg_app rs_30_0 (process (transition)))
(allow rs_30_0 rs_exec_30_0 (file (read getattr map execute open entrypoint)))
(allow rs_30_0 mslg_app (process (sigchld)))
(dontaudit mslg_app rs_30_0 (process (noatsecure)))
(allow mslg_app rs_30_0 (process (siginh rlimitinh)))
(typetransition mslg_app rs_exec_30_0 process rs)
(dontaudit mslg_app net_dns_prop_30_0 (file (read)))
(dontaudit mslg_app proc_stat_30_0 (file (read)))
(dontaudit mslg_app proc_vmstat_30_0 (file (read)))
(dontaudit mslg_app proc_uptime_30_0 (file (read)))
(typetransition mslg_app devpts_30_0 chr_file mslg_app_devpts)
(allow mslg_app mslg_app_devpts (chr_file (ioctl read write getattr open)))
(allowx mslg_app mslg_app_devpts (ioctl chr_file (((range 0x5401 0x5404)) 0x540b ((range 0x540e 0x5411)) ((range 0x5413 0x5414)) ((range 0x5450 0x5451)))))
(neverallowx base_typeattr_185_30_0 mslg_app_devpts (ioctl chr_file (0x5412)))
(allow mslg_app simpleperf_30_0 (process (signal)))
(allow mslg_app system_app_service_30_0 (service_manager (find)))
(allow mslg_app vendor_hal_perf_hwservice (hwservice_manager (find)))
(allow mslg_app hal_mslgkeeper_hwservice (hwservice_manager (find)))
(allow mslg_app hal_mslgkeeper_default (binder (call transfer)))
(allow mslg_app procfs_memory (file (read)))
(allow mslg_app mslgd (unix_dgram_socket (sendto)))
(allow mslg_app procfs_memory (file (open)))
(allow mslg_app mslg_rootfs_file (sock_file (write create getattr setattr unlink rename)))
(allow mslg_app mslg_rootfs_file (file (ioctl read write create getattr setattr lock append map unlink link rename open watch)))
(allow mslg_app mslg_rootfs_file (fifo_file (read write create unlink open)))
(allow mslg_app mslg_rootfs_file (dir (read write create getattr setattr open watch add_name remove_name search rmdir)))
(allow mslg_app mslg_rootfs_file (lnk_file (read getattr)))
(allow mslg_app tmpfs_30_0 (sock_file (write create)))
(allow mslg_app tmpfs_30_0 (file (read open)))
(allow mslg_app app_api_service (service_manager (find)))
(allow mslg_app mslg_app (tcp_socket (read write)))
(allow mslg_app property_socket_30_0 (sock_file (write)))
(allow mslg_app init_30_0 (unix_stream_socket (connectto)))
(allow mslg_app system_prop_30_0 (property_service (set)))
(allow init_30_0 mslg_init_exec (file (read getattr map execute open)))
(allow init_30_0 mslg_init (process (transition)))
(allow mslg_init mslg_init_exec (file (read getattr map execute open entrypoint)))
(dontaudit init_30_0 mslg_init (process (noatsecure)))
(allow init_30_0 mslg_init (process (siginh rlimitinh)))
(typetransition init_30_0 mslg_init_exec process mslg_init)
(allow init_30_0 mslgd_exec (file (read getattr map execute open)))
(allow init_30_0 mslgd (process (transition)))
(allow mslgd mslgd_exec (file (read getattr map execute open entrypoint)))
(dontaudit init_30_0 mslgd (process (noatsecure)))
(allow init_30_0 mslgd (process (siginh rlimitinh)))
(typetransition init_30_0 mslgd_exec process mslgd)
(allow mslgd su_30_0 (unix_stream_socket (accept setopt)))
(allow mslgd mslg_rootfs_file (file (ioctl read write create getattr setattr lock append map unlink link rename open watch)))
(allow mslgd mslg_rootfs_file (fifo_file (read write create unlink open)))
(allow mslgd mslg_rootfs_file (dir (read write create getattr setattr rename open watch add_name remove_name search rmdir)))
(allow mslgd mslg_rootfs_file (lnk_file (read create getattr setattr link)))
(allow mslgd mslgd (netlink_kobject_uevent_socket (create getattr bind setopt)))
(allow mslgd devpts_30_0 (chr_file (ioctl read write getattr setattr append open)))
(allow mslgd devpts_30_0 (dir (read open)))
(allow mslgd proc_filesystems_30_0 (file (read getattr open)))
(allow mslgd proc_30_0 (file (ioctl read getattr lock map open watch watch_reads)))
(allow mslgd sysfs_30_0 (dir (read)))
(allow mslgd system_app_30_0 (process (signull)))
(allow mslgd untrusted_app_30_0 (process (signull)))
(allow mslgd priv_app_30_0 (process (signull)))
(allow mslgd mslgd (netlink_audit_socket (read write create nlmsg_relay)))
(allow mslgd mslgd (key (write search)))
(allow mslgd kernel_30_0 (key (link)))
(allow mslgd kernel_30_0 (dir (search)))
(allow mslgd kernel_30_0 (file (read open)))
(allow mslgd hal_graphics_composer_default (process (signull)))
(allow mslgd init_30_0 (dir (search)))
(allow mslgd init_30_0 (file (read getattr open)))
(allow mslgd init_30_0 (lnk_file (read)))
(allow mslgd fuse_30_0 (dir (ioctl read write create getattr setattr lock rename open watch watch_reads add_name remove_name reparent search rmdir)))
(allow mslgd fuse_30_0 (file (ioctl read write create getattr setattr lock append map unlink rename open watch watch_reads)))
(allow mslgd mslg_rootfs_file (sock_file (write create getattr setattr unlink rename)))
(allow mslgd mslgd (capability (chown fowner fsetid kill setgid setuid setpcap sys_chroot sys_admin sys_nice sys_resource audit_write)))
(allow mslgd mslgd (netlink_route_socket (read write create getattr bind setopt nlmsg_read nlmsg_readpriv)))
(allow mslgd mslgd (tcp_socket (read write create bind connect listen accept setopt name_connect)))
(allow mslgd mslgd (tcp_socket (ioctl shutdown)))
(allow mslgd port_30_0 (tcp_socket (name_bind name_connect)))
(allow mslgd node_30_0 (tcp_socket (node_bind)))
(allow mslgd mslgd (udp_socket (read write create getattr connect setopt)))
(allow mslgd untrusted_app_30 (unix_dgram_socket (sendto)))
(allow mslgd tmpfs_30_0 (chr_file (ioctl read write getattr setattr open)))
(allow mslgd tmpfs_30_0 (dir (ioctl read write create getattr setattr lock rename open watch watch_reads add_name remove_name reparent search rmdir)))
(allow mslgd tmpfs_30_0 (file (ioctl read write create getattr setattr lock append map unlink rename open watch watch_reads)))
(allow mslgd tmpfs_30_0 (lnk_file (read)))
(allow mslgd mslgd_exec (file (ioctl read getattr lock map execute open execute_no_trans entrypoint)))
(allow mslgd mslgd_exec (dir (read getattr add_name remove_name search)))
(allow mslgd mslgd_exec (lnk_file (read getattr)))
(allow mslgd mslgd (process (execmem)))
(allow mslgd mslg_app (unix_dgram_socket (sendto)))
(allow mslgd mslgd (process (setexec execmem)))
(allow mslgd kernel_30_0 (file (getattr)))
(allow mslgd kernel_30_0 (lnk_file (read)))
(allow mslgd platform_app_30_0 (process (signull)))
(allow mslgd su_30_0 (fd (use)))
(allow mslgd su_30_0 (process (transition noatsecure siginh rlimitinh)))
(allow mslgd vendor_init_30_0 (dir (search)))
(allow mslgd vendor_init_30_0 (file (read open)))
(allow mslgd vendor_toolbox_exec_30_0 (file (execute_no_trans)))
(allow mslgd vendor_file_30_0 (file (read)))
(allow mslgd block_device_30_0 (dir (ioctl read getattr lock open watch watch_reads search)))
(allow mslgd loop_device_30_0 (blk_file (ioctl read write getattr lock append map open watch watch_reads)))
(allowx mslgd loop_device_30_0 (ioctl blk_file (0x1261)))
(allowx mslgd loop_device_30_0 (ioctl blk_file (((range 0x4c00 0x4c01)) ((range 0x4c04 0x4c05)) ((range 0x4c08 0x4c0a)))))
(allow kernel_30_0 mslgd (fd (use)))
(allow mslgd vendor_mslg_prop (file (read getattr map open)))
(allow mslgd property_socket_30_0 (sock_file (write)))
(allow mslgd init_30_0 (unix_stream_socket (connectto)))
(allow mslgd vendor_mslg_prop (property_service (set)))
(allow mslgd vendor_mslg_prop (file (read getattr map open)))
(allow kernel_30_0 vendor_file_30_0 (file (read)))
(allow mslgd loop_control_device_30_0 (chr_file (ioctl read write open)))
(allow vendor_init_30_0 vendor_mslg_prop (property_service (set)))
(allow vendor_init_30_0 vendor_mslg_prop (file (read getattr map open)))
(allow vendor_init_30_0 vendor_mslg_prop (file (read getattr map open)))
(allow mslgd hwservicemanager_prop_30_0 (file (read getattr map open)))
(allow hwservicemanager_30_0 mslgd (binder (transfer)))
(allow mslgd hwservicemanager_30_0 (binder (call)))
(allow mslgd hal_mslgkeeper_default (binder (call)))
(allow mslgd hal_mslgkeeper_hwservice (hwservice_manager (find)))
(allow platform_app_30_0 hal_mslgkeeper_hwservice (hwservice_manager (find)))
(allow platform_app_30_0 hal_mslgkeeper_default (binder (call transfer)))
(allow platform_app_30_0 mslg_rootfs_file (dir (ioctl read getattr lock open watch watch_reads search)))
(allow platform_app_30_0 mslg_rootfs_file (file (ioctl read getattr lock map open watch watch_reads)))
(allow platform_app_30_0 mslg_rootfs_file (sock_file (write)))
(allow system_app_30_0 hal_mslgkeeper_hwservice (hwservice_manager (find)))
(allow system_app_30_0 hal_mslgkeeper_default (binder (call)))
(allow system_server_30_0 hal_mslgkeeper_default (binder (call)))
(allow system_server_30_0 hal_mslgkeeper_hwservice (hwservice_manager (find)))
(allow untrusted_app_30 mslg_rootfs_file (dir (ioctl read write getattr lock open watch watch_reads add_name remove_name search)))
(allow untrusted_app_30 mslgd (unix_dgram_socket (sendto)))
(allow untrusted_app_30 mslgd (unix_stream_socket (connectto)))
(allow untrusted_app_30 zygote_30_0 (unix_stream_socket (getopt)))
(allow untrusted_app_30 procfs_memory (file (open)))
(allow untrusted_app_30 mslg_rootfs_file (file (ioctl read write getattr lock append map open watch watch_reads)))
(allow untrusted_app_30 mslg_rootfs_file (sock_file (write create getattr setattr unlink rename)))

新增行
(typetransition mslg_app mslg_app anon_inode "[userfaultfd]" mslg_app_userfaultfd)

新增行
(typeattribute base_typeattr_729_30_0)
(typeattributeset base_typeattr_729_30_0 ((and (domain) ((not (crash_dump_30_0 runas_app_30_0 simpleperf_30_0 mslg_app))))))
(typeattribute base_typeattr_728_30_0)
(typeattributeset base_typeattr_728_30_0 ((and (appdomain) ((not (runas_app_30_0 shell_30_0 simpleperf_30_0 mslg_app))))))
(typeattribute base_typeattr_727_30_0)
(typeattributeset base_typeattr_727_30_0 ((and (mslg_app) ((not (runas_app_30_0 shell_30_0 simpleperf_30_0))))))
(typeattribute base_typeattr_726_30_0)
(typeattributeset base_typeattr_726_30_0 ((not (mslg_app_userfaultfd))))
(typeattribute base_typeattr_725_30_0)
(typeattributeset base_typeattr_725_30_0 ((and (domain) ((not (mslg_app))))))
(typeattribute base_typeattr_724_30_0)
(typeattributeset base_typeattr_724_30_0 ((and (domain) ((not (hal_mslgkeeper_server))))))
```

## 重新打包mi_ext、odm、system、system_ext、vendor、product分区，未完成
先用make_ext4fs或者e2fsdroid+mke2fs打包为raw image，  
这里的目标是打包成super.img，vab机器一般是线刷用fastboot刷进super分区，卡刷用zstd压缩后在recovery里解压到super分区
解包打包偷懒就找个安卓工具箱，米欧、dna、多幸运之类的，直接一键打包

## avb验证文件，直接替换成关闭avb的文件  
vbmeta.img
