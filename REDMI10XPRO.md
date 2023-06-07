# 红米 10X PRO 移植MIUI 14记录
资源来源于网络，仅供交流学习，不得用做任何商业用途，不提供任何技术支持，请在下载后24小时内删除  
基于miui_BOMB_V13.0.5.0，移植文件来源于miui_CEZANNE_V14.0.1.0，卡刷包解包就行了，  
思路就是对比miui_CEZANNE_V13.0.5.0与miui_CEZANNE_V14.0.1.0，取出变更了的部分，未变化的文件仍然保留10x原本的文件,  
由于project treble这个东西，厂商把大部分机器直接硬件相关的文件放进了单独的vendor分区，使得product和system分区在不同机器上有了很大程度的通用性，  
再加上MIUI现在很多资源都是集成在一个文件里，通过检测机型代码来切换功能，移植就很方便了  
同处理器同系统版本机型非常相近，但还是有需要修改的部分  
本文仅记录一下修改内容，具体修改行以及内容以实际文件对比结果为准

## product分区修改，整体上照搬k30u，但要注意以下部分

保留10x特有刘海  
product\app\NotchOverlay\NotchOverlay.apk

增加动态分区列表和记录分区列表，其中第二部分记录分区为miui14新增，由于内核没有实际升级，所以这段感觉上无效  
product\etc\device_features\bomb.xml
```
    <string-array name="dynamic_partition_list">
        <item>/system</item>
        <item>/product</item>
        <item>/vendor</item>
    </string-array>
    <string-array name="log_partition_list">
        <item>rescue</item>
        <item>oops</item>
        <item>minidump</item>
        <item>rawdump</item>
        <item>crash_history</item>
        <item>expdb</item>
    </string-array>
```
build.prop修改编译日期和指纹、版本号  
product\etc\build.prop
```
ro.product.build.date=Tue Mar 14 08:22:24 CST 2023
ro.product.build.date.utc=1678753344
ro.product.build.fingerprint=Redmi/bomb/bomb:12/SP1A.210812.016/V14.0.1.0.SJNCNXM:user/release-keys
ro.product.build.version.incremental=V14.0.1.0.SJNCNXM
```
overlay升级以下apk  
product\overlay\AospFrameworkResOverlay.apk  
product\overlay\MccMncOverlay.apk  

overlay保留10x本身设备的apk  
product\overlay\DevicesAndroidOverlay.apk  
product\overlay\DevicesOverlay.apk  
product\overlay\MiuiBiometricResOverlay.apk  
product\overlay\MiuiFrameworkResOverlay.apk  

## system分区修改，整体上照搬k30u，但要注意以下部分

system\system\app  
去除k30u指纹识别
GFDelmarSetting  
保留10x指纹识别
goodix_sz  

删除data-app推广应用（番茄免费小说、抖音短视频）  
system\system\data-app\com.dragon.read_104  
system\system\data-app\com.ss.android.ugc.aweme_15  

miui14新增misys服务  
system\system\etc\permissions\vendor.xiaomi.hardware.misys-V4.0-java-permission.xml

保留10x震动设备相关文件  
system\system\etc\excluded-input-devices.xml

新增MiSansC字体  
system\system\fonts\MiSansC_3.005.ttf

删除system分区里k30u马达、震动服务相关文件  
system\system\framework\vendor.xiaomi.hardware.motor-V1.0-java.jar  
system\system\lib\vendor.xiaomi.hardware.motor@1.0.so  
system\system\lib\vendor.xiaomi.hardware.vibratorfeature@1.0.so  
system\system\lib64\vendor.xiaomi.hardware.motor@1.0.so  
system\system\lib64\vendor.xiaomi.hardware.vibratorfeature@1.0.so  

miui14新增misys服务相关文件  
system\system\framework\vendor.xiaomi.hardware.misys-V4.0-java.jar  
system\system\lib\vendor.xiaomi.hardware.misys@4.0.so  
system\system\lib64\vendor.xiaomi.hardware.misys@4.0.so

保留10x人脸识别相关lib，MiuiBiometric有通过链接使用到这三个lib  
system\system\lib64\libjni_faceunlock.so  
system\system\lib64\libjni_stfaceunlock_api.so  
system\system\lib64\libstfaceunlockocl.so

MIUI14默认主题、开机动画、壁纸、图标、资源文件，照搬note12pro  
system\system\media

保留10x本身的收音机apk  
system\system\system_ext\app\FM  

build.prop修改编译日期和指纹、版本号  
system\system\system_ext\etc\build.prop
```
ro.system_ext.build.date=Tue Mar 14 08:22:24 CST 2023
ro.system_ext.build.date.utc=1678753344
ro.system_ext.build.fingerprint=Redmi/bomb/bomb:12/SP1A.210812.016/V14.0.1.0.SJNCNXM:user/release-keys
ro.system_ext.build.version.incremental=V14.0.1.0.SJNCNXM
```
保留10x本身的收音机lib文件  
system\system\system_ext\lib64\libfmjni.so

键位文件由于还是安卓12整体没变  
system\system\usr\keylayout  
去除k30u键位文件  
ACCDET_cen.kl  
保留10x键位文件  
ACCDET.kl  

