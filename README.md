# 红米 10X PRO 移植红米Note11 Pro安卓13版本HyperOS记录
资源来源于网络，仅供交流学习，不得用做任何商业用途，不提供任何技术支持，请在下载后24小时内删除  
基于miui_BOMB_V13.0.7.0，移植文件来源于miui_PISSARRO_OS1.0.2.0   
本文仅记录一下修改内容，具体修改行以及内容以实际文件对比结果为准

## mi_ext分区修改，整体上照搬pissarro，但要注意以下部分
build.prop修改机型代号，这里这个代号是miui ota更新服务器用来识别推送更新用的，你都刷第三方rom了这个就不重要了，除非你能用到那个服务器推送更新  
把这个东西改掉的好处就是可以屏蔽更新，不会收到移植的那个机型的更新，导致用户误升级变砖  
修改版本号为TJLCNXM  
mi_ext\etc\build.prop
```
ro.product.mod_device=bomb
ro.mi.os.version.incremental=OS1.0.2.0.TJLCNXM
```
## product分区修改，整体上照搬pissarro，但要注意以下部分
product\app  
删除无法使用的人脸识别 MiuiBiometric33148  
目前没有修复办法  

按需精简  
快应用服务引擎  
product\app\HybridPlatform  
智能服务  
product\app\MSA  

data-app可卸载的预装app，其中不少app都是可以在应用商店里重新安装的，  
product\data-app\  
百度输入法小米版  
product\data-app\BaiduIME  
讯飞输入法小米版  
product\data-app\com.iflytek.inputmethod.miui  
小米运动健康  
product\data-app\Health  
小米云盘  
product\data-app\MIUIMiDrive  
内容中心  
product\data-app\MIUINewHome_Removable  
小米社区  
product\data-app\MIUIVipAccount  
全球上网  
product\data-app\MIUIVirtualSim  
米家  
product\data-app\SmartHome  

一堆有用没用的功能开关、剃刀计划：可卸载系统app名单  
product\etc\device_features\bomb.xml
```
    <bool name="support_cabc">true</bool>
    <!--whether the device supports voip record -->
    <bool name="support_voip_record">true</bool>

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
        <item>/system/data-app/MIGalleryLockScreenGlobal/MIGalleryLockScreenGlobal.apk</item>
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

    <!--These region support face unlock-->
    <string-array name="support_face_unlock_region_global">
    <item>ALL</item>
    </string-array>

    <bool name="support_multi_face_input_global">true</bool>

    <!-- whether support Macro Keyboard -->
    <bool name="support_mi_game_macro">true</bool>
    <!-- gamebooster mi time -->
    <bool name="support_game_mi_time">true</bool>

    <!-- open game dim featrue -->
    <bool name="support_game_dim">true</bool>
    <!-- open dim time from cloud featrue -->
    <bool name="support_cloud_dim">true</bool>
    <!-- whether remove vgpaper -->
    <bool name="remove_vgpaper">true</bool>

    <string-array name="dynamic_partition_list">
        <item>/system</item>
        <item>/system_ext</item>
        <item>/product</item>
        <item>/vendor</item>
        <item>/mi_ext</item>
    </string-array>
    <string-array name="log_partition_list">
        <item>rescue</item>
        <item>oops</item>
        <item>minidump</item>
        <item>rawdump</item>
        <item>crash_history</item>
        <item>expdb</item>
    </string-array>
    <string name="partition_name_path">/dev/block/by-name/</string>
```
屏幕亮度曲线？  
product\etc\displayconfig\display_id_0.xml
```
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<!-- MIUI ADD: Adapt_MIUIAdaptFrame -->

<displayConfiguration>
    <!-- Global Brightness-Nit mapping -->
    <screenBrightnessMap>
        <point>
            <value>0.003421310</value>
            <nits>2.0</nits>
        </point>
        <point>
            <value>1.0</value>
            <nits>500.0</nits>
        </point>
    </screenBrightnessMap>
    <ambientLightHorizonLong>3000</ambientLightHorizonLong>
    <ambientLightHorizonShort>1000</ambientLightHorizonShort>
</displayConfiguration>
<!-- END Adapt_MIUIAdaptFrame -->
```
build.prop修改编译日期和指纹、版本号  
product\etc\build.prop
```
ro.product.product.name=miproduct_bomb_cn
ro.product.build.fingerprint=Xiaomi/miproduct_bomb_cn/missi:13/TP1A.220624.014/V816.0.2.0.TJLCNXM:user/release-keys
ro.product.build.version.incremental=V816.0.2.0.TJLCNXM

#支持存储卡
ro.build.characteristics=default

#删除ro.product.ab_ota_partitions=product这一行
#ro.product.ab_ota_partitions=product

ro.product.mod_device=bomb
```
overlay保留10x本身设备的apk  
product\overlay\DevicesAndroidOverlay.apk  
product\overlay\DevicesOverlay.apk  
product\overlay\AospFrameworkResOverlay.apk  
product\overlay\MiuiFrameworkResOverlay.apk  

