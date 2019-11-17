---
layout: post
title:  "使用MacOS制作Windows 10系统的USB启动程序"
date:   2019-11-17 23:57:54
categories: diskutil
tags: 制作U盘
---

* content
{:toc}

当要给新买的组装台式机安装系统时，身边又没有Windows系统的电脑，只有Mac电脑，这时怎么利害Mac电脑来制作U盘启动程序给机器安装系统，
其他方法很简历，比在Windows电脑上还简单


# 使用MacOS制作Windows 10系统的USB启动程序

## 所需工具及材料

* MacOS电脑（我的是Macbook pro, 版本是`Catalina 10.15`）
* 16GB的U盘
* Windows 10的ISO镜像文件（注意有文件大小大于4GB和小于4GB是有区别的）

## 制作步骤

### 一、插入U盘到Mac电脑
### 二、挂载ISO镜像文件（双击它就好了）
### 三、查看已经挂载的文件，输入以下命令

```shell
$ diskutil list
```

会出现以下内容
```shell
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *121.3 GB   disk0
   1:                        EFI EFI                     314.6 MB   disk0s1
   2:                 Apple_APFS Container disk1         121.0 GB   disk0s2

/dev/disk1 (synthesized):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      APFS Container Scheme -                      +121.0 GB   disk1
                                 Physical Store disk0s2
   1:                APFS Volume Macintosh HD - 数据     102.0 GB   disk1s1
   2:                APFS Volume Preboot                 79.5 MB    disk1s2
   3:                APFS Volume Recovery                528.9 MB   disk1s3
   4:                APFS Volume VM                      3.2 GB     disk1s4
   5:                APFS Volume Macintosh HD            10.6 GB    disk1s5

/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *500.1 GB   disk2
   1:               Windows_NTFS Seagate Expansion Drive 500.1 GB   disk2s1

/dev/disk3 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *15.7 GB    disk3
   1:             Windows_FAT_32 GSP1RMCULFR             15.7 GB    disk3s4

/dev/disk4 (disk image):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:                            CPBA_X64FRE_ZH-CN_DV9  +5.3 GB     disk4
```

其中的`/dev/disk4`就是我的U盘，`/dev/disk4`就是被我挂载的ISO镜像文件，记住这个名字`CPBA_X64FRE_ZH-CN_DV9`下面要用到

这里有个要注意的地方，现在的Windows系统都比较大，像我这个就已经超过4GB，但是一般情况下我们在制作U盘启动程序的时候，它会默认把U盘格式化为FAT32, 
因为FAT32的文件系统不支持单个文件超过4GB，所以在把IOS镜像里的`install.wim`文件复制到U盘时会失败， 所以会有以下几种方法解决（ISO镜像文件没有超过4GB可以跳过这一步）

* 将U盘格式化为NTFS或ExFAT
* 将`install.wim`分割成两个文件
* 将U盘分成两个区

我这里用到简单的方法就是直接把U盘格式化为ExFAT的格式，其他两种方法可以去网上搜索，一盘在Windows上操作比较方便，我这里是Mac就不演示了

### 四、格式化U盘

* ISO文件大于4GB
  
```shell
$ diskutil eraseDisk ExFAT "WINDOWS10" disk3
```

* ISO文件小于4GB
  
```sheall
$ diskutil eraseDisk FAT32 "WINDOWS10" MBR disk3
```

其中的`"WINDOWS10"`和`disk3`指的是把我的U盘格式化为EXFAT或FAT32，并把U盘名称改成`WINDOWS10`

### 五、拷贝文件

```shell
$ cp -rp /Volumes/CPBA_X64FRE_ZH-CN_DV9/* /Volumes/WINDOWS10/
```

其中`CPBA_X64FRE_ZH-CN_DV9`就是ISO镜像文件的挂载后的名称，`WINDOWS10`是U盘的名称，这里的意思是把ISO镜像文件里的所有文件拷贝到U盘里，
这里要等一段时间，因为ISO文件很大

### 六、拷贝完成后没有任务报错，就说明制作完成，拿到机器上去装系统吧！