build.prop修改编译日期和指纹、版本号  
system\system\build.prop
```
ro.system.build.date=Tue Mar 14 08:22:24 CST 2023
ro.system.build.date.utc=1678753344
ro.system.build.fingerprint=Redmi/bomb/bomb:12/SP1A.210812.016/V14.0.1.0.SJNCNXM:user/release-keys
ro.system.build.version.incremental=V14.0.1.0.SJNCNXM
ro.build.version.incremental=V14.0.1.0.SJNCNXM
ro.build.version.security_patch=2023-02-01
ro.build.date=Tue Mar 14 08:22:24 CST 2023
ro.build.date.utc=1678753344
ro.build.description=bomb-user 12 SP1A.210812.016 V14.0.1.0.SJNCNXM release-keys
ro.miui.ui.version.code=14
ro.miui.ui.version.name=V140
ro.vendor.build.software.version=Android12_140
ro.miui.version.code_time=1678723200
```
## vendor分区修改，整体上保留10x，但要注意以下部分

miui14新增misys服务相关文件  
vendor\bin\hw\vendor.xiaomi.hardware.misys@4.0-service  
vendor\etc\init\vendor.xiaomi.hardware.misys@4.0-service.rc  

修改权限设置文件，增加misys服务  
以及之前有提到过的miui14新增的记录分区相关权限，由于内核没有实际升级，所以这段感觉上无效  
vendor\etc\selinux\vendor_file_contexts
```
/(vendor|system/vendor)/bin/hw/vendor.xiaomi.hardware.misys@4.0-service     u:object_r:hal_misys_default_exec:s0
/dev/block/by-name/cust                          u:object_r:cust_block_device:s0
/dev/block/by-name/oops                          u:object_r:oops_block_device:s0
/dev/block/by-name/rescue                        u:object_r:rescue_block_device:s0
/dev/block/by-name/logdump                          u:object_r:vendor_logdump_partition:s0
/dev/block/by-name/minidump                          u:object_r:minidump_partition:s0
```
修改se策略文件，  
第一段大部分为miui13本身就有的，在最后两行中间追加了内容，   
第二段和第三段是新增内容，  
主要是增加之前有提到过的miui14新增的记录分区相关，由于内核没有实际升级，所以这段感觉上无效  
vendor\etc\selinux\vendor_sepolicy.cil  
```
cam_cal_device ebr_device expdb_device fat_device logo_device loop-control_device mbr_device met_device misc_device misc2_device mtfreqhopping_device mtgpio_device mtk_kpd_device network_device nvram_device pmt_device preloader_device pro_info_device protect_f_device protect_s_device psaux_device ptyp_device recovery_device sec_ro_device seccfg_device tee_part_device snapshot_device tgt_device touch_device tpd_em_log_device ttyp_device uboot_device uibc_device usrdata_device zram0_device hwzram0_device RT_Monitor_device kick_powerkey_device agps_device mnld_device geo_device mdlog_device md32_device scp_device adsp_device audio_scp_device sspm_device etb_device MT_pmic_adc_cali_device mtk-adc-cali_device MT_pmic_cali_device otp_device otp_part_block_device qemu_pipe_device icusb_device nlop_device irtx_device pmic_ftm_device charger_ftm_device shf_device keyblock_device offloadservice_device ttyACM_device hrm_device lens_device nvdata_device mcf_ota_block_device nvcfg_device expdb_block_device misc2_block_device logo_block_device para_block_device tee_block_device seccfg_block_device secro_block_device preloader_block_device lk_block_device protect1_block_device protect2_block_device keystore_block_device oemkeystore_block_device sec1_block_device md1img_block_device md1dsp_block_device md1arm7_block_device md3img_block_device mmcblk1_block_device mmcblk1p1_block_device bootdevice_block_device odm_block_device oem_block_device vendor_block_device dtbo_block_device loader_ext_block_device spm_device persist_block_device md_block_device spmfw_block_device mcupmfw_block_device scp_block_device sspm_block_device dsp_block_device ppl_block_device nvcfg_block_device ancservice_device mbim_device audio_ipi_device cam_vpu_block_device boot_para_block_device mtk_dfrc_device vbmeta_block_device alarm_device mdp_device mrdump_device kb_block_device dkb_block_device sar_device mtk_radio_device dpm_block_device audio_dsp_block_device gz_block_device pi_img_device vpud_device vcu_device mml_pq_device hwmsensor_device msensor_device gsensor_device als_ps_device gyroscope_device barometer_device humidity_device biometric_device sensorlist_device hf_manager_device m_batch_misc_device m_als_misc_device m_ps_misc_device m_baro_misc_device m_hmdy_misc_device m_acc_misc_device m_mag_misc_device m_gyro_misc_device m_act_misc_device m_pedo_misc_device m_situ_misc_device m_step_c_misc_device m_fusion_misc_device m_bio_misc_device dri_device postinstall_block_device ccci_wifi_proxy_device teei_fp_device teei_client_device teei_config_device utr_tui_device teei_vfs_device teei_rpmb_device ut_keymaster_device nwkopt_device tx_device gdix_mt_wrapper_device gdix_thp_device mddp_device tkcore_admin_device tkcore_block_device mobicore_admin_device mobicore_user_device mobicore_tui_device rpmb_block_device rpmb_device fingerprint_device widevine_drv_device ccci_aud_device ccci_ccb_device ccci_mdmonitor_device hall_device motor_device ccci_vts_device sound_device oops_block_device rescue_block_device cust_block_device minidump_partition vendor_logdump_partition touchfeature_device aed_device ccci_mdl_device vendor_fingerprint_device gsort_block_device ffu_partition))

(type oops_block_device)
(roletype object_r oops_block_device)
(type rescue_block_device)
(roletype object_r rescue_block_device)
(type cust_block_device)
(roletype object_r cust_block_device)
(type minidump_partition)
(roletype object_r minidump_partition)
(type vendor_logdump_partition)
(roletype object_r vendor_logdump_partition)

(allow hal_misys_default block_device_31_0 (dir (ioctl read getattr lock open watch watch_reads search)))
(allow hal_misys_default super_block_device_31_0 (blk_file (ioctl read getattr lock map open watch watch_reads)))
(allow hal_misys_default oops_block_device (blk_file (ioctl read getattr lock map open watch watch_reads)))
(allow hal_misys_default rescue_block_device (blk_file (ioctl read getattr lock map open watch watch_reads)))
(allow hal_misys_default cust_block_device (blk_file (ioctl read getattr lock map open watch watch_reads)))
(allow hal_misys_default expdb_block_device (blk_file (ioctl read getattr lock map open watch watch_reads)))
```
miui14新增misys服务相关文件  
vendor\etc\vintf\manifest\vendor.xiaomi.hardware.misys@4.0.xml  
vendor\lib\hw\vendor.xiaomi.hardware.misys@4.0-impl.so  
vendor\lib\vendor.xiaomi.hardware.misys@4.0.so  
vendor\lib64\hw\vendor.xiaomi.hardware.misys@4.0-impl.so  
vendor\lib64\vendor.xiaomi.hardware.misys@4.0.so  