修复息屏显示功能：反编译DevicesAndroidOverlay，修改资源文件strings.xml  
把com.miui.aod/com.miui.aod.doze.DozeService改成com.android.systemui/com.android.systemui.doze.DozeService  

product\pangu\system\app\  
删除无法使用的系统自带硬件检测、NFC功能   
MiuiCit  
NQNfcNci  
目前没有修复办法    
删除配套lib文件  
product\pangu\system\lib\libsensor_cal.so  
product\pangu\system\lib\libsensor_calJNI.so  
product\pangu\system\lib\vendor.xiaomi.hardware.citsensorservice@1.0.so  
product\pangu\system\lib\vendor.xiaomi.hardware.citsensorservice@1.1.so  

product\pangu\system\lib64\libsensor_cal.so  
product\pangu\system\lib64\libsensor_calJNI.so  
product\pangu\system\lib64\vendor.xiaomi.hardware.citsensorservice@1.0.so  
product\pangu\system\lib64\vendor.xiaomi.hardware.citsensorservice@1.1.so  

## system分区修改，整体上照搬pissarro，但要注意以下部分
build.prop修改编译日期和指纹、版本号  
system\system\system_ext\etc\build.prop
```
ro.system.build.fingerprint=Xiaomi/missi/missi:13/TP1A.220624.014/V816.0.2.0.TJLCNXM:user/release-keys
ro.system.build.version.incremental=V816.0.2.0.TJLCNXM
ro.build.version.incremental=V816.0.2.0.TJLCNXM
ro.build.description=missi_phoneext4_cn-user 13 TP1A.220624.014 V816.0.2.0.TJLCNXM release-keys
```
键位文件，修复音量键无效问题  
system\system\usr\keylayout  
保留10x键位文件  
ACCDET.kl  
mtk-kpd.kl  
## system_ext分区修改，整体上照搬pissarro，但要注意以下部分
build.prop修改编译日期和指纹、版本号  
可选修改  
system\system\build.prop
```
ro.system_ext.build.fingerprint=Xiaomi/missi/missi:13/TP1A.220624.014/V816.0.2.0.TJLCNXM:user/release-keys
ro.system_ext.build.version.incremental=V816.0.2.0.TJLCNXM
```
## vendor分区修改，整体上保留10x，但要注意以下部分
vendor\app  
删除Joyose空文件夹  

miui14新增misys服务相关文件  
vendor\bin\hw\vendor.xiaomi.hardware.misys@4.0-service  
vendor\etc\init\vendor.xiaomi.hardware.misys@4.0-service.rc  

保留10x振动马达相关文件，从底包的system\system\etc\excluded-input-devices.xml复制过来  
vendor\etc\excluded-input-devices.xml

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
添加内容第一段大部分为miui13本身就有的，在最后两行中间追加了内容，   
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
可改可不改，因为sepolicy的大头在找个文件里  
precompiled_sepolicy  
这玩意只能从源码编译出来，不能二次编辑，所以上面其实是改了个寂寞

miui14新增misys服务相关文件  
vendor\etc\vintf\manifest\vendor.xiaomi.hardware.misys@4.0.xml  
vendor\lib\hw\vendor.xiaomi.hardware.misys@4.0-impl.so  
vendor\lib\vendor.xiaomi.hardware.misys@4.0.so  
vendor\lib64\hw\vendor.xiaomi.hardware.misys@4.0-impl.so  
vendor\lib64\vendor.xiaomi.hardware.misys@4.0.so  

