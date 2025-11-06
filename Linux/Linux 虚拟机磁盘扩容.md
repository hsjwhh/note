### Linux 虚拟机磁盘扩容（Esxi 6.7）

- Esxi 编辑设置，将硬盘 10G 修改为 20G；
- 重启服务器`reboot`；
- 查看硬盘，确认已生效
    ```shell
    [root@localhost ~]# fdisk -l

    Disk /dev/sda: 21.5 GB, 21474836480 bytes, 41943040 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk label type: dos
    Disk identifier: 0x000db710

    Device Boot      Start         End      Blocks   Id  System
    /dev/sda1   *        2048     2099199     1048576   83  Linux
    /dev/sda2         2099200    10485759     4193280   8e  Linux LVM
    /dev/sda3        10485760    20971519     5242880   8e  Linux LVM

    Disk /dev/mapper/cl-root: 9118 MB, 9118416896 bytes, 17809408 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes


    Disk /dev/mapper/cl-swap: 536 MB, 536870912 bytes, 1048576 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    ```
    **`Disk /dev/sda: 21.5 GB`**
- 扩展分区
    ```shell
    [root@localhost ~]# fdisk /dev/sda
    Welcome to fdisk (util-linux 2.23.2).

    Changes will remain in memory only, until you decide to write them.
    Be careful before using the write command.


    Command (m for help): n
    Partition type:
    p   primary (3 primary, 0 extended, 1 free)
    e   extended
    Select (default e): p
    Selected partition 4
    First sector (20971520-41943039, default 20971520): 
    Using default value 20971520
    Last sector, +sectors or +size{K,M,G} (20971520-41943039, default 41943039): 
    Using default value 41943039
    Partition 4 of type Linux and of size 10 GiB is set

    Command (m for help): t
    Partition number (1-4, default 4): 
    Hex code (type L to list all codes): L

    0  Empty           24  NEC DOS         81  Minix / old Lin bf  Solaris        
    1  FAT12           27  Hidden NTFS Win 82  Linux swap / So c1  DRDOS/sec (FAT-
    2  XENIX root      39  Plan 9          83  Linux           c4  DRDOS/sec (FAT-
    3  XENIX usr       3c  PartitionMagic  84  OS/2 hidden C:  c6  DRDOS/sec (FAT-
    4  FAT16 <32M      40  Venix 80286     85  Linux extended  c7  Syrinx         
    5  Extended        41  PPC PReP Boot   86  NTFS volume set da  Non-FS data    
    6  FAT16           42  SFS             87  NTFS volume set db  CP/M / CTOS / .
    7  HPFS/NTFS/exFAT 4d  QNX4.x          88  Linux plaintext de  Dell Utility   
    8  AIX             4e  QNX4.x 2nd part 8e  Linux LVM       df  BootIt         
    9  AIX bootable    4f  QNX4.x 3rd part 93  Amoeba          e1  DOS access     
    a  OS/2 Boot Manag 50  OnTrack DM      94  Amoeba BBT      e3  DOS R/O        
    b  W95 FAT32       51  OnTrack DM6 Aux 9f  BSD/OS          e4  SpeedStor      
    c  W95 FAT32 (LBA) 52  CP/M            a0  IBM Thinkpad hi eb  BeOS fs        
    e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux a5  FreeBSD         ee  GPT            
    f  W95 Ext'd (LBA) 54  OnTrackDM6      a6  OpenBSD         ef  EFI (FAT-12/16/
    10  OPUS            55  EZ-Drive        a7  NeXTSTEP        f0  Linux/PA-RISC b
    11  Hidden FAT12    56  Golden Bow      a8  Darwin UFS      f1  SpeedStor      
    12  Compaq diagnost 5c  Priam Edisk     a9  NetBSD          f4  SpeedStor      
    14  Hidden FAT16 <3 61  SpeedStor       ab  Darwin boot     f2  DOS secondary  
    16  Hidden FAT16    63  GNU HURD or Sys af  HFS / HFS+      fb  VMware VMFS    
    17  Hidden HPFS/NTF 64  Novell Netware  b7  BSDI fs         fc  VMware VMKCORE 
    18  AST SmartSleep  65  Novell Netware  b8  BSDI swap       fd  Linux raid auto
    1b  Hidden W95 FAT3 70  DiskSecure Mult bb  Boot Wizard hid fe  LANstep        
    1c  Hidden W95 FAT3 75  PC/IX           be  Solaris boot    ff  BBT            
    1e  Hidden W95 FAT1 80  Old Minix      
    Hex code (type L to list all codes): 8e
    Changed type of partition 'Linux' to 'Linux LVM'

    Command (m for help): w
    The partition table has been altered!

    Calling ioctl() to re-read partition table.

    WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
    The kernel still uses the old table. The new table will be used at
    the next reboot or after you run partprobe(8) or kpartx(8)
    Syncing disks.
    ```