build.prop修改编译日期和指纹、版本号  
vendor\odm\etc\build.prop
```
ro.odm.build.date=Tue Mar 14 08:22:24 CST 2023
ro.odm.build.date.utc=1678753344
ro.odm.build.fingerprint=Redmi/bomb/bomb:12/SP1A.210812.016/V14.0.1.0.SJNCNXM:user/release-keys
ro.odm.build.version.incremental=V14.0.1.0.SJNCNXM
```
build.prop修改编译日期和指纹、版本号  
vendor\odm_dlkm\etc\build.prop
```
ro.odm_dlkm.build.date=Tue Mar 14 08:22:24 CST 2023
ro.odm_dlkm.build.date.utc=1678753344
ro.odm_dlkm.build.fingerprint=Redmi/bomb/bomb:12/SP1A.210812.016/V14.0.1.0.SJNCNXM:user/release-keys
ro.odm_dlkm.build.version.incremental=V14.0.1.0.SJNCNXM
```
build.prop修改编译日期和指纹、版本号  
vendor\vendor_dlkm\etc\build.prop
```
ro.vendor_dlkm.build.date=Tue Mar 14 08:22:24 CST 2023
ro.vendor_dlkm.build.date.utc=1678753344
ro.vendor_dlkm.build.fingerprint=Redmi/bomb/bomb:12/SP1A.210812.016/V14.0.1.0.SJNCNXM:user/release-keys
ro.vendor_dlkm.build.version.incremental=V14.0.1.0.SJNCNXM
```
build.prop修改编译日期和指纹、版本号  
vendor\build.prop
```
ro.vendor.build.date=Tue Mar 14 08:22:24 CST 2023
ro.vendor.build.date.utc=1678753344
ro.vendor.build.fingerprint=Redmi/bomb/bomb:12/SP1A.210812.016/V14.0.1.0.SJNCNXM:user/release-keys
ro.vendor.build.version.incremental=V14.0.1.0.SJNCNXM
```
## 重新打包system、vendor、product分区
先用make_ext4fs或者e2fsdroid+mke2fs打包为raw image，  
使用img2simg工具把raw image转换为sparse image，  
使用img2sdat把sparse image转换为dat格式，  
使用brotli把dat格式转换为new.dat.br格式，  
就可以替换进zip卡刷包里了  
解包打包偷懒就找个安卓工具箱，dna、多幸运之类的

## 动态分区配置列表
刷入卡刷包的时候会根据这个文件调整手机里动态分区的实际分布，根据上面打包得到的raw image文件大小，修改文件里对应的rawimg字节大小  
dynamic_partitions_op_list
```
# Grow partition system from 0 to rawimg字节大小
resize system rawimg字节大小
# Grow partition vendor from 0 to rawimg字节大小
resize vendor rawimg字节大小
# Grow partition product from 0 to rawimg字节大小
resize product rawimg字节大小
```
## avb验证文件，直接替换成关闭avb的文件  
vbmeta.img