相机加速配置文件  
vendor\odm\etc\camera\camerabooster.json
```
{
    "version": "1.0",
    "product": "bomb",
    "threshold": {
            "lowerAdj_freeMem_threshold": 307200,
            "update_state_delay_ms": 5000,
            "skip_task":1      
        },
    "support": {
        "mms_camcpt_enable": true,
        "adj_swap_support": true,
        "perceptible_support": true
        },
    "configs_6Gmem": [
        {
            "name": "white_list",
            "config": [
                "com.android.camera",
                "com.miui.home",
                "com.tencent.mm",
                "com.tencent.mm:push",
                "com.tencent.mobileqq:MSF",
                "com.tencent.qqmusic:QQPlayerService",
                "com.ss.android.lark.kami:wschannel",
                "com.xiaomi.mi_connect_service",
                "com.phonetest.application:CameraMemoryWatcher"
            ]
        }
    ],
    "threshold_6Gmem": {
            "lowerAdj_freeMem_threshold": 307200,
            "cam_boost_threshold": 2621440,
            "update_state_delay_ms": 5000,
            "skip_task_lower":1,
            "kill_tag": 7,
            "lowAdj_threshold": 0
        }
}
```
build.prop修改编译日期和指纹、版本号  
vendor\odm\etc\build.prop
```
ro.product.odm.cert=M2004J7BC

ro.odm.build.date=Wed Jul  5 20:16:31 CST 2023
ro.odm.build.date.utc=1688559391
ro.odm.build.fingerprint=Redmi/bomb/bomb:12/SP1A.210812.016/V14.0.5.0.SJLCNXM:user/release-keys
ro.odm.build.version.incremental=V14.0.5.0.SJLCNXM
```
build.prop修改编译日期和指纹、版本号  
vendor\odm_dlkm\etc\build.prop
```
ro.odm_dlkm.build.date=Wed Jul  5 20:16:31 CST 2023
ro.odm_dlkm.build.date.utc=1688559391
ro.odm_dlkm.build.fingerprint=Redmi/bomb/bomb:12/SP1A.210812.016/V14.0.5.0.SJLCNXM:user/release-keys
ro.odm_dlkm.build.version.incremental=V14.0.5.0.SJLCNXM
```
build.prop修改编译日期和指纹、版本号  
vendor\vendor_dlkm\etc\build.prop
```
ro.vendor_dlkm.build.date=Wed Jul  5 20:16:31 CST 2023
ro.vendor_dlkm.build.date.utc=1688559391
ro.vendor_dlkm.build.fingerprint=Redmi/bomb/bomb:12/SP1A.210812.016/V14.0.5.0.SJLCNXM:user/release-keys
ro.vendor_dlkm.build.version.incremental=V14.0.5.0.SJLCNXM
```
build.prop修改编译日期和指纹、版本号  
vendor\build.prop
```
ro.vendor.build.date=Wed Jul  5 20:16:31 CST 2023
ro.vendor.build.date.utc=1688559391
ro.vendor.build.fingerprint=Redmi/bomb/bomb:12/SP1A.210812.016/V14.0.5.0.SJLCNXM:user/release-keys
ro.vendor.build.version.incremental=V14.0.5.0.SJLCNXM

# 机型代号
ro.product.vendor.cert=M2004J7BC

# 玄学代码，提高显示流畅性，分辨率调整
#latch unsignaled buffer to improve display fluency
debug.sf.latch_unsignaled=1
#disable backpressure to improve display fluency
debug.sf.disable_backpressure=1
#default resolution and density
persist.sys.miui_resolution=1080,2400,440

# 其他属性
ro.rom.zone=1
ro.product.mod_device=bomb
ro.vendor.mtk_aod_support=1

# 游戏加载加速
debug.game.video.support=true
debug.game.video.speed=true
```
删除恢复官方recovery功能的相关文件  
vendor\bin\install-recovery.sh  
vendor\etc\init\vendor_flash_recovery.rc  
vendor\recovery-from-boot.p  