- 加载分区表
    ```shell
    [root@localhost ~]# partprobe
    ```
    > 执行 partprobe命令用于将磁盘分区表变化信息通知内核，并请求操作系统重新加载分区表，此方法可以不用重启系统；或者使用 reboot 重启服务器也是可以的。
- 查看加载的分区表
    ```shell
    [root@localhost ~]# fdisk -l

    Disk /dev/sda: 21.5 GB, 21474836480 bytes, 41943040 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk label type: dos
    Disk identifier: 0x000db710

    Device Boot      Start         End      Blocks   Id  System
    /dev/sda1   *        2048     2099199     1048576   83  Linux
    /dev/sda2         2099200    10485759     4193280   8e  Linux LVM
    /dev/sda3        10485760    20971519     5242880   8e  Linux LVM
    /dev/sda4        20971520    41943039    10485760   8e  Linux LVM

    Disk /dev/mapper/cl-root: 9118 MB, 9118416896 bytes, 17809408 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes


    Disk /dev/mapper/cl-swap: 536 MB, 536870912 bytes, 1048576 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    ```
    > 可以看到已经添加了 /dev/sda4
- 扩展 vg
    > 先创建 pv
    ```shell
    [root@localhost ~]# pvcreate /dev/sda4
    Physical volume "/dev/sda4" successfully created.
    ```
    > 命令 pvs、pvscan 和 pvdisplay 可以查看 pv 的相关信息
    > 然后使用 vgs 查看
    ```shell
    [root@localhost ~]# vgs
    VG #PV #LV #SN Attr   VSize VFree
    cl   2   2   0 wz--n- 8.99g    0 
    ```
    > 请注意 VG 这里是 cl，下面命令需要用到
- 把 sda4 加入到 LVM 组中
    ```shell
    # 先添加
    [root@localhost ~]# vgextend cl /dev/sda4
    Volume group "cl" successfully extended

    # 然后查看 vgs
    [root@localhost ~]# vgs
    VG #PV #LV #SN Attr   VSize   VFree  
    cl   3   2   0 wz--n- <18.99g <10.00g
    ```
- 扩展 lv
    * 先将 `/dev/sda4` 扩展到 `/dev/mapper/cl-root` (根据实际情况自行更改)
    ```shell
    [root@localhost ~]# lvextend /dev/mapper/cl-root /dev/sda4
    Size of logical volume cl/root changed from 8.49 GiB (2174 extents) to <18.49 GiB (4733 extents).
    Logical volume cl/root successfully resized.
    ```
    > 其中 `/dev/sda4`` 为 pv 的名称不带参数默认扩展所有 Free 空间，可以通过 -L 参数指定具体扩容的大小（单位为 “kKmMgGtT” 字节）例如扩展 1024 兆空间的命令为：lvextend -L +1024M /dev/mapper/cl-root /dev/sda4
    * 然后使用 lvs 查看情况，确认已增长
    ```shell
    [root@localhost ~]# lvs
    LV   VG Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
    root cl -wi-ao---- <18.49g                                                    
    swap cl -wi-ao---- 512.00m  
    ```
- xfs 在线扩容
    ```shell
    [root@localhost ~]# xfs_growfs /dev/mapper/cl-root
    meta-data=/dev/mapper/cl-root    isize=512    agcount=10, agsize=229120 blks
            =                       sectsz=512   attr=2, projid32bit=1
            =                       crc=1        finobt=0 spinodes=0
    data     =                       bsize=4096   blocks=2226176, imaxpct=25
            =                       sunit=0      swidth=0 blks
    naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
    log      =internal               bsize=4096   blocks=2560, version=2
            =                       sectsz=512   sunit=0 blks, lazy-count=1
    realtime =none                   extsz=4096   blocks=0, rtextents=0
    data blocks changed from 2226176 to 4846592
    ```
- 最终成功扩容
    ```shell
    root@localhost ~]# df -kh
    Filesystem           Size  Used Avail Use% Mounted on
    devtmpfs             484M     0  484M   0% /dev
    tmpfs                496M     0  496M   0% /dev/shm
    tmpfs                496M  7.1M  489M   2% /run
    tmpfs                496M     0  496M   0% /sys/fs/cgroup
    /dev/mapper/cl-root   19G  6.8G   12G  37% /
    /dev/sda1           1014M  248M  767M  25% /boot
    overlay               19G  6.8G   12G  37% /var/lib/docker/overlay2/3ea9819dc8d8b5c1e5fc1251e3b0720332b769ac7cefa13b477bca02/merged
    tmpfs                100M     0  100M   0% /run/user/0
    ```

> 本文依据 https://blog.csdn.net/catoop/article/details/116133631 操作完成