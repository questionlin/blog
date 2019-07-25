---
title: 在MAC上制作Win10启动盘
date: 2019-07-25 10:32:59
tags: 
id: 1564022016
---
```
$ diskutil list
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *251.0 GB   disk0
   1:                        EFI EFI                     209.7 MB   disk0s1
   2:                 Apple_APFS Container disk1         250.8 GB   disk0s2

/dev/disk1 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +250.8 GB   disk1
                                 Physical Store disk0s2
   1:                APFS Volume Macintosh HD            158.6 GB   disk1s1
   2:                APFS Volume Preboot                 45.8 MB    disk1s2
   3:                APFS Volume Recovery                509.8 MB   disk1s3
   4:                APFS Volume VM                      3.2 GB     disk1s4

/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *8.0 GB     disk2
   1:               Windows_NTFS 未命名                  8.0 GB     disk2s1

```
从大小我们知道u盘的名称是 disk2

```
diskutil eraseDisk ExFAT "WINDOWS10" MBRFormat disk2
```
这里比如指定 ExFAT 格式，因为 install.wim 文件大于 4GB，其他教材用的 MS-DOS(FAT32) 格式不能保存

```
$ hdiutil mount cn_windows_10_enterprise_ltsc_2019_x64_dvd_9c09ff24.iso
/dev/disk3          	                               	/Volumes/CES_X64FREV_ZH-CN_DV5
```
把 windows 镜像装载到 /Volumes/CES_X64FREV_ZH-CN_DV5 盘符

```
cp -rp /Volumes/CES_X64FREV_ZH-CN_DV5/* /Volumes/WINDOWS10/
```