修改fstab，添加mi_ext、system_ext分区，添加overlay相关代码  
vendor\etc\fstab.emmc  
vendor\etc\fstab.mt6873  
```
system_ext /system_ext ext4 ro wait,,logical,first_stage_mount

mi_ext /mnt/vendor/mi_ext ext4 ro wait,,logical,first_stage_mount,nofail

/mnt/vendor/mi_ext /mi_ext ext4 ro,bind wait,nofail
overlay /product/overlay overlay ro,lowerdir=/mnt/vendor/mi_ext/product/overlay/:/product/overlay check,nofail
overlay /product/app overlay ro,lowerdir=/mnt/vendor/mi_ext/product/app/:/product/app check,nofail
overlay /product/priv-app overlay ro,lowerdir=/mnt/vendor/mi_ext/product/priv-app/:/product/priv-app check,nofail
overlay /product/lib overlay ro,lowerdir=/mnt/vendor/mi_ext/product/lib/:/product/lib check,nofail
overlay /product/lib64 overlay ro,lowerdir=/mnt/vendor/mi_ext/product/lib64/:/product/lib64 check,nofail
overlay /product/bin overlay ro,lowerdir=/mnt/vendor/mi_ext/product/bin/:/product/bin check,nofail
overlay /product/framework overlay ro,lowerdir=/mnt/vendor/mi_ext/product/framework/:/product/framework check,nofail
overlay /product/media overlay ro,lowerdir=/mnt/vendor/mi_ext/product/media/:/product/media check,nofail
overlay /product/opcust overlay ro,lowerdir=/mnt/vendor/mi_ext/product/opcust/:/product/opcust check,nofail
overlay /product/data-app overlay ro,lowerdir=/mnt/vendor/mi_ext/product/data-app/:/product/data-app check,nofail
overlay /product/etc/sysconfig overlay ro,lowerdir=/mnt/vendor/mi_ext/product/etc/sysconfig/:/product/etc/sysconfig check,nofail
overlay /product/etc/permissions overlay ro,lowerdir=/mnt/vendor/mi_ext/product/etc/permissions/:/product/etc/permissions check,nofail
overlay /system/app overlay ro,lowerdir=/mnt/vendor/mi_ext/system/app/:/product/pangu/system/app/:/system/app check,nofail
overlay /system/priv-app overlay ro,lowerdir=/mnt/vendor/mi_ext/system/priv-app/:/product/pangu/system/priv-app/:/system/priv-app check,nofail
overlay /system/framework overlay ro,lowerdir=/product/pangu/system/framework/:/system/framework check,nofail
overlay /system/etc/sysconfig overlay ro,lowerdir=/mnt/vendor/mi_ext/system/etc/sysconfig/:/system/etc/sysconfig check,nofail
overlay /system/etc/permissions overlay ro,lowerdir=/mnt/vendor/mi_ext/system/etc/permissions/:/product/pangu/system/etc/permissions/:/system/etc/permissions check,nofail
overlay /system/lib overlay ro,lowerdir=/product/pangu/system/lib/:/system/lib check,nofail
overlay /system/lib64 overlay ro,lowerdir=/product/pangu/system/lib64/:/system/lib64 check,nofail
overlay /product/usr overlay ro,lowerdir=/mnt/vendor/mi_ext/product/usr:/product/usr check,nofail
overlay /product/etc/precust_theme overlay ro,lowerdir=/mnt/vendor/mi_ext/product/etc/precust_theme:/product/etc/precust_theme check,nofail
overlay /product/etc/preferred-apps overlay ro,lowerdir=/mnt/vendor/mi_ext/product/etc/preferred-apps:/product/etc/preferred-apps check,nofail
overlay /product/etc/security overlay ro,lowerdir=/mnt/vendor/mi_ext/product/etc/security:/product/etc/security check,nofail
overlay /system_ext/etc/permissions overlay ro,lowerdir=/mnt/vendor/mi_ext/system_ext/etc/permissions:/system_ext/etc/permissions check,nofail
```
可选关闭avb验证，修改fstab文件去除avb代码  
vendor\etc\fstab.emmc  
vendor\etc\ffstab.mt6873  
把system那一行的flags从`,avb_keys=`开始把后面的内容全删除，所有`,avb=vbmeta_system`删除，所有`,avb=vbmeta`删除，所有`,avb`删除  
## boot分区
ramdisk\fstab.emmc  
ramdisk\fstab.mt6873  
照搬vendor\etc\fstab.emmc和fstab.mt6873  

使用https://github.com/R0rt1z2/mtk-bpf-patcher 工具，修补kernel文件，修复bpf问题导致的无法开机  
## 重新打包mi_ext、system、system_ext、vendor、product分区
先用make_ext4fs或者e2fsdroid+mke2fs打包为raw image，  
使用img2simg工具把raw image转换为sparse image，  
使用img2sdat把sparse image转换为dat格式，  
使用brotli把dat格式转换为new.dat.br格式，  
就可以替换进zip卡刷包里了  
解包打包偷懒就找个安卓工具箱，米欧、dna、多幸运之类的，直接一键打包 

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

#加上mi_ext、system_ext分区
add mi_ext main
# Add partition system_ext to group main
add system_ext main

# Grow partition mi_ext from 0 to rawimg字节大小
resize mi_ext rawimg字节大小
# Grow partition system_ext from 0 to rawimg字节大小
resize system_ext rawimg字节大小
```
## avb验证文件，直接替换成关闭avb的文件  
vbmeta.img
